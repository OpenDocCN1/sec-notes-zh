
# CVE-2022-0847 分析

## 第一部分

```c
static ssize_t
    pipe_read(struct kiocb *iocb, struct iov_iter *to)
{
    size_t total_len = iov_iter_count(to);
    struct file *filp = iocb->ki_filp;
    struct pipe_inode_info *pipe = filp->private_data;
    bool was_full, wake_next_reader = false;
    ssize_t ret;

    /* Null read succeeds. */
    if (unlikely(total_len == 0))
        return 0;

    ret = 0;
    __pipe_lock(pipe);

    /*
     * We only wake up writers if the pipe was full when we started
     * reading in order to avoid unnecessary wakeups.
     *
     * But when we do wake up writers, we do so using a sync wakeup
     * (WF_SYNC), because we want them to get going and generate more
     * data for us.
     */
    was_full = pipe_full(pipe->head, pipe->tail, pipe->max_usage);
    //...
}
```

依然先特判读取长度是否为 0。然后锁上管道，判断管道是否满。

## 第二部分

```c
static ssize_t

    pipe_read(struct kiocb *iocb, struct iov_iter *to)

{

    for (;;) {

        unsigned int head = pipe->head;

        unsigned int tail = pipe->tail;

        unsigned int mask = pipe->ring_size - 1;

        #ifdef CONFIG_WATCH_QUEUE

        if (pipe->note_loss) {

            struct watch_notification n;

            if (total_len note_loss = false;

        }

        #endif

        if (!pipe_empty(head, tail)) {

            struct pipe_buffer *buf = &pipe->bufs[tail & mask];

            size_t chars = buf->len;

            size_t written;

            int error;

            if (chars > total_len) {

                if (buf->flags & PIPE_BUF_FLAG_WHOLE) {

                    if (ret == 0)

                        ret = -ENOBUFS;

                    break;

                }

                chars = total_len;

            }

            error = pipe_buf_confirm(pipe, buf);

            if (error) {

                if (!ret)

                    ret = error;

                break;

            }

            written = copy_page_to_iter(buf->page, buf->offset, chars, to);

            if (unlikely(written offset += chars;

            buf->len -= chars;

            /* Was it a packet buffer? Clean up and exit */

            if (buf->flags & PIPE_BUF_FLAG_PACKET) {

                total_len = chars;

                buf->len = 0;

            }

            if (!buf->len) {

                pipe_buf_release(pipe, buf);

                spin_lock_irq(&pipe->rd_wait.lock);

                #ifdef CONFIG_WATCH_QUEUE

                if (buf->flags & PIPE_BUF_FLAG_LOSS)

                    pipe->note_loss = true;

                #endif

                tail++;

                pipe->tail = tail;

                spin_unlock_irq(&pipe->rd_wait.lock);

            }

            total_len -= chars;

            if (!total_len)

                break;  /* common path: read succeeded */

            if (!pipe_empty(head, tail))    /* More to do? */

                continue;

        }

        if (!pipe->writers)

            break;

        if (ret)

            break;

        if (filp->f_flags & O_NONBLOCK) {

            ret = -EAGAIN;

            break;

        }

        __pipe_unlock(pipe);

        /*

         * We only get here if we didn't actually read anything.

         *

         * However, we could have seen (and removed) a zero-sized

         * pipe buffer, and might have made space in the buffers

         * that way.

         *

         * You can't make zero-sized pipe buffers by doing an empty

         * write (not even in packet mode), but they can happen if

         * the writer gets an EFAULT when trying to fill a buffer

         * that already got allocated and inserted in the buffer

         * array.

         *

         * So we still need to wake up any pending writers in the

         * _very_ unlikely case that the pipe was full, but we got

         * no data.

         */

        if (unlikely(was_full))

            wake_up_interruptible_sync_poll(&pipe->wr_wait, EPOLLOUT | EPOLLWRNORM);

        kill_fasync(&pipe->fasync_writers, SIGIO, POLL_OUT);

        /*

         * But because we didn't read anything, at this point we can

         * just return directly with -ERESTARTSYS if we're interrupted,

         * since we've done any required wakeups and there's no need

         * to mark anything accessed. And we've dropped the lock.

         */

        if (wait_event_interruptible_exclusive(pipe->rd_wait, pipe_readable(pipe)) head, pipe->tail, pipe->max_usage);

        wake_next_reader = true;

    }

    //...

}
```

紧接着跟一个 `for` 循环，中间包着一个宏可以不用管，这个应该没有编译进去。

先判是否非空，如果非空说明可以读数据，这里也不赘述取尾部缓冲区的操作了，不明白可以看前面的详细讲解。取出来之后呢，判断，如果该缓冲区数据的长度大于读取字节数 `chars > total_len`，那么说明这个缓冲区的数据已经够了，读取所需要的字节数就不需要管了，中间判一个标记字段，如果设置了这个标记且还没有读过数据（`ret==0`）那么报错退出，如果仅仅设置了标记，那么就直接 `break`。若没设置，则将取出的缓冲区长度设置为读取长度 `chars=total_len`。

随后再判一下缓冲区内容是否 good，将整个缓冲区拷贝给用户。若 `total_len>chars`，则拷贝 `chars`（缓冲区长度），否则把 `chars` 设置为要读取的总长度并读取这么多。然后设置一下缓冲区的 `offset` 和 `len`，其实这里差不多可以理解 `offset` 和 `len` 字段的具体含义了。

- `len`：缓冲区还有多少数据。

- `offset`：缓冲区的数据从哪里开始。

紧接着判，如果是一个分组管道，也就是网络管道，那么清除整个页并直接退出。

