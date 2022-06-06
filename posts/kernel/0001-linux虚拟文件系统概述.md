---
title: "Linux虚拟文件系统概述"
date: 2022-05-18T13:46:53+08:00
tags: ['linux', '虚拟文件系统', 'Linux vfs', vfs]
categories: ['kernel']
draft: false
---

## 概述
虚拟文件系统(也称为虚拟文件系统交换机)是内核中的软件层，它为用户空间程序提供文件系统接口。它还在内核中提供了一个抽象，允许不同的文件系统实现共存。

VFS系统调用open(2)、stat(2)、read(2)、write(2)、chmod(2)等等都是从进程上下文中调用的。

> 文件系统锁在文档"锁"中进行了介绍。

## 目录条目缓存(dcache)

VFS实现了open(2)、stat(2)、chmod(2)和类似的系统调用。传递给它们的pathname参数由VFS用于搜索目录条目缓存(也称为dentry缓存或dcache)。这提供了一种非常快速的查找机制，可以将路径名(filename)转换为特定的dentry。dentry存储在RAM中，从不保存到磁盘:它们的存在只是为了性能。

dentry缓存是整个文件空间的视图。由于大多数计算机不能同时将所有dentry放入RAM中，所以缓存的一些位元会丢失。为了将路径名解析为dentry, VFS可能不得不在过程中创建dentry，然后加载inode。这是通过查找inode来完成的。

## inode 对象

单个dentry通常有一个指向inode的指针。inode是文件系统对象，比如普通文件、目录、fifo 和其他东西。它们要么存在于磁盘(对于块设备文件系统)，要么存在于内存(对于伪文件系统)。当需要时，将磁盘上的inode复制到内存中，并将对inode的更改写回磁盘。一个inode可以被多个dentry指向(例如，硬链接就是这样做的)。

要查找一个inode, VFS需要调用父目录inode的lookup()方法。该方法由inode所在的特定文件系统实现安装。一旦VFS拥有了所需的dentry(以及inode)，我们就可以执行所有无聊的操作，比如打开(2)文件或stat(2)文件以查看inode数据。stat(2)操作相当简单:一旦VFS有了dentry，它就会查看inode数据，并将其中一些数据传递回用户空间。

## File 对象

打开文件需要另一项操作:分配文件结构(这是文件描述符的内核端实现)。使用指向dentry的指针和一组文件操作成员函数初始化新分配的文件结构。这些数据来自inode数据。然后调用open()文件方法，这样特定的文件系统实现就可以完成它的工作。您可以看到，这是由VFS执行的另一个开关。文件结构被放置到进程的文件描述符表中。

读取、写入和关闭文件(以及其他各种VFS操作)是通过使用用户空间文件描述符来获取适当的文件结构，然后调用所需的文件结构方法来完成所需的操作。只要文件处于打开状态，它就保持使用dentry，这又意味着VFS inode仍在使用。

## 注册和挂载文件系统

注册和注销文件系统，使用以下API函数:

```c
#include <linux/fs.h>

extern int register_filesystem(struct file_system_type *);
extern int unregister_filesystem(struct file_system_type *);
```

传递的`struct file_system_type`描述了文件系统。当需要将文件系统挂载到名称空间中的目录时，VFS将为特定的文件系统调用适当的`mount()`方法。新vfmount将把 `->mount()` 返回的树形结构附加到挂载点，这样当路径名解析到达挂载点时，它将跳转到该vfmount的根目录。

您可以在文件`/proc/filesystems`中看到注册到内核的所有文件系统。

### `struct file_system_type`

这描述了文件系统。在2.6.39内核中，定义了以下成员:
```c
struct file_system_type {
    const char *name;
    int fs_flags;
    struct dentry *(*mount) (struct file_system_type *, int, const char *, void *);
    void (*kill_sb) (struct super_block *);
    struct module *owner;
    struct file_system_type * next;
    struct list_head fs_supers;
    struct lock_class_key s_lock_key;
    struct lock_class_key s_umount_key;
};
```
- `name`
文件系统类型的名称，例如“ext2”、“iso9660”、“msdos”等。
- `fs_flags`
各种标志(例如:`FS_REQUIRES_DEV`, `FS_NO_DCACHE`等)。
- `mount`
当应该安装该文件系统的新实例时调用的方法。`mount()`方法必须返回调用者请求的树的根`dentry`。对其超级块的激活引用必须被抓取，并且超级块必须被锁定。失败时，它应该返回`ERR_PTR`(错误)。
    - 参数1(`struct file_system_type *`):描述由特定文件系统代码部分初始化的文件系统
    - 参数2(`int`):挂载flag; 
    - 参数3(`const char *`): 要挂载的设备名；
    - 参数4(`void*`)：任意的挂载选项，通常是ASCII字符串(参见“Mount Options”一节)

