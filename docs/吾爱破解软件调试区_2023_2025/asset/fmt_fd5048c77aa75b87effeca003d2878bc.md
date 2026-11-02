
### `fsopen` 系统调用实现

```c
/*
 * 我们可以指定将在其中打开文件系统的容器，
 * 从而指示将使用哪些名称空间（特别是，哪个网络名称空间将用于网络文件系统）。
 */
```

```c
SYSCALL_DEFINE2(fsopen, const char __user *, _fs_name, unsigned int, flags)
{
        struct file_system_type *fs_type;
        struct fs_context *fc;
        const char *fs_name;
        int ret;
        if (!ns_capable(current->nsproxy->mnt_ns->user_ns, CAP_SYS_ADMIN))
                return -EPERM;
        if (flags & ~FSOPEN_CLOEXEC)
                return -EINVAL;
        fs_name = strndup_user(_fs_name, PAGE_SIZE);
        if (IS_ERR(fs_name))
                return PTR_ERR(fs_name);
        fs_type = get_fs_type(fs_name);                 //通过我们传入文件系统的名称来获取对应的fs_system_type
        kfree(fs_name);
        if (!fs_type)
                return -ENODEV;
        fc = fs_context_for_mount(fs_type, 0);         //通过fs_type来准备mount上下文
        ...
}
```

这里需要注意我们的`fsopen`并不是打开了一个位于硬盘上的具体的文件系统`on-disk`，而是打开了一个文件系统类型（`file_system_type`）

### `fs_context_for_mount`

其中后面会调用`fs_context_for_mount`函数，然后调用到`alloc_fs_context`，其中传入的参数如下：

```c
enum fs_context_purpose {
        FS_CONTEXT_FOR_MOUNT,                /* New superblock for explicit mount */
        FS_CONTEXT_FOR_SUBMOUNT,        /* New superblock for automatic submount */
        FS_CONTEXT_FOR_RECONFIGURE,        /* Superblock reconfiguration (remount) */
};

struct fs_context *fs_context_for_mount(struct file_system_type *fs_type,
                                        unsigned int sb_flags)
{
        return alloc_fs_context(fs_type, NULL, sb_flags, 0,
                                        FS_CONTEXT_FOR_MOUNT);
}
```

### `alloc_fs_context`

```c
/**
 * alloc_fs_context - Create a filesystem context.
 * @fs_type: The filesystem type.
 * @reference: The dentry from which this one derives (or NULL)
 * @sb_flags: Filesystem/superblock flags (SB_*)
 * @sb_flags_mask: Applicable members of @sb_flags
 * @purpose: The purpose that this configuration shall be used for.
 *
 * 打开文件系统并创建挂载上下文。
 * 挂载上下文使用提供的标志进行初始化，并且如果提供了来自另一个超级块（由 @reference 引用）的子挂载/自动挂载，
 * 则可能具有从该超级块复制的名称空间等参数。
 */
static struct fs_context *alloc_fs_context(struct file_system_type *fs_type,
                                      struct dentry *reference,
                                      unsigned int sb_flags,
                                      unsigned int sb_flags_mask,
                                      enum fs_context_purpose purpose)
{
        int (*init_fs_context)(struct fs_context *);
        struct fs_context *fc;
        int ret = -ENOMEM;
        fc = kzalloc(sizeof(struct fs_context), GFP_KERNEL_ACCOUNT);
        if (!fc)
                return ERR_PTR(-ENOMEM);
        fc->purpose        = purpose;
        fc->sb_flags        = sb_flags;
        fc->sb_flags_mask = sb_flags_mask;
        fc->fs_type        = get_filesystem(fs_type);
        fc->cred        = get_current_cred();
        fc->net_ns        = get_net(current->nsproxy->net_ns);
        fc->log.prefix        = fs_type->name;
        mutex_init(&fc->uapi_mutex);
        switch (purpose) {
        case FS_CONTEXT_FOR_MOUNT:
                fc->user_ns = get_user_ns(fc->cred->user_ns);
                break;
        case FS_CONTEXT_FOR_SUBMOUNT:
                fc->user_ns = get_user_ns(reference->d_sb->s_user_ns);
                break;
        case FS_CONTEXT_FOR_RECONFIGURE:
                atomic_inc(&reference->d_sb->s_active);
                fc->user_ns = get_user_ns(reference->d_sb->s_user_ns);
                fc->root = dget(reference);
                break;
        }
        /* TODO: Make all filesystems support this unconditionally */
        init_fs_context = fc->fs_type->init_fs_context;
        if (!init_fs_context)
                init_fs_context = legacy_init_fs_context;
        ret = init_fs_context(fc);
        if (ret need_free = true;
        return fc;
err_fc:
        put_fs_context(fc);
        return ERR_PTR(ret);
}
```

