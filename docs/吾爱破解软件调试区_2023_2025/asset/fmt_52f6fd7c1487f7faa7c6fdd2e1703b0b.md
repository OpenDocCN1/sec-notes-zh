
## TCP 相关处理

```c
struct sk_buff *skb_to_free;

const struct iphdr *iph;

const struct tcphdr *th;

struct sock *sk;

......

.........

th = (const struct tcphdr *)skb->data;

iph = ip_hdr(skb);

// 获取等待tcp包的合适socket

lookup:
sk = __inet_lookup_skb(&tcp_hashinfo, skb, __tcp_hdrlen(th), th->source,
 th->dest, sdif, &refcounted);

if (!sk)
 goto no_tcp_socket;

process:
if (sk->sk_state == TCP_TIME_WAIT)
 goto do_time_wait;
// 进入TCP_NEW_SYN_RECV逻辑
if (sk->sk_state == TCP_NEW_SYN_RECV) {
 .........
 if (!tcp_filter(sk, skb)) {
 th = (const struct tcphdr *)skb->data;
 iph = ip_hdr(skb);
 tcp_v4_fill_cb(skb, iph, th);
 nsk = tcp_check_req(sk, skb, req, false, &req_stolen);
 }
 .........
 if (tcp_filter(sk, skb)) {
 drop_reason = SKB_DROP_REASON_SOCKET_FILTER;
 goto discard_and_relse;
 }
 .....
 if (!sock_owned_by_user(sk)) {
 skb_to_free = sk->sk_rx_skb_cache;
 sk->sk_rx_skb_cache = NULL;
 ret = tcp_v4_do_rcv(sk, skb);
 } else {
 if (tcp_add_backlog(sk, skb))
 goto discard_and_relse;
 skb_to_free = NULL;
 }
 .........
 switch (tcp_timewait_state_process(inet_twsk(sk), skb, th)) {
 case TCP_TW_SYN: {
 struct sock *sk2 = inet_lookup_listener(dev_net(skb->dev),
 &tcp_hashinfo, skb,
 __tcp_hdrlen(th),
 iph->saddr, th->source,
 iph->daddr, th->dest,
 inet_iif(skb),
 sdif);
 if (sk2) {
 inet_twsk_deschedule_put(inet_twsk(sk));
 sk = sk2;
 tcp_v4_restore_cb(skb);
 refcounted = false;
 goto process;
 }
 }
 /* to ACK */
 fallthrough;
 case TCP_TW_ACK:
 tcp_v4_timewait_ack(sk, skb);
 break;
 case TCP_TW_RST:
 tcp_v4_send_reset(sk, skb);
 inet_twsk_deschedule_put(inet_twsk(sk));
 goto discard_it;
 case TCP_TW_SUCCESS;;
 }
 goto discard_it;
}
```

## SOCKET_DGRAM

搜索`sk_filter`找到了`udp_queue_rcv_one_skb`函数。这个函数位于`udp_queue_rcv_skb`内部

```c
static int udp_queue_rcv_skb(struct sock *sk, struct sk_buff *skb)
{
 struct sk_buff *next, *segs;
 int ret;

 if (likely(!udp_unexpected_gso(sk, skb)))
 return udp_queue_rcv_one_skb(sk, skb);
 ......
}
```

逆向一路向上跟进至`udp_rcv`，可知检测逻辑在UDP协议栈rcv处理函数内部。从`udp_rcv`顺序分析

```c
int udp_rcv(struct sk_buff *skb)
{
 return __udp4_lib_rcv(skb, &udp_table, IPPROTO_UDP);
}
```

跟进`__udp4_lib_rcv`

```c
/*
 * All we need to do is get the socket, and then do a checksum.
 */

int __udp4_lib_rcv(struct sk_buff *skb, struct udp_table *udptable,
 int proto)
{
 struct sock *sk;
 struct udphdr *uh;
 struct rtable *rt = skb_rtable(skb);
 __be32 saddr, daddr;
 struct net *net = dev_net(skb->dev);
 ......

 sk = skb_steal_sock(skb, &refcounted);
 if (sk) {
 struct dst_entry *dst = skb_dst(skb);
 int ret;

 if (unlikely(rcu_dereference(sk->sk_rx_dst) != dst))
 udp_sk_rx_dst_set(sk, dst);

 ret = udp_unicast_rcv_skb(sk, skb, uh);
 if (refcounted)
 sock_put(sk);
 return ret;
 }

 if (rt->rt_flags & (RTCF_BROADCAST|RTCF_MULTICAST))
 return __udp4_lib_mcast_deliver(net, skb, uh,
 saddr, daddr, udptable, proto);

 sk = __udp4_lib_lookup_skb(skb, uh->source, uh->dest, udptable);
 if (sk)
 return udp_unicast_rcv_skb(sk, skb, uh);
 ......
}
```

`udp_unicast_rcv_skb`和`__udp4_lib_mcast_deliver`都调用了`udp_queue_rcv_skb`，而`udp_queue_rcv_skb`内部包含`udp_queue_rcv_one_skb`。

最后在`udp_queue_rcv_one_skb`中调用了`sk_filter_trim_cap`。