- `kill_sb`
当该文件系统的实例应该关闭时要调用的方法
- `owner`
对于内部VFS使用:你应该在大多数情况下初始化`THIS_MODULE`。
- `next`
对于内部VFS使用:你应该将其初始化为NULL
- `s_lock_key`, `s_umount_key`
lockdep-specific...

## superblock 对象

超级块对象表示一个挂载的文件系统。

### `struct super_operations`
这描述了VFS如何操作文件系统的超级块。在2.6.22内核中，定义了以下成员:

```c
struct super_operations {
    struct inode *(*alloc_inode)(struct super_block *sb);
    void (*destroy_inode)(struct inode *);

    void (*dirty_inode) (struct inode *, int flags);
    int (*write_inode) (struct inode *, int);
    void (*drop_inode) (struct inode *);
    void (*delete_inode) (struct inode *);
    void (*put_super) (struct super_block *);
    int (*sync_fs)(struct super_block *sb, int wait);
    int (*freeze_fs) (struct super_block *);
    int (*unfreeze_fs) (struct super_block *);
    int (*statfs) (struct dentry *, struct kstatfs *);
    int (*remount_fs) (struct super_block *, int *, char *);
    void (*clear_inode) (struct inode *);
    void (*umount_begin) (struct super_block *);

    int (*show_options)(struct seq_file *, struct dentry *);

    ssize_t (*quota_read)(struct super_block *, int, char *, size_t, loff_t);
    ssize_t (*quota_write)(struct super_block *, int, const char *, size_t, loff_t);
    int (*nr_cached_objects)(struct super_block *);
    void (*free_cached_objects)(struct super_block *, int);
};
```
除非另有说明，否则所有方法都是在不持有任何锁的情况下调用的。这意味着大多数方法都可以安全地阻塞。所有方法只能从进程上下文中调用(即不能从中断处理程序或下半部调用)。

- `alloc_inode` 
该方法由`alloc_inode()`调用，为结构`inode`分配内存并对其进行初始化。如果未定义此函数，则分配一个简单的`struct inode`。通常，`alloc_inode` 会被用来分配一个更大的结构，其中包含一个`struct inode`嵌入其中。

- `destroy_inode`
`destroy_inode()`调用这个方法来释放分配给结构`inode`的资源。只有在定义了`->alloc_inode`时才需要它，它只是简单地撤销`->alloc_inode`所做的任何事情。

- `dirty_inode`
当一个`inode`被标记为dirty时，VFS会调用这个方法。这是专门针对被标记为`dirty`的`inode`本身，而不是它的数据。如果`fdatasync()`需要持久化更新，则`I_DIRTY_DATASYNC`将在`flags`参数中设置。

- `write_inode`
当VFS需要将一个inode写入磁盘时，调用此方法。第二个参数表示写是否应该是同步的，并不是所有的文件系统都检查这个标志。

- `drop_inode`
在删除对inode的最后一次访问时调用，并保持`inode->i_lock`自旋锁。
此方法应该是`NULL`(普通UNIX文件系统语义)或`generic_delete_inode`(对于不希望缓存`inode`的文件系统-导致无论`i_nlink`的值如何，总是调用`delete_inode`)
`generic_delete_inode()`行为相当于在`put_inode()`情况下使用`force_delete`的旧实践，但不具有`force_delete()`方法所具有的竞争。

- `delete_inode`
当VFS想要删除一个inode时调用

- `put_super`
当VFS希望释放超级块(即卸载)时调用。在超级块锁被持有时调用

- `sync_fs`
当VFS写入与超级块关联的所有脏数据时调用。第二个参数指示方法是否应该等待直到写操作完成。可选的。

- `freeze_fs`
当VFS锁定文件系统并强制其进入一致状态时调用。该方法目前由逻辑卷管理器(LVM)使用。

- `unfreeze_fs`
当VFS解锁文件系统并使其重新可写时调用。

- `statfs`
当VFS需要获取文件系统统计信息时调用。

- `remount_fs`
当重新装载文件系统时调用。这是在内核锁被持有时调用的

