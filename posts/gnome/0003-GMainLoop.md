---
title: "GMainLoop"
date: 2023-04-14T09:04:30+08:00
tags: ['linux', 'gnome', 'glib', 'gmainloop', 'gio']
categories: ['gnome']
draft: false
---

## 从g\_main\_loop\_new开始...

```c
GMainLoop* g_main_loop_new (GMainContext* context, gboolean is_running)
{
    GMainLoop *loop;

    if (!context)
        context = g_main_context_default();
  
    g_main_context_ref (context);

    loop = g_new0 (GMainLoop, 1);
    loop->context = context;
    loop->is_running = is_running != FALSE;
    loop->ref_count = 1;

    return loop;
}
```

```c
GMainContext* g_main_context_default (void)
{
    static GMainContext *default_main_context = NULL;

    if (g_once_init_enter (&default_main_context)) {
        GMainContext *context;
        // 等价于：g_main_context_new_with_flags (G_MAIN_CONTEXT_FLAGS_NONE);
        context = g_main_context_new ();
        g_once_init_leave (&default_main_context, context);
    }

    return default_main_context;
}
```

```c
GMainContext* g_main_context_ref (GMainContext *context)
{
    int old_ref_count;

    g_return_val_if_fail (context != NULL, NULL);

    old_ref_count = g_atomic_int_add (&context->ref_count, 1);
    g_return_val_if_fail (old_ref_count > 0, NULL);

    return context;
}
```

```c
GMainContext* g_main_context_new_with_flags (GMainContextFlags flags)
{
    static gsize initialised;
    GMainContext *context;

    if (g_once_init_enter (&initialised)) {
        g_once_init_leave (&initialised, TRUE);
    }

    context = g_new0 (GMainContext, 1);

    g_mutex_init (&context->mutex);
    g_cond_init (&context->cond);

    context->sources = g_hash_table_new (NULL, NULL);
    context->owner = NULL;
    context->flags = flags;
    context->waiters = NULL;

    context->ref_count = 1;

    context->next_id = 1;
  
    context->source_lists = NULL;
  
    context->poll_func = g_poll;
  
    context->cached_poll_array = NULL;
    context->cached_poll_array_size = 0;
  
    context->pending_dispatches = g_ptr_array_new ();
  
    context->time_is_fresh = FALSE;
  
    context->wakeup = g_wakeup_new ();
    g_wakeup_get_pollfd (context->wakeup, &context->wake_up_rec);
    g_main_context_add_poll_unlocked (context, 0, &context->wake_up_rec);

    G_LOCK (main_context_list);
    main_context_list = g_slist_append (main_context_list, context);

    G_UNLOCK (main_context_list);

    return context;
}
```

## g\_main\_loop\_run

```c
// 全局变量
G_LOCK_DEFINE_STATIC (main_context_list);
static GSList *main_context_list = NULL;
```

```c
void g_main_loop_run (GMainLoop *loop)
{
    GThread *self = G_THREAD_SELF;

    g_return_if_fail (loop != NULL);
    g_return_if_fail (g_atomic_int_get (&loop->ref_count) > 0);

    /* Hold a reference in case the loop is unreffed from a callback function */
    g_atomic_int_inc (&loop->ref_count);

    /**
     * 尝试成为指定上下文的所有者。
     * 如果其他线程是该上下文的所有者，则立即返回FALSE。
     * 所有者可以再次要求所有权，并且当g_main_context_release()被调用的次数与g_main_context_acquire()相同时，所有者将释放所有权。
     *
     * 先不必查看此处代码，多线程情况下....
     */
    if (!g_main_context_acquire (loop->context)) {
        gboolean got_ownership = FALSE;
      
        /* Another thread owns this context */
        LOCK_CONTEXT (loop->context);

        g_atomic_int_set (&loop->is_running, TRUE);

        while (g_atomic_int_get (&loop->is_running) && !got_ownership) {
            got_ownership = g_main_context_wait_internal (loop->context, &loop->context->cond, &loop->context->mutex);
        }
      
        if (!g_atomic_int_get (&loop->is_running)) {
	        UNLOCK_CONTEXT (loop->context);
	        if (got_ownership)
	            g_main_context_release (loop->context);
	        g_main_loop_unref (loop);
	        return;
	    }

        g_assert (got_ownership);
    }
    else
        LOCK_CONTEXT (loop->context);

    if (loop->context->in_check_or_prepare) {
        g_warning ("g_main_loop_run(): called recursively from within a source's "
		    "check() or prepare() member, iteration not possible.");
        g_main_loop_unref (loop);
        return;
    }

    g_atomic_int_set (&loop->is_running, TRUE);

    // 这里已经是死循环了
    while (g_atomic_int_get (&loop->is_running)) {
        // 处理事件
        g_main_context_iterate (loop->context, TRUE, TRUE, self);
    }

    UNLOCK_CONTEXT (loop->context);
  
    g_main_context_release (loop->context);
  
    g_main_loop_unref (loop);
}
```

