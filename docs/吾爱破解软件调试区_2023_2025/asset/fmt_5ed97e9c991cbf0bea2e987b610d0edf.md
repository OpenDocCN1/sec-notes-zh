
``` 
                switch (po->tp_version) {
                case TPACKET_V3:
                        /* Block transmit is not supported yet */
                        if (!tx_ring) {
                                init_prb_bdqc(po, rb, pg_vec, req_u);

                        ...

                }
```

`init_prb_bdqc`函数的功能是将环形缓冲区的参数复制到 `struct pack_ring_buffer.prb_bdqc`字段，根据计算设置参数，然后设置 block retire timer,最后调用 `prb_open_block`函数来初始化第一个 block

```
static void init_prb_bdqc(struct packet_sock *po,

                        struct packet_ring_buffer *rb,

                        struct pgv *pg_vec,

                        union tpacket_req_u *req_u)

{

        struct tpacket_kbdq_core *p1 = GET_PBDQC_FROM_RB(rb);

        struct tpacket_block_desc *pbd;

        memset(p1, 0x0, sizeof(*p1));

        p1->knxt_seq_num = 1;

        p1->pkbdq = pg_vec;

        pbd = (struct tpacket_block_desc *)pg_vec[0].buffer;

        p1->pkblk_start        = pg_vec[0].buffer;

        p1->kblk_size = req_u->req3.tp_block_size;

        p1->knum_blocks        = req_u->req3.tp_block_nr;

        p1->hdrlen = po->tp_hdrlen;

        p1->version = po->tp_version;

        p1->last_kactive_blk_num = 0;

        po->stats.stats3.tp_freeze_q_cnt = 0;

        if (req_u->req3.tp_retire_blk_tov)

                p1->retire_blk_tov = req_u->req3.tp_retire_blk_tov;

        else

                p1->retire_blk_tov = prb_calc_retire_blk_tmo(po,

                                                req_u->req3.tp_block_size);

        p1->tov_in_jiffies = msecs_to_jiffies(p1->retire_blk_tov);

        p1->blk_sizeof_priv = req_u->req3.tp_sizeof_priv;

        rwlock_init(&p1->blk_fill_in_prog_lock);

        p1->max_frame_len = p1->kblk_size - BLK_PLUS_PRIV(p1->blk_sizeof_priv);

        prb_init_ft_ops(p1, req_u);

        prb_setup_retire_blk_timer(po);

        prb_open_block(p1, pbd);

}
```

而 `prb_open_block()`函数中实现的一个功能就是将 `struct tpacket_kbdq_core->nxt_offset`设置在每个块私有区域之后

```
static void prb_open_block(struct tpacket_kbdq_core *pkc1,
        struct tpacket_block_desc *pbd1)
{
        ...

        pkc1->pkblk_start = (char *)pbd1;
        pkc1->nxt_offset = pkc1->pkblk_start + BLK_PLUS_PRIV(pkc1->blk_sizeof_priv);

        ...
}
```

以上我们就得到了一个内核当中以页为单位的分配原语，这有利于我们进行页级的堆风水

值得注意的是，在BitsByWill师傅的博客中并没有完全采取当初p0团队的分配方式，其中不同的点就是该博客当中使用的是 `TPACKET_V1`/`TPACKET_V2`，并且在创建 ring buffer的时候创建的是 `PACKET_TX_RING`类型的缓冲区，该缓冲区同 `PACKET_RX_RING`的区别在上面代码注释提了一嘴，但是这并不会影响我们的页分配，如下是我们 `packet_setsockopt()`部分源码

```
        case PACKET_RX_RING:
        case PACKET_TX_RING:
        {

                union tpacket_req_u req_u;

                int len;

                lock_sock(sk);

                switch (po->tp_version) {

                case TPACKET_V1:

                case TPACKET_V2:

                        len = sizeof(req_u.req);

                        break;

                case TPACKET_V3:

                default:

                        len = sizeof(req_u.req3);

                        break;

                }

                if (optlen 
```

可以看到这俩区别在分配的时候不大，最后仍是调用了 `packet_set_ring`函数，后面的步骤在上面也已经讲解就不多说了

这里需要额外提一下，因为我们直接是调用到了page分配这层，因此我们提供的是 `order`，而这个`order`是由我们最开始传入的 `struct tpacket_req_u` `req_u->req->tp_block_size`来决定的，根据 `packet_setsockopt()`进行分析不难得出这一点

而我们如果说要释放掉我们分配的 `block_nr`个相应 `order`的block,只需要简单的关闭对应 `socket`的`fd`指针即可,但是这里仍然会存在一个问题

> 
The only issue is that default low privileged users can’t utilize these functions in the root namespace