- `clear_inode`
调用，则VFS清除该inode。可选

- `umount_begin`
当VFS卸载文件系统时调用。

- `show_options`
由VFS调用，以显示`/proc//mounts`。(参见“安装选项”一节)

- `quota_read`
调用VFS来读取文件系统配额文件。

- `quota_write`
调用VFS来写入文件系统的配额文件。

- `nr_cached_objects`
由sb缓存收缩函数为文件系统调用，以返回它所包含的可释放缓存对象的数量。可选的。

- `free_cache_objects`
由sb缓存收缩函数为文件系统调用，以扫描指定的尝试释放对象的数量。可选，但任何实现此方法的文件系统还需要实现`->nr_cached_objects`才能正确调用它。
我们无法处理文件系统可能遇到的任何错误，因此返回类型为void。如果VM尝试在`GFP_NOFS`条件下回收，则永远不会调用该方法，该方法本身不需要处理这种情况。
实现必须包括已完成的任何扫描循环内的条件重调度调用。这使得VFS可以确定适当的扫描批处理大小，而不必担心实现是否会由于大的扫描批处理大小而导致延迟问题。

设置inode的人负责填充`i_op`字段。这是一个指向`struct inode_operations`的指针，该指针描述了可以在单个`inode上`执行的方法。

### `struct xattr_handlers`
在支持扩展属性(`xattrs`)的文件系统上，`s_xattr`超级块字段指向一个以null结尾的xattr处理程序数组。扩展属性是 `key:value` 对。
- `name`
指示处理程序匹配具有指定名称的属性(例如`system.posix_acl_access`);`prefix`字段必须为NULL。
- `prefix`
指示处理程序匹配所有具有指定名称前缀的属性(如“user.”);名称字段必须为NULL。

- `list`
确定是否应该为特定dentry列出与这个xattr处理程序匹配的属性。用于一些`listxattr`实现，如`generic_listxattr`。

- `get`
由VFS调用以获取特定扩展属性的值。该方法由`getxattr(2)`系统调用调用。

- `set`
由VFS调用，以设置特定扩展属性的值。当新值为NULL时，调用该函数以删除特定的扩展属性。该方法由`setxattr(2)`和`removexattr(2)`系统调用调用。

当文件系统的xattr处理程序与指定的属性名不匹配或者文件系统不支持扩展属性时，所有`*xattr(2)`系统调用将返回`-EOPNOTSUPP`。

## inode 对象
inode对象表示文件系统中的一个对象

### `struct inode_operations`
这描述了VFS如何操作文件系统中的inode。在2.6.22内核中，定义了以下成员:

```c
struct inode_operations {
    int (*create) (struct user_namespace *, struct inode *,struct dentry *, umode_t, bool);
    struct dentry * (*lookup) (struct inode *,struct dentry *, unsigned int);
    int (*link) (struct dentry *,struct inode *,struct dentry *);
    int (*unlink) (struct inode *,struct dentry *);
    int (*symlink) (struct user_namespace *, struct inode *,struct dentry *,const char *);
    int (*mkdir) (struct user_namespace *, struct inode *,struct dentry *,umode_t);
    int (*rmdir) (struct inode *,struct dentry *);
    int (*mknod) (struct user_namespace *, struct inode *,struct dentry *,umode_t,dev_t);
    int (*rename) (struct user_namespace *, struct inode *, struct dentry *,
                       struct inode *, struct dentry *, unsigned int);
    int (*readlink) (struct dentry *, char __user *,int);
    const char *(*get_link) (struct dentry *, struct inode *,
                                 struct delayed_call *);
    int (*permission) (struct user_namespace *, struct inode *, int);
    struct posix_acl * (*get_acl)(struct inode *, int, bool);
    int (*setattr) (struct user_namespace *, struct dentry *, struct iattr *);
    int (*getattr) (struct user_namespace *, const struct path *, struct kstat *, u32, unsigned int);
    ssize_t (*listxattr) (struct dentry *, char *, size_t);
    void (*update_time)(struct inode *, struct timespec *, int);
    int (*atomic_open)(struct inode *, struct dentry *, struct file *,
                           unsigned open_flag, umode_t create_mode);
    int (*tmpfile) (struct user_namespace *, struct inode *, struct dentry *, umode_t);
    int (*set_acl)(struct user_namespace *, struct inode *, struct posix_acl *, int);
    int (*fileattr_set)(struct user_namespace *mnt_userns,
                            struct dentry *dentry, struct fileattr *fa);
    int (*fileattr_get)(struct dentry *dentry, struct fileattr *fa);
};
```
同样，调用所有方法时不会持有任何锁，除非另有说明。