```c
static gboolean g_main_context_wait_internal (GMainContext *context, GCond* cond, GMutex* mutex)
{
    gboolean result = FALSE;
    GThread *self = G_THREAD_SELF;
    gboolean loop_internal_waiter;
  
    if (context == NULL)
        context = g_main_context_default ();

    loop_internal_waiter = (mutex == &context->mutex);
  
    if (!loop_internal_waiter)
        LOCK_CONTEXT (context);

    // context 不属于本线程 情况(比如：属于主线程context)
    if (context->owner && context->owner != self) {
        GMainWaiter waiter;

        waiter.cond = cond;
        waiter.mutex = mutex;

        context->waiters = g_slist_append (context->waiters, &waiter);
      
        if (!loop_internal_waiter)
            UNLOCK_CONTEXT (context);
        g_cond_wait (cond, mutex);
        if (!loop_internal_waiter)
            LOCK_CONTEXT (context);

        context->waiters = g_slist_remove (context->waiters, &waiter);
    }

    if (!context->owner) {
        context->owner = self;
        g_assert (context->owner_count == 0);
    }

    if (context->owner == self) {
        context->owner_count++;
        result = TRUE;
    }

    if (!loop_internal_waiter)
        UNLOCK_CONTEXT (context); 
  
    return result;
}
```

```c
static gboolean g_main_context_iterate (GMainContext *context, gboolean block, gboolean dispatch, GThread* self)
{
    gint max_priority = 0;
    gint timeout;
    gboolean some_ready;
    gint nfds, allocated_nfds;
    GPollFD *fds = NULL;
    gint64 begin_time_nsec G_GNUC_UNUSED;

    UNLOCK_CONTEXT (context);

    begin_time_nsec = G_TRACE_CURRENT_TIME;

    if (!g_main_context_acquire (context)) {
        gboolean got_ownership;

        LOCK_CONTEXT (context);

        if (!block)
	        return FALSE;

        got_ownership = g_main_context_wait_internal (context, &context->cond, &context->mutex);

        if (!got_ownership)
	        return FALSE;
    }
    else {
        LOCK_CONTEXT (context);
    }
  
    if (!context->cached_poll_array) {
        context->cached_poll_array_size = context->n_poll_records;
        context->cached_poll_array = g_new (GPollFD, context->n_poll_records);
    }

    allocated_nfds = context->cached_poll_array_size;
    fds = context->cached_poll_array;

    UNLOCK_CONTEXT (context);

    // prepare: 准备在主循环中轮询源。轮询的结果信息由调用g_main_context_query()确定。
    // 主要依次调用 GSource 中的 prepare() 函数
    //      1. 把 prepare() 返回 True 的事件源及其父事件源 标为 READY
    //      2. 把定时类事件中，超时的事件标记出来标记位 READY
    g_main_context_prepare (context, &max_priority); 

    // prepare -- query (检查有事件发生的GSource有几个)
    while ((nfds = g_main_context_query (context, max_priority, &timeout, fds, allocated_nfds)) > allocated_nfds) {
        LOCK_CONTEXT (context);
        g_free (fds);
        context->cached_poll_array_size = allocated_nfds = nfds;
        context->cached_poll_array = fds = g_new (GPollFD, nfds);
        UNLOCK_CONTEXT (context);
    }

    if (!block) {
        timeout = 0;
    }

    // polling: 开始监控事件
    g_main_context_poll (context, timeout, max_priority, fds, nfds);

    // polling -- check
    //  1. 检测监听的文件描述符是否有事件发生，有则传出
    //  2. 手动调用GSource的check()函数，返回 TRUE 则标记为 READY 并把 GSource 放到 dispatch 链表中
    some_ready = g_main_context_check (context, max_priority, fds, nfds);

    // dispatch
    //  调用 dispatch() 函数并根据返回值确定是否要释放资源
    if (dispatch) {
        g_main_context_dispatch (context);
    }

    g_main_context_release (context);

    g_trace_mark (begin_time_nsec, G_TRACE_CURRENT_TIME - begin_time_nsec,
            "GLib", "g_main_context_iterate",
            "Context %p, %s ⇒ %s", context, block ? "blocking" : "non-blocking", some_ready ? "dispatched" : "nothing");

    LOCK_CONTEXT (context);

    return some_ready;
}
```

