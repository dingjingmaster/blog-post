---
title: "Linux文件系统API"
date: 2022-05-30T16:57:08+08:00
tags: ['linux', 'kernel', 'filesystem', '文件系统']
categories: ['linux']
draft: true
---
## Linux 文件系统API
> 本节包含api级文档，大部分摘自源代码本身

### 文件系统类型 
#### `enum positive_aop_returns`
> Aop返回具有特定语义的代码。`address_space_operation`函数返回这些大常量，以向调用者表明特殊的语义。这些字节数要比页面中的字节数大得多，以允许函数返回在给定页面中操作的字节数。
- `AOP_WRITEPAGE_ACTIVATE`
通知调用者页回写已完成，页仍处于锁定状态，应视为活动页。VM使用这个提示将页面返回到活动列表—在不久的将来，它不会再次成为回写的候选者。其他调用者如果得到这个返回，必须小心地解锁页面。返回writepage ();
- `AOP_TRUNCATED_PAGE`
交付一个锁定页面的AOP方法已经解锁了它，并且页面可能已经被截断了。调用者应该返回获取新页面并再次尝试。aop将采取合理的预防措施，不发生骚乱。如果调用者持有一个页引用，它应该在重试之前删除它。返回`readpage()`。

#### `struct address_space`
> 可缓存、可映射对象的内容。
```c
struct address_space {
  struct inode            *host;
  struct xarray           i_pages;
  struct rw_semaphore     invalidate_lock;
  gfp_t gfp_mask;
  atomic_t i_mmap_writable;
#ifdef CONFIG_READ_ONLY_THP_FOR_FS;
  atomic_t nr_thps;
#endif;
  struct rb_root_cached   i_mmap;
  struct rw_semaphore     i_mmap_rwsem;
  unsigned long           nrpages;
  pgoff_t writeback_index;
  const struct address_space_operations *a_ops;
  unsigned long           flags;
  errseq_t wb_err;
  spinlock_t private_lock;
  struct list_head        private_list;
  void *private_data;
};
```

#### `struct file_ra_state`
> 跟踪文件的预读状态

```c
struct file_ra_state {
  pgoff_t start;
  unsigned int size;
  unsigned int async_size;
  unsigned int ra_pages;
  unsigned int mmap_miss;
  loff_t prev_pos;
};
```

#### `struct renamedata`
> 包含重命名所需的所有信息
```c
struct renamedata {
  struct user_namespace *old_mnt_userns;
  struct inode *old_dir;
  struct dentry *old_dentry;
  struct user_namespace *new_mnt_userns;
  struct inode *new_dir;
  struct dentry *new_dentry;
  struct inode **delegated_inode;
  unsigned int flags;
};
```

#### ``


---- 未完待续...