后面判断，如果读取完成之后缓冲区没数据了（同时，网络管道在这里设置为 0 就一定可以进这个分支），那么直接清除这个页。随后上读锁，让 `tail++`。如果没有要读取数据了，那么退出，随后看看如果还有要读取数据且管道不为空，则继续循环（`continue`）。

之后的分支都是处理管道为空的情况，不管是经历了一次读或者没有经历过读，都到这里来。判断写者还在不在，如果不在那么退循环，已经读取了若干数据也退循环，主要是没有经历过读 `ret==0`，那么在这里就一定能要给它一点数据，倘若管道设置为不能阻塞 `O_NONBLOCK`，那么直接返回错误。

后面的其实跟写差不多，但是这里有点不太明白，如果到这里没有读出数据的话，大概率是管道为空。那么需要唤醒写者来写些数据上去。如果管道满，那么唤醒下一个读者。

## 第三部分

```c
static ssize_t
    pipe_read(struct kiocb *iocb, struct iov_iter *to)
{
    //...
    if (pipe_empty(pipe->head, pipe->tail))
        wake_next_reader = false;
    __pipe_unlock(pipe);

    if (was_full)
        wake_up_interruptible_sync_poll(&pipe->wr_wait, EPOLLOUT | EPOLLWRNORM);
    if (wake_next_reader)
        wake_up_interruptible_sync_poll(&pipe->rd_wait, EPOLLIN | EPOLLRDNORM);
    kill_fasync(&pipe->fasync_writers, SIGIO, POLL_OUT);
    if (ret > 0)
        file_accessed(filp);
    return ret;
}
```

如果管道为空，那么不唤醒下一个读者。最后就是判断如果读取字节大于 0，那么更新文件访问时间。

## 总结

- 遍历循环队列的列，如果能全读那么全读整个页并释放该页数据，并继续下个页的遍历。

- 长度不足以读完整个页，那么读部分直接退出。

- 根据管道情况选择唤醒读者或者写者。

## EXP 重写方案

前面写的 EXP 虽然能成功，但是还不够完善，因为经过调试之后，发现 `set_can_merge` 这里不需要遍历整个 `pipe` 缓冲区，正如前面分析的，如果管道为空或者读取字节为 `PAGE_SIZE` 整数倍，那么都直接复制页，不选择续写，所以我们连着读写 16 次就可以给所有的缓冲区打上 `PIPE_BUF_FLAG_CAN_MERGE` 标志。

```c
#define _GNU_SOURCE

#include 

#include 

#include 

#include 

#include 

#include 

#include 

#ifndef PAGE_SIZE

#define PAGE_SIZE 4096

#endif

#define PIPE_SIZE 16

void SetCanMerge(int fd[2]){

    char buf;

    pipe(fd);

    for(int i=0;i
```

## `splice` 分析

同样使用上述 EXP，等到执行 `splice(fd,NULL,pipefd[1],NULL,1,0)` 。

这里我们在 `do_splice_to` 函数下断点，然后直接运行，`continue` 直接走完 16 次的读写。

最后调试的时候可以发现

![](https://xia0ji233.pro/2024/02/02/CVE-2022-0847/9.png)

这里一部分将文件页挂在管道上。

这里发现管道的 `head` 值已经是 17 了，说明文件页挂在了 16 的位置（也就是 0 ）。

![](https://xia0ji233.pro/2024/02/02/CVE-2022-0847/10.png)

此时发现，管道不为空，写入字节不为 `PAGE_SIZE` 整数倍，因此进入这个 `if` 分支。

![](https://xia0ji233.pro/2024/02/02/CVE-2022-0847/11.png)

此时因为前面的填充，导致这个缓冲区一直带着 `PIPE_BUF_FLAG_CAN_MERGE` 属性，并且在刚刚的 `splice` 当中并没有消除这个位。因此我们写可以跟着这个文件页后面写。

![](https://xia0ji233.pro/2024/02/02/CVE-2022-0847/12.png)

这里也是可以看到文件成功被写入了。

我们创造了一个没有密码的 `uid=0` 的用户，所以可以直接 `su` 切换。

## 总结

这个漏洞利用的前提是：文件是可读的，我们需要通过零拷贝，把文件页挂在管道上，并且文件第一个字节不可写，因为我们需要至少一个字节的拷贝才能让文件页挂在管道缓冲区。随后写入字节不能为 `PAGE_SIZE` 的整数倍，否则不能续写。

所以这个漏洞基本可以认为是，只要具有读权限，基本就可以写这个文件了，因此我们修改 `passwd` 配置提权。

## 修复方案

究其原因，`pipe` 缓冲区在使用的时候，没有初始化 `flag` 字段，这个问题在很早版本前就引入了，到 Linux 5.8 版本后，将各种类型的 `pipe` 缓冲区结构体合并之后，就引入了这个 bug。

修复这个问题只需要初始化 `flag` 字段即可，可以在 github 的提交上看到这次 [`github.com/torvalds/linux/commit/9d2231c5d74e13b2a0546fee6737ee4446017903`](https://github.com/torvalds/linux/commit/9d2231c5d74e13b2a0546fee6737ee4446017903)。

![](https://xia0ji233.pro/2024/02/02/CVE-2022-0847/13.png)

P.S，才刚学会 Kernel 调试环境的搭建，分析的肯定有不周到的地方，对于存在的问题，还请师傅们多多指出。

本篇分析于过年回家的路上完成，这路太堵了（哭）。

![](https://xia0ji233.pro/2024/02/02/CVE-2022-0847/14.png)

希望能平安到家，也祝各位师傅们新年快乐。