- `create`
由`open(2)`和`create(2)`系统调用调用。只有在支持常规文件时才需要。你得到的`dentry`不应该有一个`inode`(即，它应该是一个负dentry)。在这里，您可能会使用`dentry`和新创建的`inode`调用`d_instantiate()`

- `lookup`
当VFS需要在父目录中查找一个`inode`时调用。要查找的名字在`dentry`中找到。这个方法必须调用`d_add()`来将找到的`inode`插入`dentry`。`inode`结构中的“`i_count`”字段应该递增。如果指定的`inode`不存在，则应该在`dentry`中插入一个`NULL inode`(这称为负dentry)。从这个例程返回错误代码必须只在真正发生错误时才可以，否则使用`create(2)`、`mknod(2)`、`mkdir(2)`等系统调用创建索引节点将会失败。如果你希望重载`dentry`方法，那么你应该初始化`dentry`中的“`d_dop`”字段;这是一个指向结构体“`dentry_operations`”的指针。在保存目录`inode`信号量的情况下调用此方法

- `link`
由`link(2)`系统调用调用。只有在支持硬链接时才需要。您可能需要调用`d_instantiate()`，就像在`create()`方法中那样

- `unlink`

- `symlink`
由`symlink(2)`系统调用调用。只有在希望支持符号链接时才需要。您可能需要调用`d_instantiate()`，就像在`create()`方法中那样

- `mkdir`

- `rmdir`

- `mknod`
由`mknod(2)`系统调用来创建一个设备(char, block) inode或一个命名管道(FIFO)或套接字。只有在希望支持创建这些类型的索引节点时才需要。您可能需要调用`d_instantiate()`，就像在`create()`方法中那样

- `rename`
由`rename(2)`系统调用调用，重命名对象，使其具有由第二个inode和dentry提供的父节点和名称。对于任何不支持或未知的标志，文件系统必须返回`-EINVAL`。目前实现了以下标志:
    - (1) `RENAME_NOREPLACE`:该标志表示如果重命名目标存在，重命名将失败，使用`-EEXIST`代替目标。VFS已经检查是否存在，因此对于本地文件系统，`RENAME_NOREPLACE`实现等价于普通重命名。
    - (2) `RENAME_EXCHANGE`: `exchange`源和目标。必须存在;这由VFS检查。与普通重命名不同，源和目标可能具有不同的类型。

- `get_link`
由VFS调用，以跟随到它所指向的`inode`的符号链接。只有在希望支持符号链接时才需要。这个方法返回要遍历的符号链接体(并可能使用`nd_jump_link()`重置当前位置)。如果在`inode`消失之前，主体不会消失，那么其他东西就不需要了;如果需要以其他方式固定它，通过让`get_link(…，…，done)`执行`set_delayed_call(done, destructor, argument)`来安排它的释放。在这种情况下，一旦VFS处理完您返回的函数体，就会调用`estructor(argument)`。可以在RCU模式下调用;由NULL dentry参数表示。如果请求不离开RCU模式就不能被处理，让它返回`ERR_PTR`(`-ECHILD`)。如果文件系统将符号链接目标存储在`->i_link`中，VFS可以直接使用它而不调用`->get_link()`;但是，`->get_link()`仍然必须提供。`—>i_link`必须在RCU宽限期后才能释放。

- `readlink`
这现在只是`readlink(2)`在`->get_link`使用`nd_jump_link()`或`object`实际上不是符号链接的情况下使用的重载。通常文件系统应该只实现`->get_link`用于符号链接，`readlink(2)`将自动使用它。

- `permission`
由VFS调用，以检查类`posix`文件系统上的访问权限。可以在`rcu-walk`模式下调用(`mask & MAY_NOT_BLOCK`)。如果在`rcu-walk`模式下，文件系统必须检查权限，而不能阻塞或存储到`inode`。如果遇到`rcu-walk`不能处理的情况，返回`-ECHILD`，它将在`refwalk`模式中再次被调用。

- `setattr`

- `getattr`

- `listxattr`

- `update_time`

