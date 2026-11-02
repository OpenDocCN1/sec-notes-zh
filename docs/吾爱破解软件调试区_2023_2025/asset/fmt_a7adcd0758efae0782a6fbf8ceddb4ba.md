
```c
                            CGROUP_INET_INGRESS); \

                                          \

    __ret;                                    \

})
```

调用 `security_sock_rcv_skb` 函数，这是 LSM 的预留 hook 点，检查 socket 是否有权限接收 `skb`。

接下来，获取读锁，防止 `sk_filter` 被并发修改，从 `sk` 中获取 `sk_filter` 结构体指针。

取出 `sk->filter` 后，更新 `skb` 中的 `sock` 为当前传入 socket，并且调用 `bpf_prog_run_save_cb` 执行 bpf 程序。然后把 `skb->sk` 赋值旧的 socket 回去。

```c
rcu_read_lock();
filter = rcu_dereference(sk->sk_filter);
if (filter) {
    struct sock *save_sk = skb->sk;
    unsigned int pkt_len;

    skb->sk = sk;
    pkt_len = bpf_prog_run_save_cb(filter->prog, skb);
    skb->sk = save_sk;
    err = pkt_len ? pskb_trim(skb, max(cap, pkt_len)) : -EPERM;
}
rcu_read_unlock();
```

如果返回的长度不为 0，就调用 `pskb_trim` 函数，将 `skb` 的数据部分裁剪到 `cap` 和返回的长度中的较大值，如果裁剪失败，就返回错误码；如果返回的长度为 0，就将错误码设置为 `-EPERM`，表示要丢弃 packet。

### 3> `bpf_prog_run_save_cb`

bpf prog 执行的细节可以简单看一下。

其中涉及细节放到 BPF 系统源码分析里讲。

```c
static inline u32 bpf_prog_run_save_cb(const struct bpf_prog *prog,
                       struct sk_buff *skb)
{
    u32 res;

    migrate_disable();
    res = __bpf_prog_run_save_cb(prog, skb);
    migrate_enable();

    return res;
}

/* Must be invoked with migration disabled */
static inline u32 __bpf_prog_run_save_cb(const struct bpf_prog *prog,
                     const void *ctx)
{
    const struct sk_buff *skb = ctx;
    u8 *cb_data = bpf_skb_cb(skb);
    u8 cb_saved[BPF_SKB_CB_LEN];
    u32 res;

    if (unlikely(prog->cb_access)) {
        memcpy(cb_saved, cb_data, sizeof(cb_saved));
        memset(cb_data, 0, sizeof(cb_saved));
    }

    res = bpf_prog_run(prog, skb);
    if (unlikely(prog->cb_access))
        memcpy(cb_data, cb_saved, sizeof(cb_saved));

    return res;
}

static inline u8 *bpf_skb_cb(const struct sk_buff *skb)
{
    /* eBPF programs may read/write skb->cb[] area to transfer meta
     * data between tail calls. Since this also needs to work with
     * tc, that scratch memory is mapped to qdisc_skb_cb's data area.
     *
     * In some socket filter cases, the cb unfortunately needs to be
     * saved/restored so that protocol specific skb->cb[] data won't
     * be lost. In any case, due to unpriviledged eBPF programs
     * attached to sockets, we need to clear the bpf_skb_cb() area
     * to not leak previous contents to user space.
     */
    BUILD_BUG_ON(sizeof_field(struct __sk_buff, cb) != BPF_SKB_CB_LEN);
    BUILD_BUG_ON(sizeof_field(struct __sk_buff, cb) !=
             sizeof_field(struct qdisc_skb_cb, data));

    return qdisc_skb_cb(skb)->data;
}

static __always_inline u32 bpf_prog_run(const struct bpf_prog *prog, const void *ctx)
{
    return __bpf_prog_run(prog, ctx, bpf_dispatcher_nop_func);
}

static __always_inline u32 __bpf_prog_run(const struct bpf_prog *prog,
                      const void *ctx,
                      bpf_dispatcher_fn dfunc)
{
    u32 ret;

    cant_migrate();
    if (static_branch_unlikely(&bpf_stats_enabled_key)) {
        struct bpf_prog_stats *stats;
        u64 start = sched_clock();
        unsigned long flags;

        ret = dfunc(ctx, prog->insnsi, prog->bpf_func);
        stats = this_cpu_ptr(prog->stats);
        flags = u64_stats_update_begin_irqsave(&stats->syncp);
        u64_stats_inc(&stats->cnt);
        u64_stats_add(&stats->nsecs, sched_clock() - start);
        u64_stats_update_end_irqrestore(&stats->syncp, flags);
    } else {
        ret = dfunc(ctx, prog->insnsi, prog->bpf_func);
    }

    return ret;
}
```