prepare
```c
gboolean g_main_context_prepare (GMainContext* context, gint* priority)
{
    guint i;
    gint n_ready = 0;
    gint current_priority = G_MAXINT;
    GSource *source;
    GSourceIter iter;

    if (context == NULL)
        context = g_main_context_default ();
  
    LOCK_CONTEXT (context);

    context->time_is_fresh = FALSE;

    if (context->in_check_or_prepare) {
        g_warning ("g_main_context_prepare() called recursively from within a source's check() or prepare() member.");
        UNLOCK_CONTEXT (context);
        return FALSE;
    }

    /* If recursing, clear list of pending dispatches */
    for (i = 0; i < context->pending_dispatches->len; i++) {
        if (context->pending_dispatches->pdata[i]) {
            g_source_unref_internal ((GSource *)context->pending_dispatches->pdata[i], context, TRUE);
        }
    }
    g_ptr_array_set_size (context->pending_dispatches, 0);
  
    /* Prepare all sources */
    context->timeout = -1;
  
    g_source_iter_init (&iter, context, TRUE);
    while (g_source_iter_next (&iter, &source)) {
        gint source_timeout = -1;

        if (SOURCE_DESTROYED (source) || SOURCE_BLOCKED (source))
	        continue;
        if ((n_ready > 0) && (source->priority > current_priority))
	        break;

        if (!(source->flags & G_SOURCE_READY)) {
	        gboolean result;
	        gboolean (*prepare) (GSource* source, gint* timeout);

            // 调用 prepare
            prepare = source->source_funcs->prepare;
            if (prepare) {
                gint64 begin_time_nsec G_GNUC_UNUSED;

                context->in_check_or_prepare++;
                UNLOCK_CONTEXT (context);

                begin_time_nsec = G_TRACE_CURRENT_TIME;

                result = (*prepare) (source, &source_timeout);

                LOCK_CONTEXT (context);
                context->in_check_or_prepare--;
            }
            else {
                source_timeout = -1;
                result = FALSE;
            }

            if (result == FALSE && source->priv->ready_time != -1) {
                if (!context->time_is_fresh) {
                    context->time = g_get_monotonic_time ();
                    context->time_is_fresh = TRUE;
                }

                if (source->priv->ready_time <= context->time) {
                    source_timeout = 0;
                    result = TRUE;
                }
                else {
                    gint64 timeout;

                    /* rounding down will lead to spinning, so always round up */
                    timeout = (source->priv->ready_time - context->time + 999) / 1000;

                    if (source_timeout < 0 || timeout < source_timeout)
                        source_timeout = MIN (timeout, G_MAXINT);
                }
            }

	        if (result) {
	            GSource *ready_source = source;

	            while (ready_source) {
		            ready_source->flags |= G_SOURCE_READY;
		            ready_source = ready_source->priv->parent_source;
		        }
	        }
	    }

        if (source->flags & G_SOURCE_READY) {
	        n_ready++;
	        current_priority = source->priority;
	        context->timeout = 0;
	    }
      
        if (source_timeout >= 0) {
	        if (context->timeout < 0)
	            context->timeout = source_timeout;
	        else
	            context->timeout = MIN (context->timeout, source_timeout);
	    }
    }
  
    g_source_iter_clear (&iter);

    UNLOCK_CONTEXT (context);
  
    if (priority)
        *priority = current_priority;

    return (n_ready > 0);
}
```