```c
/* returns:
 * -1: error
 * 0: success
 * >0: "udp encap" protocol resubmission
 *
 * Note that in the success and error cases, the skb is assumed to
 * have either been requeued or freed.
 */

static int udp_queue_rcv_one_skb(struct sock *sk, struct sk_buff *skb)
{
 struct udp_sock *up = udp_sk(sk);
 int is_udplite = IS_UDPLITE(sk);

 /*
 * Charge it to the socket, dropping if the queue is full.
 */
 if (!xfrm4_policy_check(sk, XFRM_POLICY_IN, skb))
 goto drop;
 nf_reset_ct(skb);
 ......

 if (sk_filter_trim_cap(sk, skb, sizeof(struct udphdr)))
 goto drop;
 ......
}
```

调用链：`udp_rcv->__udp4_lib_rcv->udp_unicast_rcv_skb/__udp4_lib_mcast_deliver->udp_queue_rcv_skb->udp_queue_rcv_one_skb->sk_filter_trim_cap`

## 其他协议

还有一些内核函数也调用`sk_filter`相关函数，但是属于通用`sock`处理逻辑（`__sk_receive_skb`），一些其他协议使用，比如`DCCP`、`pppoe`、`l2tp`等，这里就不加以分析。

## SOCKET 析构逻辑

跟踪点是`__sk_destruct(net/core/sock.c)`，其中会检查`sock_filter`是否还存在，还存在的话调用`sk_filter_uncharge`删除分配的内存。

```c
/* Sockets having SOCK_RCU_FREE will call this function after one RCU
 * grace period. This is the case for UDP sockets and TCP listeners.
 */
static void __sk_destruct(struct rcu_head *head)
{
 struct sock *sk = container_of(head, struct sock, sk_rcu);
 struct sk_filter *filter;

 if (sk->sk_destruct)
 sk->sk_destruct(sk);

 filter = rcu_dereference_check(sk->sk_filter,
 refcount_read(&sk->sk_wmem_alloc) == 0);
 if (filter) {
 sk_filter_uncharge(sk, filter);
 RCU_INIT_POINTER(sk->sk_filter, NULL);
 }
 ......
#ifdef CONFIG_BPF_SYSCALL
 bpf_sk_storage_free(sk);
#endif
 ......
 sk_prot_free(sk->sk_prot_creator, sk);
}
```

跟进到`sk_destruct`

```c
void sk_destruct(struct sock *sk)
{
 bool use_call_rcu = sock_flag(sk, SOCK_RCU_FREE);
 if (rcu_access_pointer(sk->sk_reuseport_cb)) {
 reuseport_detach_sock(sk);
 use_call_rcu = true;
 }
 if (use_call_rcu)
 call_rcu(&sk->sk_rcu, __sk_destruct);
 else
 __sk_destruct(&sk->sk_rcu);
}
```

跟进`__sk_free`

```c
static void __sk_free(struct sock *sk)
{
 if (likely(sk->sk_net_refcnt))
 sock_inuse_add(sock_net(sk), -1);

 if (unlikely(sk->sk_net_refcnt && sock_diag_has_destroy_listeners(sk)))
 sock_diag_broadcast_destroy(sk);
 else
 sk_destruct(sk);
}
```

跟进至`sk_free`

```c
void sk_free(struct sock *sk)
{
 /*
 * We subtract one from sk_wmem_alloc and can know if
 * some packets are still in some tx queue.
 * If not null, sock_wfree() will call __sk_free(sk) later
 */
 if (refcount_dec_and_test(&sk->sk_wmem_alloc))
 __sk_free(sk);
}
EXPORT_SYMBOL(sk_free);
```

`sk_free`是内核删除`socket`对象的函数。内核通过`sk_alloc`分配`socket`对象。以下为`tipc_sk_create(net/tipc/socket.c)`的示例。通过`sk_alloc`创建`socket`对象，然后判断创建失败，`sk_free`释放内存。

```c
/* Allocate socket's protocol area */
// sk_alloc - All socket objects are allocated here
sk = sk_alloc(net, AF_TIPC, GFP_KERNEL, &tipc_proto, kern);
if (sk == NULL)
 return -ENOMEM;

tsk = tipc_sk(sk);
tsk->max_pkt = MAX_PKT_DEFAULT;
tsk->maxnagle = 0;
tsk->nagle_start = NAGLE_START_INIT;
INIT_LIST_HEAD(&tsk->publications);
INIT_LIST_HEAD(&tsk->cong_links);
msg = &tsk->phdr;

/* Finish initializing socket data structures */
sock->ops = ops;
sock_init_data(sock, sk);
tipc_set_sk_state(sk, TIPC_OPEN);
if (tipc_sk_insert(tsk)) {
 sk_free(sk);
 pr_warn("Socket create failed; port number exhausted\n");
 return -EINVAL;
}
```

处理逻辑调用链：`sk_free->sk_free->sk_destruct->sk_destruct`

## SOCKET_PACKET

`net/packet/af_packet.c`的`run_filter`会取出`sk->sk_filter->prog`程序，`bpf`执行

```c
static unsigned int run_filter(struct sk_buff *skb,
 const struct sock *sk,
 unsigned int res)
{
 struct sk_filter *filter;
 rcu_read_lock();
 filter = rcu_dereference(sk->sk_filter);
 if (filter != NULL)
 res = bpf_prog_run_clear_cb(filter->prog, skb);
 rcu_read_unlock();
 return res;
}
```

跟进到`packet_rcv`