这里我们发现代码首先是利用`kzalloc`分配了一个`fs_context`结构体，也就是我们的一个文件系统上下文，如下：

### `fs_context` 结构体

```c
/*
 * 文件系统上下文，用于保存创建或重新配置超级块时使用的参数。
 *
 * Superblock creation fills in ->root whereas reconfiguration begins with this
 * already set.
 *
 * See Documentation/filesystems/mount_api.rst
 */
struct fs_context {
        const struct fs_context_operations *ops;
        struct mutex                uapi_mutex;        /* Userspace access mutex */
        struct file_system_type        *fs_type;          /* 文件系统类型 */
        void                        *fs_private;        /* 文件系统上下文 */
        void                        *sget_key;
        struct dentry                *root;                /* rooth */
        struct user_namespace        *user_ns;        /* 本次mount的用户命名空间 */
        struct net                *net_ns;        /* 本次mount的网络命名空间 */
        const struct cred        *cred;                /* The mounter's credentials */
        struct p_log                log;                /* 日志缓冲区 */
        const char                *source;        /* The source name (eg. dev path) */
        void                        *security;        /* Linux S&M options */
        void                        *s_fs_info;        /* Proposed s_fs_info */
        unsigned int                sb_flags;        /* Proposed superblock flags (SB_*) */
        unsigned int                sb_flags_mask;        /* Superblock flags that were changed */
        unsigned int                s_iflags;        /* OR'd with sb->s_iflags */
        unsigned int                lsm_flags;        /* Information flags from the fs to the LSM */
        enum fs_context_purpose        purpose:8;
        enum fs_context_phase        phase:8;        /* The phase the context is in */
        bool                        need_free:1;        /* Need to call ops->free() */
        bool                        global:1;        /* Goes into &init_user_ns */
        bool                        oldapi:1;        /* Coming from mount(2) */
};
```

看到上面代码后面有这样一段

```c
...
/* TODO: Make all filesystems support this unconditionally */
init_fs_context = fc->fs_type->init_fs_context;
if (!init_fs_context)
        init_fs_context = legacy_init_fs_context;
ret = init_fs_context(fc);
...
```

其中调用了我们`fs_type->init_fs_context`函数（但是该指针好像初始为0，所以一般`init_fs_context`会指向`legacy_init_fs_context`，该函数将会在之后漏洞介绍开始讲解），这是在新一代mount系统调用中增添的函数指针字段，之后就是初始化我们新创建的`fs_context`，然后返回给最初的`fsopen`系统调用

因此我们接着来分析后半段的`fsopen`

```c
...
put_filesystem(fs_type);
if (IS_ERR(fc))
        return PTR_ERR(fc);

fc->phase = FS_CONTEXT_CREATE_PARAMS;

ret = fscontext_alloc_log(fc);
if (ret 
```

首先他会调用`fscontext_alloc_log(fc)`，该函数的作用就是为该次文件系统上下文的`log`字段分配空间，然后将其中的`log`字段的`owner`指向当前文件系统类型`filesystem_type`的模块指针`owner`字段

之后我们调用`fscontext_create_fd`函数，其中存在以下调用链

```c
fscontext_create_fd(fc, flags & FSOPEN_CLOEXEC ? O_CLOEXEC : 0)
        anon_inode_getfd("[fscontext]", &fscontext_fops, fc, O_RDWR | o_flags)
                __anon_inode_getfd(name, fops, priv, flags, NULL, false)
```

其功能是为了返回一个同文件系统类型相联系的`fd`号，就跟咱们打开文件一样

### `__anon_inode_getfd`

```c
const struct file_operations fscontext_fops = {
        .read                = fscontext_read,
        .release        = fscontext_release,
        .llseek                = no_llseek,
};

static int __anon_inode_getfd(const char *name,
                              const struct file_operations *fops,
                              void *priv, int flags,                                         //这里priv就是之前创建的fscontext
                              const struct inode *context_inode,                 //为NULL，匿名inode
                              bool secure)
{
        int error, fd;
        struct file *file;

        error = get_unused_fd_flags(flags);         //获取空闲文件描述符
        if (error 
```

该函数整体的功能就是创建一个`[fscontext]`文件实例，然后返回`fd`号，这里也是同我们之前的`fscontext`创建了联系，创建联系的关键函数就是

```c
__anon_inode_getfile(name, fops, priv, flags, context_inode, secure)
```

该函数有这样一段代码