// query()
```c
gint g_main_context_query (GMainContext *context, gint max_priority, gint* timeout, GPollFD* fds, gint n_fds)
{
    gint n_poll;
    GPollRec *pollrec, *lastpollrec;
    gushort events;
  
    LOCK_CONTEXT (context);

    /**
     * fds is filled sequentially from poll_records. Since poll_records
     * are incrementally sorted by file descriptor identifier, fds will
     * also be incrementally sorted.
     */
    n_poll = 0;
    lastpollrec = NULL;
    for (pollrec = context->poll_records; pollrec; pollrec = pollrec->next) {
        if (pollrec->priority > max_priority) {
            continue;
        }

        /**
         * In direct contradiction to the Unix98 spec, IRIX runs into
         * difficulty if you pass in POLLERR, POLLHUP or POLLNVAL
         * flags in the events field of the pollfd while it should
         * just ignoring them. So we mask them out here.
         */
         // 忽略事件
        events = pollrec->fd->events & ~(G_IO_ERR|G_IO_HUP|G_IO_NVAL);

        /* This optimization --using the same GPollFD to poll for more
         * than one poll record-- relies on the poll records being
         * incrementally sorted.
         */
        if (lastpollrec && pollrec->fd->fd == lastpollrec->fd->fd) {
            if (n_poll - 1 < n_fds) {
                fds[n_poll - 1].events |= events;
            }
        }
        else {
            if (n_poll < n_fds) {
                fds[n_poll].fd = pollrec->fd->fd;
                fds[n_poll].events = events;
                fds[n_poll].revents = 0;
            }
            n_poll++;
        }

        lastpollrec = pollrec;
    }

    context->poll_changed = FALSE;
  
    if (timeout) {
        *timeout = context->timeout;
        if (*timeout != 0) {
            context->time_is_fresh = FALSE;
        }
    }

    UNLOCK_CONTEXT (context);

    return n_poll;
}
```

// poll
```c
static void g_main_context_poll (GMainContext *context, gint timeout, gint priority, GPollFD* fds, gint n_fds)
{
    GPollFunc poll_func;

    if (n_fds || timeout != 0) {
        int ret, errsv;

        LOCK_CONTEXT (context);

        poll_func = context->poll_func;

        UNLOCK_CONTEXT (context);
        ret = (*poll_func) (fds, n_fds, timeout);
        errsv = errno;
        if (ret < 0 && errsv != EINTR) {
            // 错误警告
	    }
    } /* if (n_fds || timeout != 0) */
}
```