- `atomic_open`
对一个`open`的最后一个组件调用。使用这个可选的方法，文件系统可以在一个原子操作中查找、创建和打开文件。如果它想把实际打开留给调用者(例如，如果文件是一个符号链接、设备，或者只是文件系统不会原子打开的东西)，它可能通过返回`finish_no_open(file, dentry)`来发出信号。仅当最后一个组件为负数或需要查找时，才调用此方法。缓存的正向`dentry`仍然由`f_op->open()`处理。如果文件已经创建，`FMODE_CREATED`标志应该设置在`file->f_mode`中。在`O_EXCL`的情况下，只有在文件不存在的情况下，该方法才会成功，因此`FMODE_CREATED`总是在成功时设置。

- `tmpfile`
在`O_TMPFILE` `open()`的末尾调用。可选，相当于在给定目录中自动创建、打开和解链接文件。

- `fileattr_get`
调用`ioctl(FS_IOC_GETFLAGS)`和`ioctl(FS_IOC_FSGETXATTR)`来检索杂项文件标志和属性。也可以在相关的`SET`操作之前调用，以检查发生了什么更改(在本例中，`i_rwsem`是锁独占的)。如果未设置，则退回到`f_op->ioctl()`。

- `fileattr_set`

## 地址空间对象(The Address Space Object)
地址空间对象用于对页缓存中的页进行分组和管理。它可以用来跟踪文件(或其他文件)中的页面，还可以跟踪文件各部分到进程地址空间的映射。

地址空间可以提供许多不同但相关的服务。这些方法包括通信内存压力、按地址查找页面以及跟踪标记为`Dirty`或`Writeback`的页面。

第一个可以单独用于其他的。VM可以尝试写入脏页以清除它们，也可以释放干净页以重用它们。为此，它可以在脏页面上调用`->writepage`方法，在干净页面上调用`->releasepage`方法，并设置`pagprivate`。没有`PagePrivate`和没有外部引用的干净页面将在不通知`address_space`的情况下被释放。

为了实现此功能，需要将页面放置在`LRU`上，并在使用页面时调用`lru_cache_add`和`mark_page_active`。

页面通常通过`->index`保存在基数树索引中。该树维护每个页面的`PG_Dirty`和`PG_Writeback`状态信息，以便快速找到带有这两个标志的页面。

`Dirty`标记主要由`mpage_writpages`(默认的`->writpages`方法)使用。它使用标记查找脏页并调用`->writepage`。如果没有使用`mpage_writepages`(即地址提供了自己的`->writepages`)，那么`PAGECACHE_TAG_DIRTY`标签几乎没有被使用。`Write_inode_now`和`sync_inode`使用它(通过`__sync_single_inode`)来检查`->writepages`是否成功地写出了整个地址空间。

`Writeback`标记由`filemap*` `wait*`和`sync_page*`函数使用，通过`filemap_fdatawait_range`函数等待所有的Writeback完成。

`address_space`处理程序可以附加额外的信息到一个页面，通常使用'`struct page`'中的'`private`'字段。如果附加了这些信息，则应该设置`PG_Private`标志。这将导致各种VM例程对`address_space`处理程序进行额外调用来处理该数据。

地址空间充当存储和应用程序之间的中介。每次将整个页的数据读入地址空间，并通过复制页或对页进行内存映射的方式提供给应用程序。应用程序将数据写入地址空间，然后通常以整个页面的形式将数据写回存储，但是`address_space`可以更好地控制写大小。

`read`进程实际上只需要'`readpage`'。写过程更为复杂，使用`write_begin`/`write_end`或`dirty_folio`将数据写入`address_space`，使用`writepage`和`writepage`将数据回写到存储。

在地址空间中添加和删除页面由`inode`的`i_mutex`保护。

当数据写入一个页面时，应该设置`PG_Dirty`标志。它通常保持设置状态，直到`writepage`要求写入它。这应该清除`PG_Dirty`并设置`PG_Writeback`。它实际上可以在`PG_Dirty`被清除后的任何时候被写入。一旦知道它是安全的，就清除`PG_Writeback`。

`Writeback`利用`writeback_control`结构来指导操作。这为`writepage`和`writpages`操作提供了一些关于回写请求的性质和原因，以及在哪些约束条件下执行的信息。它还用于向调用者返回关于写页面或写页面请求结果的信息。

## 处理回写期间的错误
大多数进行缓冲`I/O`的应用程序将定期调用文件同步调用(`fsync`、`fdatasync`、`msync`或`sync_file_range`)，以确保写入的数据已经到达后备存储。当回写过程中出现错误时，他们希望在文件同步请求时报告错误。在报告了一个请求的错误之后，对同一文件描述符的后续请求应该返回`0`，除非在上一次文件同步之后发生了进一步的回写错误。