也就是说默认的低特权用户不能在root命名空间下用到上面这些函数，但我们通常可以在许多linux系统中创建自己的非特权用户命名空间。虽然说我们仍然可以通过大量堆喷某个数据结构来耗尽我们某个 `order`下`free_list`的block， 典型的例子就是常用的 `msg_msg`了，但他使用通用 `slab`进行分配不太可靠，最重要的是本题禁止用它了已经 :^(

## 初步构造思路

出题人给出了一个十分优雅的提权手法，那就是修改 `cred`,他的好处在于根本不用担心 `KASLR`,泄露地址，构造ROP链等等，我们都知道在现如今的版本， `cred`位于一个独立的 `kmem_cache`当中，名叫 `cred_jar`，我们可以首先耗尽该 `cred_jar`，因此未来他将使得分配器从伙伴系统 `order_0`处分配页面,并且将高阶 `order`的block进行拆解分配 ，这里我们需要free掉一部分，以免他们进行合并返回高阶 `order_*`，然后我们需要再次堆喷 `cred`,然后我们再次释放一些`pages`,之后再次堆喷攻击对象，并且至少要有一个攻击对象需要在 `cred`所在`slab`的正上方

那么该如何大量堆喷 `cred`结构体呢，我们只需要通过 `fork`大量创建子进程即可，虽说在 `fork`的过程中会产生大量噪音(分配过程中对其他无关对象的分配),但作者说这并不影响他们这个初始化堆喷的过程

## fork中噪声的处理

在构造页级堆风水的过程当中，对于内存的分配十分严格，因此我们需要尽量减少噪声对于我们的干扰

在 `fork`的过程当中，最为核心的函数就是 `kernel_clone`,我们需要牢记这一点，如果我们在传统 `fork`调用的过程当中没有设置任何 `kernel_clone_args`的`flag`参数，那么就会出现以下步骤：

`kernel_clone_args()`函数调用 `copy_process()`函数

`copy_process()`函数调用 `dup_task_struct()`函数，他将从目标内核系统中 `order_2`分配出一个 `task_struct`数据结构，然后 `dup_task_struct()` 调用 `alloc_thread_stack_node()`,如果说没有缓存栈可用的化，那么这个函数将使用 `__vmalloc_node_range`来分配一个虚拟连续的16kb区域来作为内核线程栈，这里通常会需要分配四个 `order_0`的`pages`

上面的 `vmalloc`将会分配一个 `kmalloc-64`的`chunk`来帮助建立vmalloc虚拟映射，然后，内核将会从 `vmap_area_cachep` 分配两个 `vmap_area` `chunk`。在这个系统和内核当中，第一个是从 `alloc_vmap_area`当中分配,第二个可能是从 `preload_this_cpu_lock`中分配

然后 `copy_process()`函数将会调用 `call_creds()`,他将会触发从 `prepare_creds()`中对于 `creds`的分配，这里如果设置了 `CLONE_THREAD`参数的化就不会发生这一步 

```
int copy_creds(struct task_struct *p, unsigned long clone_flags)
{
       struct cred *new;
       int ret;

#ifdef CONFIG_KEYS_REQUEST_CACHE
       p->cached_requested_key = NULL;
#endif

       if (
#ifdef CONFIG_KEYS
               !p->cred->thread_keyring &&
#endif
               clone_flags & CLONE_THREAD
           ) {
               p->real_cred = get_cred(p->cred);
               get_cred(p->cred);
               alter_cred_subscribers(p->cred, 2);
               kdebug("share_creds(%p{%d,%d})",
                      p->cred, atomic_read(&p->cred->usage),
                      read_cred_subscribers(p->cred));
               inc_rlimit_ucounts(task_ucounts(p), UCOUNT_RLIMIT_NPROC, 1);
               return 0;
       }

       new = prepare_creds();

...
```

在这之后 `copy_process()`函数开启了一系列 `copy_*()`函数，这个*是一系列进程所需要的资源，而这些函数便会触发内存的分配，除非设置了其各自的 `CLONE`标志，在平常的 `fork`当中，人们更希望从 `files_cache`,`fs_cache`,`sighand_cahe` 和 `signal_cache`当中分配新的`chunk`。其中最大的噪音是当没有设置 `CLONE_VM`标志位时建立`mm_struct`而产生的，而这反而会在 `vm_area_struct`,`anon_vma_chain`和 `anon_vma`等缓存当中触发大量内存分配活动，而这里所有分配都由该系统上的 `order_0`页面支持

```
if (retval)
               goto bad_fork_cleanup_audit;
       retval = copy_semundo(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_security;
       retval = copy_files(clone_flags, p, args->no_files);
       if (retval)
               goto bad_fork_cleanup_semundo;
       retval = copy_fs(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_files;
       retval = copy_sighand(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_fs;
       retval = copy_signal(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_sighand;
       retval = copy_mm(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_signal;
       retval = copy_namespaces(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_mm;
       retval = copy_io(clone_flags, p);
       if (retval)
               goto bad_fork_cleanup_namespaces;
       retval = copy_thread(p, args);
       if (retval)
               goto bad_fork_cleanup_io;
```

最后，内核将分配出一个 `pid` `chunk`，其中所属的 `slab`也来自于 `order_0`

下面便是忽略vmalloc，仅仅专注于 slab分配的情况下，单一一个 `fork`调用在这个系统当中将会触发的分配

```
task_struct
kmalloc-64
vmap_area
vmap_area
cred_jar
files_cache
fs_cache
sighand_cache
signal_cache
mm_struct
vm_area_struct
vm_area_struct
vm_area_struct
vm_area_struct
anon_vma_chain
anon_vma
anon_vma_chain
vm_area_struct
anon_vma_chain
anon_vma
anon_vma_chain
vm_area_struct
anon_vma_chain
anon_vma
anon_vma_chain
vm_area_struct
anon_vma_chain
anon_vma
anon_vma_chain
vm_area_struct
anon_vma_chain
anon_vma
anon_vma_chain
vm_area_struct
vm_area_struct
pid
```

可以看到确实产生了很多噪音，在经过 BitsByWill师傅分析源码还有查看clone手册的努力下(bushi，使用以下的`flag`能极大的降低`fork`当中产生的噪音：

```
CLONE_FILES | CLONE_FS | CLONE_VM | CLONE_SIGHAND
```

当设置了这些`flags`之后，我们产生的噪音将会降低至下述情况