// check
```c
gboolean g_main_context_check (GMainContext *context, gint max_priority, GPollFD* fds, gint n_fds)
{
    GSource *source;
    GSourceIter iter;
    GPollRec *pollrec;
    gint n_ready = 0;
    gint i;
   
    LOCK_CONTEXT (context);

    if (context->in_check_or_prepare) {
        g_warning ("g_main_context_check() called recursively from within a source's check() or prepare() member.");
        UNLOCK_CONTEXT (context);
        return FALSE;
    }

    for (i = 0; i < n_fds; i++) {
        if (fds[i].fd == context->wake_up_rec.fd) {
            if (fds[i].revents) {
              g_wakeup_acknowledge (context->wakeup);
            }
            break;
        }
    }

    /* If the set of poll file descriptors changed, bail out
    * and let the main loop rerun
    */
    if (context->poll_changed) {
        UNLOCK_CONTEXT (context);
        return FALSE;
    }

    /* The linear iteration below relies on the assumption that both
    * poll records and the fds array are incrementally sorted by file
    * descriptor identifier.
    */
    i = 0;
    pollrec = context->poll_records;
    while (pollrec && i < n_fds) {
        /* Make sure that fds is sorted by file descriptor identifier. */
        g_assert (i <= 0 || fds[i - 1].fd < fds[i].fd);

        /* Skip until finding the first GPollRec matching the current GPollFD. */
        while (pollrec && pollrec->fd->fd != fds[i].fd)
            pollrec = pollrec->next;

        /* Update all consecutive GPollRecs that match. */
        while (pollrec && pollrec->fd->fd == fds[i].fd) {
            if (pollrec->priority <= max_priority) {
                pollrec->fd->revents = fds[i].revents & (pollrec->fd->events | G_IO_ERR | G_IO_HUP | G_IO_NVAL);
            }
            pollrec = pollrec->next;
        }

        /* Iterate to next GPollFD. */
        i++;
    }

    g_source_iter_init (&iter, context, TRUE);
    while (g_source_iter_next (&iter, &source)) {
        if (SOURCE_DESTROYED (source) || SOURCE_BLOCKED (source))
	        continue;
        if ((n_ready > 0) && (source->priority > max_priority))
	        break;

        if (!(source->flags & G_SOURCE_READY)) {
            gboolean result;
            gboolean (*check) (GSource *source);

            check = source->source_funcs->check;

            if (check) {
                gint64 begin_time_nsec G_GNUC_UNUSED;

                /* If the check function is set, call it. */
                context->in_check_or_prepare++;
                UNLOCK_CONTEXT (context);

                begin_time_nsec = G_TRACE_CURRENT_TIME;

                result = (* check) (source);

                TRACE (GLIB_MAIN_AFTER_CHECK (source, check, result));

                g_trace_mark (begin_time_nsec, G_TRACE_CURRENT_TIME - begin_time_nsec,
                            "GLib", "GSource.check",
                            "%s ⇒ %s",
                            (g_source_get_name (source) != NULL) ? g_source_get_name (source) : "(unnamed)",
                            result ? "dispatch" : "ignore");

                LOCK_CONTEXT (context);
                context->in_check_or_prepare--;
            }
            else {
                result = FALSE;
            }

            
            if (result == FALSE) {
                GSList *tmp_list;

                /* If not already explicitly flagged ready by ->check()
                * (or if we have no check) then we can still be ready if
                * any of our fds poll as ready.
                */
                for (tmp_list = source->priv->fds; tmp_list; tmp_list = tmp_list->next) {
                    GPollFD *pollfd = tmp_list->data;

                    if (pollfd->revents) {
                        result = TRUE;
                        break;
                    }
                }
            }

            if (result == FALSE && source->priv->ready_time != -1) {
                if (!context->time_is_fresh) {
                    context->time = g_get_monotonic_time ();
                    context->time_is_fresh = TRUE;
                }

                if (source->priv->ready_time <= context->time)
                    result = TRUE;
            }

	        if (result) {
	            GSource *ready_source = source;
	            while (ready_source) {
		            ready_source->flags |= G_SOURCE_READY;
		            ready_source = ready_source->priv->parent_source;
		        }
	        }
	    }

        if (source->flags & G_SOURCE_READY) {
            g_source_ref (source);
	        g_ptr_array_add (context->pending_dispatches, source);

	        n_ready++;

            /* never dispatch sources with less priority than the first
            * one we choose to dispatch
            */
            max_priority = source->priority;
	    }
    }
    
    g_source_iter_clear (&iter);

    TRACE (GLIB_MAIN_CONTEXT_AFTER_CHECK (context, n_ready));

    UNLOCK_CONTEXT (context);

    return n_ready > 0;
}
```

// dispatch
```c
void g_main_context_dispatch (GMainContext *context)
{
    LOCK_CONTEXT (context);

    if (context->pending_dispatches->len > 0) {
        g_main_dispatch (context);
    }

    UNLOCK_CONTEXT (context);
}
```