理想情况下，内核只会在文件描述中报告错误，而这些文件描述的写操作后来又没有被写回。但是，通用的页面缓存基础设施不会跟踪已经污染每个单独页面的文件描述，因此不可能确定哪个文件描述符应该返回错误。

相反，内核中的通用回写错误跟踪基础设施解决了在错误发生时向`fsync`报告所有打开的文件描述的错误。在有多个写入器的情况下，所有写入器都会在后续的`fsync`中返回一个错误，即使通过特定文件描述符完成的所有写入都成功了(或者即使对该文件描述符根本没有写入)。

希望使用此基础结构的文件系统应该调用`mapping_set_error`，以便在发生错误时在`address_space`中记录错误。然后，在他们的`->fsync`操作中从页面缓存写回数据后，他们应该调用`file_check_and_advance_wb_err`来确保`struct file`的错误游标已经前进到由备份设备发出的错误流中的正确点。

### `struct address_space_operations`
这描述了VFS如何操作文件到文件系统中页面缓存的映射。定义了以下成员:
```c
struct address_space_operations {
    int (*writepage)(struct page *page, struct writeback_control *wbc);
    int (*readpage)(struct file *, struct page *);
    int (*writepages)(struct address_space *, struct writeback_control *);
    bool (*dirty_folio)(struct address_space *, struct folio *);
    void (*readahead)(struct readahead_control *);
    int (*write_begin)(struct file *, struct address_space *mapping,
            loff_t pos, unsigned len, unsigned flags, struct page **pagep, void **fsdata);
    int (*write_end)(struct file *, struct address_space *mapping,
            loff_t pos, unsigned len, unsigned copied, struct page *page, void *fsdata);
    sector_t (*bmap)(struct address_space *, sector_t);
    void (*invalidate_folio) (struct folio *, size_t start, size_t len);
    int (*releasepage) (struct page *, int);
    void (*freepage)(struct page *);
    ssize_t (*direct_IO)(struct kiocb *, struct iov_iter *iter);

    /* isolate a page for migration */
    bool (*isolate_page) (struct page *, isolate_mode_t);

    /* migrate the contents of a page to the specified target */
    int (*migratepage) (struct page *, struct page *);

    /* put migration-failed page back to right list */
    void (*putback_page) (struct page *);
    int (*launder_folio) (struct folio *);

    bool (*is_partially_uptodate) (struct folio *, size_t from, size_t count);
    void (*is_dirty_writeback) (struct page *, bool *, bool *);
    int (*error_remove_page) (struct mapping *mapping, struct page *page);
    int (*swap_activate)(struct file *);
    int (*swap_deactivate)(struct file *);
};
```

- `writepage`
由`VM`调用，将脏页写入后备存储。这可能是由于数据完整性原因(即“同步”)，或为了释放内存(刷新)。这种差异可以从`wbc->sync_mode`中看出。`PG_Dirty`标志已经清除，`pagellocked`为`true`。`writepage`应该启动`writeout`，应该设置`PG_Writeback`，并且应该确保在写操作完成时，页面是同步或异步解锁的。如果`wbc->sync_mode`为`WB_SYNC_NONE`，如果出现问题，`->writepage`处理起来不会太费劲；如果更容易的话，可以选择从映射中写出其他页面(例如由于内部依赖关系)。如果它选择不启动`writeout`，它应该返回`AOP_WRITEPAGE_ACTIVATE`，这样VM就不会一直调用该页上的`->writepage`。
> 有关更多细节，请参阅文件“Locking”。

- `readpage`
调用`VM`从后台存储读取页面。当`readpage`被调用时，该页面将被锁定，并且应该在读取完成后被解锁并标记为最新。如果`->readpage`发现由于某些原因需要解锁页面，它可以这样做，然后返回`AOP_TRUNCATED_PAGE`。在这种情况下，页面将被重新定位并重新锁定，如果所有操作都成功，则将再次调用`->readpage`。

- `writepages`
由`VM`调用，以写出与`address_space`对象关联的页面。如果`wbc->sync_mode`为`WB_SYNC_ALL`，则`writeback_control`将指定必须写入的页面范围。如果它是`WB_SYNC_NONE`，则给出一个`nr_to_write`，并且如果可能的话应该写入许多页。如果没有给出`->writpages`，则使用`mpage_writpages`。这将从标记为`DIRTY`的地址空间中选择页面，并将它们传递给`->writepage`。