## 2> 调用链分析

可以通过搜索 `sk->sk_filter`、`sk_filter`、`sk_filter_trim_cap`，分析 filter 程序的调用

### 3> `SOCKET_RAW`

查找函数引用，回溯一下调用链。

查找调用 `sk_filter` 的函数，定位到 `sock_queue_rcv_skb(net/core/sock.c)`（很多函数有注释，比如）

![](https://cdn.nlark.com/yuque/0/2024/png/27955958/1708273458963-aae9617d-a8c6-4dcd-9d6b-2f215052d051.png#averageHue=%23957c4e&clientId=u89b13094-32fc-4&from=paste&height=572&id=UadgV&originHeight=715&originWidth=1222&originalType=binary&ratio=1&rotation=0&showTitle=false&size=166520&status=done&style=none&taskId=ufc96b4ec-6c89-4b76-8f3e-d5276d6fab7&title=&width=977.6)

```c
int sock_queue_rcv_skb(struct sock *sk, struct sk_buff *skb)
{
    int err;

    err = sk_filter(sk, skb);
    if (err)
        return err;

    return __sock_queue_rcv_skb(sk, skb);
}
EXPORT_SYMBOL(sock_queue_rcv_skb);
```

代码中的 `sk_filter` 就是埋点函数。

代码中有非常多的调用，很多都是各种协议的适配，比如 J1939，搜索后发现是汽车的 CAN 总线通信协议。这里我们关注 `net/ieee802154/socket.c`。

可以看到，`dgram_rcv_skb`(数据报SOCKET)和 `raw_rcv_skb` 都调用了 `sock_queue_rcv_skb`。

```c
static int raw_rcv_skb(struct sock *sk, struct sk_buff *skb)
{
    skb = skb_share_check(skb, GFP_ATOMIC);
    if (!skb)
        return NET_RX_DROP;

    if (sock_queue_rcv_skb(sk, skb)
```

ipv4 的 `raw_rcv_skb` 逻辑也差不多，后面都进入同样的 `raw_rcv`

```c
static int raw_rcv_skb(struct sock *sk, struct sk_buff *skb)
{
    /* Charge it to the socket. */

    ipv4_pktinfo_prepare(sk, skb);
    if (sock_queue_rcv_skb(sk, skb)
```

跟进到 `net/ipv4/raw.c` 的 `raw_rcv`。

```c
int raw_rcv(struct sock *sk, struct sk_buff *skb)
{
    // 安全策略检查
    if (!xfrm4_policy_check(sk, XFRM_POLICY_IN, skb)) {
        atomic_inc(&sk->sk_drops);
        kfree_skb(skb);
        return NET_RX_DROP;
    }

    //NFHOOK埋点，重置跟踪信息
    nf_reset_ct(skb);
    skb_push(skb, skb->data - skb_network_header(skb));
    raw_rcv_skb(sk, skb);
    return 0;
}
```

跟进 `raw_v4_input`。这个函数主要做 `socket_raw` 的 RX 方向 `sk` 分配。`SOCKET_RAW` 允许多个 socket 同时接收同一个数据包，

```c
/* IP input processing comes here for RAW socket delivery.
 * Caller owns SKB, so we must make clones.
 *
 * RFC 1122: SHOULD pass TOS value up to the transport layer.
 * -> It does. And not only TOS, but all IP header.
 */
static int raw_v4_input(struct sk_buff *skb, const struct iphdr *iph, int hash)
{
    ......
    // 根据网络设备寻找匹配的socket
    net = dev_net(skb->dev);
    sk = __raw_v4_lookup(net, __sk_head(head), iph->protocol,
                 iph->saddr, iph->daddr, dif, sdif);

    while (sk) {
        delivered = 1;
        if ((iph->protocol != IPPROTO_ICMP || !icmp_filter(sk, skb)) &&
            ip_mc_sf_allow(sk, iph->daddr, iph->saddr,
                   skb->dev->ifindex, sdif)) {
            // clone的目的是不共享数据包，socket拥有自己的数据包
            struct sk_buff *clone = skb_clone(skb, GFP_ATOMIC);

            /* Not releasing hash table! */
            if (clone)
                raw_rcv(sk, clone);
        }
        sk = __raw_v4_lookup(net, sk_next(sk), iph->protocol,
                     iph->saddr, iph->daddr,
                     dif, sdif);
    }
out:
    read_unlock(&raw_v4_hashinfo.lock);
    return delivered;
}
```

跟进到 `raw_local_deliver`

```c
int raw_local_deliver(struct sk_buff *skb, int protocol)
{
    int hash;
    struct sock *raw_sk;

    // 根据协议获取哈希值，从raw_v4_hashinfo链表获得socket对象
    hash = protocol & (RAW_HTABLE_SIZE - 1);
    raw_sk = sk_head(&raw_v4_hashinfo.ht[hash]);

    /* If there maybe a raw socket we must check - if not we
     * don't care less
     */
    if (raw_sk && !raw_v4_input(skb, ip_hdr(skb), hash))
        raw_sk = NULL;

    return raw_sk != NULL;
}
```

跟进 `ip_local_deliver->ip_local_deliver_finish->ip_protocol_deliver_rcu->raw_local_deliver`，这就来到了网络层转发到传输层的函数入口了。`ip_local_deliver` 负责网络层转发到上层协议。由于 `SOCKET_RAW` 跳过传输层，因此检查设置在了这，具体细节可以看网络系统文章。

[//www.yuque.com/berryalen02/notebook/svy3a6ovccohupl1?view=doc_embed&inner=qnXIA" target="_blank" rel="noopener noreferrer nofollow"](https://www.yuque.com/berryalen02/notebook/svy3a6ovccohupl1?view=doc_embed&inner=qnXIA:rel="nofollow noopener" href="https)

### 3> `SOCKET_STREAM`

`net/ipv4/tcp_ipv4.c` 的 `tcp_filter` 函数调用了 `sk_filter_trim_cap`

```c
int tcp_filter(struct sock *sk, struct sk_buff *skb)
{
    struct tcphdr *th = (struct tcphdr *)skb->data;

    return sk_filter_trim_cap(sk, skb, th->doff * 4);
}
EXPORT_SYMBOL(tcp_filter);
```

跟进到 `tcp_v4_rcv`（`AF_INET_tcp` 的 `recv` 函数）

在 `TCP_NEW_SYN_RECV` 的处理逻辑以及主体函数逻辑中，都有 `tcp_filter` 的函数调用

`tcp_v4_rcv` 函数中会对 `TCP_NEW_SYN_RECV` 进行处理，如果连接检查成功，则需要新建控制块来处理连接，这个新建控制块的状态将会使用 `TCP_SYN_RECV` 状态；

```c
/*
 *  From tcp_input.c
 */
int tcp_v4_rcv(struct sk_buff *skb)
{
    struct net *net = dev_net(skb->dev);
```