// g_main_dispatch
```c
/* HOLDS: context's lock */
static void g_main_dispatch (GMainContext *context)
{
    GMainDispatch *current = get_dispatch ();
    guint i;

    for (i = 0; i < context->pending_dispatches->len; i++) {
        GSource *source = context->pending_dispatches->pdata[i];
        context->pending_dispatches->pdata[i] = NULL;
        g_assert (source);

        source->flags &= ~G_SOURCE_READY;

        if (!SOURCE_DESTROYED (source)) {
	        gboolean was_in_call;
	        gpointer user_data = NULL;
	        GSourceFunc callback = NULL;
	        GSourceCallbackFuncs *cb_funcs;
	        gpointer cb_data;
	        gboolean need_destroy;

	        gboolean (*dispatch) (GSource*, GSourceFunc, gpointer);
            GSource *prev_source;
            gint64 begin_time_nsec G_GNUC_UNUSED;

	        dispatch = source->source_funcs->dispatch;
	        cb_funcs = source->callback_funcs;
	        cb_data = source->callback_data;

	        if (cb_funcs)
	            cb_funcs->ref (cb_data);
	  
	        if ((source->flags & G_SOURCE_CAN_RECURSE) == 0)
	            block_source (source);
	  
	        was_in_call = source->flags & G_HOOK_FLAG_IN_CALL;
	        source->flags |= G_HOOK_FLAG_IN_CALL;

	        if (cb_funcs)
	            cb_funcs->get (cb_data, source, &callback, &user_data);

	        UNLOCK_CONTEXT (context);

            /* These operations are safe because 'current' is thread-local
            * and not modified from anywhere but this function.
            */
            prev_source = current->source;
            current->source = source;
            current->depth++;

            begin_time_nsec = G_TRACE_CURRENT_TIME;

            need_destroy = !(*dispatch) (source, callback, user_data);

            g_trace_mark (begin_time_nsec, G_TRACE_CURRENT_TIME - begin_time_nsec,
                        "GLib", "GSource.dispatch",
                        "%s ⇒ %s",
                        (g_source_get_name (source) != NULL) ? g_source_get_name (source) : "(unnamed)",
                        need_destroy ? "destroy" : "keep");

            current->source = prev_source;
            current->depth--;

	        if (cb_funcs)
	            cb_funcs->unref (cb_data);

 	        LOCK_CONTEXT (context);
	  
	        if (!was_in_call)
	            source->flags &= ~G_HOOK_FLAG_IN_CALL;

	        if (SOURCE_BLOCKED (source) && !SOURCE_DESTROYED (source))
	            unblock_source (source);
	  
            /* Note: this depends on the fact that we can't switch
            * sources from one main context to another
            */
	        if (need_destroy && !SOURCE_DESTROYED (source)) {
	            g_assert (source->context == context);
	            g_source_destroy_internal (source, context, TRUE);
	        }
	    }
      
        g_source_unref_internal (source, context, TRUE);
    }
    g_ptr_array_set_size (context->pending_dispatches, 0);
}
```

## 总结

### 使用说明
1. GMainLoop(主事件循环)管理GLib和GTK+应用程序的所有可用事件源。
2. 事件源主要有：超时事件、文件描述符；其中文件描述符可以是任何打开的（管道、文件、套接字等事件，这块用 poll 监听文件描述符上的事件）
3. 为了允许在不同的线程中处理多个独立的源集，每个源都与一个GMainContext相关联。GMainContext只能在单个线程中运行，但是可以向它添加源，也可以从其他线程中删除源。所有操作在GMainContext或内置GSource上的函数都是线程安全的。
4. 每个事件源都有一个优先级。默认优先级G_PRIORITY_DEFAULT为0。小于0的值表示更高的优先级。值大于0表示优先级较低。来自高优先级源的事件总是在来自低优先级源的事件之前处理:如果有几个源已经准备好分派，具有相同最高优先级的源将在当前GMainContext迭代中分派，其余的将等待到后续的GMainContext迭代，此时它们具有准备好分派的源中的最高优先级。
5. 使用`g_source_attach`把事件添加到上下文中。

### 代码梳理

先贴glib中的一幅图（这幅图就是整个代码实现逻辑）：

<div align=center><img src='/pic/gnome/g-2-0.png'/></div>
<center>GMainLoop状态图</center>

1. `g_main_loop_new` 中主要是使用`g_main_context_default`->`g_main_context_new_with_flags`创建指定的`context`并加入到全局`context`变量列表中`main_context_list`
2. `g_main_loop_run` **死循环**调用`g_main_context_iterate`依次处理传入事件的事件源，此时针对每个事件源的处理过程如上图：'GMainLoop状态图' 所示，执行流程如下：
    - `prepare`: 准备在主循环中轮询源。轮询的结果信息由调用g_main_context_query()确定。调用 GSource 中的 prepare() 函数，如果此调用返回True或者此事件源是定时器且已经到时，则标记为 READY，以供后续处理
    - `query`: query (检查有事件发生的GSource有几个)
    - `polling`：调用 poll 设置文件描述符事件监听
    - `check`：针对READY的GSource，手动调用GSource的check()函数，返回 TRUE 则标记为 READY 并把 GSource 放到 dispatch 链表中
    - `dispatch`: 接收回调函数和用户数据，然后调用GSource指定的dispatch函数