- `dirty_folio`
`VM`调用，将对开本标记为`dirty`。如果地址空间将私有数据附加到一个 `fifo`，并且当 `fifo` 被污染时需要更新该数据，则特别需要这样做。例如，当内存映射页被修改时，就会调用这个函数。如果定义了，它应该在`i_pages`中设置`folio dirty`标志和`PAGECACHE_TAG_DIRTY`搜索标记。

- `readahead`
由`VM`调用，以读取与`address_space`对象关联的页。在页缓存中，页是连续的，并且被锁定。实现应该在启动每个页面的`I/O`后减少页面引用计数。通常，该页面将由`I/O`完成处理程序解锁。页面集被分为一些同步页面和一些异步页面，`rac->ra->async_size`给出了异步页面的数量。文件系统应该尝试读取所有同步页面，但可能在到达异步页面时决定停止。如果它决定停止尝试`I/O`，它可以简单地返回。调用者将从地址空间中删除剩余的页，解锁它们并减小页引用计数。如果`I/O`成功完成，设置`PageUptodate`。在任何页面上设置`PageError`都将被忽略;如果发生`I/O`错误，只需解锁页面。

- `write_begin`
- `write_end`
- `bmap`
- `invalidate_folio`
- `releasepage`
- `freepage`
- `direct_IO`
- `isolate_page`
- `migrate_page`
- `putback_page`
- `launder_folio`
- `is_partially_uptodate`
- `is_dirty_writeback`
- `error_remove_page`
- `swap_activate`
- `swap_deactivate`

## File 对象
文件对象表示进程打开的文件。在POSIX术语中，这也称为“打开文件描述”。

### `struct file_operations`
这描述了VFS如何操作打开的文件。在4.18内核中，定义了以下成员:
```c
struct file_operations {
        struct module *owner;
        loff_t (*llseek) (struct file *, loff_t, int);
        ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
        ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
        ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
        ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
        int (*iopoll)(struct kiocb *kiocb, bool spin);
        int (*iterate) (struct file *, struct dir_context *);
        int (*iterate_shared) (struct file *, struct dir_context *);
        __poll_t (*poll) (struct file *, struct poll_table_struct *);
        long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
        long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
        int (*mmap) (struct file *, struct vm_area_struct *);
        int (*open) (struct inode *, struct file *);
        int (*flush) (struct file *, fl_owner_t id);
        int (*release) (struct inode *, struct file *);
        int (*fsync) (struct file *, loff_t, loff_t, int datasync);
        int (*fasync) (int, struct file *, int);
        int (*lock) (struct file *, int, struct file_lock *);
        ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
        unsigned long (*get_unmapped_area)(struct file *, unsigned long,
                unsigned long, unsigned long, unsigned long);
        int (*check_flags)(int);
        int (*flock) (struct file *, int, struct file_lock *);
        ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
        ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
        int (*setlease)(struct file *, long, struct file_lock **, void **);
        long (*fallocate)(struct file *file, int mode, loff_t offset, loff_t len);
        void (*show_fdinfo)(struct seq_file *m, struct file *f);
#ifndef CONFIG_MMU
        unsigned (*mmap_capabilities)(struct file *);
#endif
        ssize_t (*copy_file_range)(struct file *, loff_t, struct file *, loff_t, size_t, unsigned int);
        loff_t (*remap_file_range)(struct file *file_in, loff_t pos_in, 
                struct file *file_out, loff_t pos_out, loff_t len, unsigned int remap_flags);
        int (*fadvise)(struct file *, loff_t, loff_t, int);
};
```
同样，调用所有方法时不会持有任何锁，除非另有说明。
- `llseek`
- `read`
- `read_iter`
- `write`
- `write_iter`
- `iopoll`
- `iterate`
- `iterate_shared`
- `poll`
- `unlocked_ioctl`
- `compat_ioctl`
- `mmap`
- `open`
- `flush`
- `release`
- `fsync`
- `fasync`
- `lock`
- `get_unmapped_area`
- `check_flags`
- `flock`
- `splice_write`
- `splice_read`
- `setlease`
- `fallocate`
- `copy_file_range`
- `remap_file_range`
- `fadvise`

