
```
    if (WARN_ON(!f->f_op)) {
        error = -ENODEV;
        goto cleanup_all;
    }

//...这里省略了很多代码

cleanup_all:
    if (WARN_ON_ONCE(error > 0))
        error = -EINVAL;

    fops_put(f->f_op);

    if (f->f_mode & FMODE_WRITER) {
        put_write_access(inode);
        __mnt_drop_write(f->f_path.mnt);
    }

cleanup_file:
    path_put(&f->f_path);
    f->f_path.mnt = NULL;
    f->f_path.dentry = NULL;
    f->f_inode = NULL;
    return error;
}
```

## 4> writev

主要要分析的是 `sys_writev`。

```
SYSCALL_DEFINE3(writev, unsigned long, fd, const struct iovec __user *, vec,
                unsigned long, vlen)
{
    return do_writev(fd, vec, vlen, 0);
}
```

深入这个函数来看看

```
static ssize_t do_writev(unsigned long fd, const struct iovec __user *vec,
                         unsigned long vlen, rwf_t flags)
{
    struct fd f = fdget_pos(fd);
    ssize_t ret = -EBADF;

    if (f.file) {
        loff_t pos, *ppos = file_ppos(f.file);

        if (ppos) {
            pos = *ppos;
            ppos = &pos;
        }

        ret = vfs_writev(f.file, vec, vlen, ppos, flags);

        if (ret >= 0 && ppos)
            f.file->f_pos = pos;

        fdput_pos(f);
    }

    if (ret > 0)
        add_wchar(current, ret);

    inc_syscw(current);

    return ret;
}
```

看起来其实非常简单，也就是先根据文件描述符去获取 `fd` 结构，`fd` 结构里面维护了当前打开的文件的写指针和读指针，第一步先获取，然后调用 `vfs_writev` 去写该文件，随后释放文件结构，如果返回值 `> 0`，则增加当前文件写入字符数（`add_wchar`），增加当前系统调用次数（`inc_syscw`）。

同样从头到尾来看看函数定义，首先是这个获取文件结构的 `fdget_pos`，

```
static inline struct fd fdget_pos(int fd)
{
    return __to_fd(__fdget_pos(fd));
}
```

然后再深入看看 `__to_fd` 和 `__fget_pos` 函数。

```
static inline struct fd __to_fd(unsigned long v)
{
    return (struct fd){(struct file *)(v & ~3),v & 3};
}
```

无疑 `__to_fd` 函数将获得的文件结构 `struct file` 转为 `struct fd`。

`__fdget_pos` 就应当是根据文件描述符来获取文件结构 `struct file`。

```
unsigned long __fdget_pos(unsigned int fd)
{
    unsigned long v = __fdget(fd);
    struct file *file = (struct file *)(v & ~3);

    if (file && (file->f_mode & FMODE_ATOMIC_POS)) {
        if (file_count(file) > 1) {
            v |= FDPUT_POS_UNLOCK;
            mutex_lock(&file->f_pos_lock);
        }
    }
    return v;
}
```

深入下去 `__fdget` 可以发现里面调用了 `__fget_light`，第二个参数被固定为 `FMODE_PATH`，对于这个函数定义：

```
/* File is opened with O_PATH; almost nothing can be done with it */
#define FMODE_PATH                ((__force fmode_t)0x4000)

unsigned long __fdget(unsigned int fd)
{
        return __fget_light(fd, FMODE_PATH);
}

/*
 * Lightweight file lookup - no refcnt increment if fd table isn't shared.
 *
 * You can use this instead of fget if you satisfy all of the following
 * conditions:
 * 1) You must call fput_light before exiting the syscall and returning control
 *    to userspace (i.e. you cannot remember the returned struct file * after
 *    returning to userspace).
 * 2) You must not call filp_close on the returned struct file * in between
 *    calls to fget_light and fput_light.
 * 3) You must not clone the current task in between the calls to fget_light
 *    and fput_light.
 *
 * The fput_needed flag returned by fget_light should be passed to the
 * corresponding fput_light.
 */
static unsigned long __fget_light(unsigned int fd, fmode_t mask)
{
    struct files_struct *files = current->files;
    struct file *file;

    if (atomic_read(&files->count) == 1) {
        file = files_lookup_fd_raw(files, fd);

        if (!file || unlikely(file->f_mode & mask))
            return 0;

        return (unsigned long)file;
    } else {
        file = __fget(fd, mask, 1);

        if (!file)
            return 0;

        return FDPUT_FPUT | (unsigned long)file;
    }
}
```

这里也解释了这个宏的定义，表示文件几乎不能做任何操作比如说 `READ`、`WRITE`，而这里的 `mask` 在后面分析是禁止的一些操作，比如文件具有 `READ` 权限但是 `mask` 被设置为 `FMODE_READ`，那么在后续的调用中会返回 `NULL`。