注意，文件操作是由inode所在的特定文件系统实现的。当打开一个设备节点(特殊字符或块)时，大多数文件系统将调用VFS中的特殊支持例程，它将定位所需的设备驱动程序信息。这些支持例程将文件系统文件操作替换为设备驱动程序的操作，然后继续为文件调用新的open()方法。这就是在文件系统中打开设备文件最终如何调用设备驱动程序open()方法的。

## 目录条目缓存(dcache)
### `struct dentry_operations`
这描述了文件系统如何重载标准`dentry`操作。`dentry`和`dcache`是`VFS`和各个文件系统实现的域。设备驱动程序在这里没有什么用。这些方法可以设置为NULL，因为它们要么是可选的，要么VFS使用默认值。在2.6.22内核中，定义了以下成员:
```c
struct dentry_operations {
    int (*d_revalidate)(struct dentry *, unsigned int);
    int (*d_weak_revalidate)(struct dentry *, unsigned int);
    int (*d_hash)(const struct dentry *, struct qstr *);
    int (*d_compare)(const struct dentry *, unsigned int, const char *, const struct qstr *);
    int (*d_delete)(const struct dentry *);
    int (*d_init)(struct dentry *);
    void (*d_release)(struct dentry *);
    void (*d_iput)(struct dentry *, struct inode *);
    char *(*d_dname)(struct dentry *, char *, int);
    struct vfsmount *(*d_automount)(struct path *);
    int (*d_manage)(const struct path *, bool);
    struct dentry *(*d_real)(struct dentry *, const struct inode *);
};
```
- `d_revalidate`
- `_weak_revalidate`
- `d_hash`
- `d_compare`
- `d_delete`
- `d_init`
- `d_release`
- `d_iput`
- `d_dname`
- `d_automount`
- `d_manage`
- `d_real`

每个dentry都有一个指向其父dentry的指针，以及子dentry的散列列表。子dentry基本上就像目录中的文件。

### 目录条目缓存API(Directory Entry Cache API)
定义了许多允许文件系统操作dentry的函数:
- `dget`
为现有的dentry打开一个新的句柄(这只是增加了使用计数)
- `dput`
关闭`dentry`的句柄(减少使用计数)。如果使用计数下降到0, `dentry`仍然在其父散列中，则调用"`d_delete`"方法来检查是否应该缓存它。如果它不应该被缓存，或者`dentry`没有被散列，它就会被删除。否则，缓存的dentry将被放入`LRU`列表中，在内存不足时回收。
- `d_drop`
这将从其父哈希列表中解算dentry。如果dentry的使用计数下降到0，则对dput()的后续调用将释放该dentry
- `d_delete`
删除一个dentry。如果没有对dentry的其他开放引用，则dentry被转换为负dentry(调用`d_iput()`方法)。如果有其他引用，则调用`d_drop()`
- `d_add`
添加一个dentry到它的父哈希列表，然后调用`d_instantiate()`
- `d_instantiate`
将`dentry`添加到`inode`的别名散列列表中，并更新“`d_inode`”成员。inode结构中的“`i_count`”成员应该设置/增加。如果`inode`指针为NULL, `dentry`称为“`负dentry`”。当为现有的`负dentry`创建一个inode时，通常会调用这个函数
- `d_lookup`
它从dcache哈希表中查找指定名称的子元素。如果找到，引用计数将增加并返回dentry。调用者在使用完dentry后必须使用dput()来释放它。

## 挂载操作(Mount Options)
### 解析挂载配置(Parsing options)
在挂载和重新挂载时，将向文件系统传递一个字符串，其中包含以逗号分隔的挂载选项列表。选项可以有以下两种形式:`option` 和 `option=value`

`<linux/parser.h>`头文件定义了一个API来帮助解析这些选项。有很多关于如何在现有文件系统中使用它的示例。

### 显示挂载配置(Showing options)
如果文件系统接受挂载选项，它必须定义`show_options()`来显示所有当前活动的选项。规则是:
- 选项必须显示哪些不是默认值，或者它们的值与默认值不同
- 选项可以显示为默认启用或具有默认值

仅在装入帮助程序和内核之间内部使用的选项(如文件描述符)，或者仅在装入过程中起作用的选项(如控制日志创建的选项)不受上述规则的约束。

使用上述规则的根本原因是确保可以根据在`/proc/mounts.conf`. 中找到的信息准确地复制一个挂载(例如，卸载和重新挂载)
> 5.x 内核没找到 `/proc/mounts.conf`