先获取当前进程的文件描述符表（`current->files`），然后判断文件描述符表的引用计数是否为 1（描述符表是否共享），如果是则调用 `files_lookup_fd_raw` 去获取文件结构指针，然后判断文件操作模式的正确性，随后返回。

```
/*
 * The caller must ensure that fd table isn't shared or hold rcu or file lock
 */
static inline struct file *files_lookup_fd_raw(struct files_struct *files, unsigned int fd)
{
    struct fdtable *fdt = rcu_dereference_raw(files->fdt);

    if (fd max_fds) {
        fd = array_index_nospec(fd, fdt->max_fds);
        return rcu_dereference_raw(fdt->fd[fd]);
    }
    return NULL;
}
```

根据注释也可以认为，需要保证文件描述符表没有被共享过，或者是持有文件锁。会返回一个 `fd` 表中的 `struct file` 结构（`fdt->fd[fd]`）。

如果引用计数不为 1，则调用 `__fget` 去获取指针，其中主要是调用了 `__fget_files` 函数。

```
#define get_file_rcu_many(x, cnt)        \
        atomic_long_add_unless(&(x)->f_count, (cnt), 0)

static struct file *__fget_files(struct files_struct *files, unsigned int fd,
                                 fmode_t mask, unsigned int refs)
{
    struct file *file;

    rcu_read_lock();
loop:
    file = files_lookup_fd_rcu(files, fd);

    if (file) {
        /* File object ref couldn't be taken.
                 * dup2() atomicity guarantee is the reason
                 * we loop to catch the new file (or NULL pointer)
                 */
        if (file->f_mode & mask)
            file = NULL;
        else if (!get_file_rcu_many(file, refs))
            goto loop;
    }

    rcu_read_unlock();

    return file;
}

static inline struct file *__fget(unsigned int fd, fmode_t mask,
                                  unsigned int refs)
{
    return __fget_files(current->files, fd, mask, refs);
}
```

这里的 `files_lookup_fd_rcu` 直接可以认为是获取文件结构体的，随后判断里面是否包含禁止的模式，然后增加文件计数引用（`get_file_rcu_many`）。

回过头来看看 `__fdget_pos` 函数

```
unsigned long __fdget_pos(unsigned int fd)
{
    unsigned long v = __fdget(fd);
    struct file *file = (struct file *)(v & ~3);

    if (file && (file->f_mode & FMODE_ATOMIC_POS)) {
        if (file_count(file) > 1) {
            v |= FDPUT_POS_UNLOCK;
            mutex_lock(&file->f_pos_lock);
        }
    }
    return v;
}
```

获取到的文件指针将最低两位置为 0（对齐），如果被设置了 `FMODE_ATOMIC_POS` 且文件引用大于 1，那么上锁，到这里，才分析完 `do_writev` 的第一句话，来看看接下来的语句，重点是 `vfs_writev` 函数。

```
static ssize_t do_iter_write(struct file *file, struct iov_iter *iter,
                loff_t *pos, rwf_t flags)
{
        size_t tot_len;
        ssize_t ret = 0;

        if (!(file->f_mode & FMODE_WRITE))
                return -EBADF;

        if (!(file->f_mode & FMODE_CAN_WRITE))
                return -EINVAL;

        tot_len = iov_iter_count(iter);

        if (!tot_len)
                return 0;

        ret = rw_verify_area(WRITE, file, pos, tot_len);

        if (ret f_op->write_iter)
                ret = do_iter_readv_writev(file, iter, pos, WRITE, flags);
        else
                ret = do_loop_readv_writev(file, iter, pos, WRITE, flags);

        if (ret > 0)
                fsnotify_modify(file);

        return ret;
}

static ssize_t vfs_writev(struct file *file, const struct iovec __user *vec,
                          unsigned long vlen, loff_t *pos, rwf_t flags)
{
    struct iovec iovstack[UIO_FASTIOV];
    struct iovec *iov = iovstack;
    struct iov_iter iter;
    ssize_t ret;

    ret = import_iovec(WRITE, vec, vlen, ARRAY_SIZE(iovstack), &iov, &iter);
    if (ret >= 0) {
        file_start_write(file);
        ret = do_iter_write(file, &iter, pos, flags);
        file_end_write(file);
        kfree(iov);
    }

    return ret;
}
```

首先根据 `writev` 的结构体解出数据和长度，然后调用 `do_iter_write` 去写文件，而在 `do_iter_write` 中可以发现，这里作权限校验了，校验了是否可写以及文件描述符是否可写，这里的两层意思分别是文件本身是否具有可写权限以及你打开的文件描述符是否包含了 `O_WRITE` 权限位。

随后进行写，写的过程会根据文件系统调用对应的写函数（`write_iter`）
