
```
                        )),

    this.internalExt = e,

    this.cursor = t;

    //下面才是核心

    const a = new p.PushFrame;

    a.setPayloadType("ack"),

    a.setPayload(

    //自执行：相当于把internalext传进去处理然后当成是payload的值

    function(e) {

           const t = [];

           for (const i of e) {

           const e = i.charCodeAt(0);

           e > 6)),

           t.push(128 + (63 & e))) : e > 12)),

           t.push(128 + (e >> 6 & 63)),

           t.push(128 + (63 & e)))

      }

       return Uint8Array.from(t)

    }(e)

    ), 

    a.setLogid(i.getLogid()),

    this.client.socket.send(a.serializeBinary())

}
```

最后

```
if ("msg" === i.getPayloadType() && (this.info("fetchSocketServer socket response: ", (()=>o.toObject())),
                this.emit(o)),//这里就是把值展示出来了，里面处理值
                "close" === i.getPayloadType())
                return t(new Error("close by payloadtype"))
}
```

大概理清楚了`onmessage`的思路，先反序列化得到我们要的数据给后面展示，`o`就是我们要的数据对象，然后如果需要`ack`就发，最后通过`emit`函数处理展示值。

我们看看实际情况：

![](https://s2.loli.net/2023/05/18/i6gXWON5oB8uyQY.png#id=KcY3J&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

所以先通过`n.ec`函数处理，然后再来反序列化。

我们复制刚刚返回的，也能得到这个序列。

![](https://s2.loli.net/2023/05/18/TCzSxDjnkXgKqJs.png#id=R8c3r&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://s2.loli.net/2023/05/18/pztSkAN3X5ULwlx.png#id=RtWWW&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

我们进去`n.ec`函数，然后移到最上面，发现是`webpack`打包的，然后我们可以看到

![](https://s2.loli.net/2023/05/18/aqvMALj5JWKSgF1.png#id=so3fC&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

![](https://s2.loli.net/2023/05/18/AI2HPZb1ELp3vkB.png#id=clPzP&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

刚刚好`Qe`就是我们进来的位置，也就是这个函数，其实名字已经暴露了是解压，我们也可以把代码扣下来使用。

去掉开头结尾就好了：

```
function i(t) {

    let e = t.length;

    for (; --e >= 0; )

        t[e] = 0

}

const n = 256

  , r = 286

  , s = 30

  , l = 15

  , o = new Uint8Array([0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3, 4, 4, 4, 4, 5, 5, 5, 5, 0])

  , h = new Uint8Array([0, 0, 0, 0, 1, 1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13])

  , d = new Uint8Array([0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 3, 7])

  , _ = new Uint8Array([16, 17, 18, 0, 8, 7, 9, 6, 10, 5, 11, 4, 12, 3, 13, 2, 14, 1, 15])

  , f = new Array(576);

i(f);

const c = new Array(60);

i(c);

const u = new Array(512);

i(u);

const w = new Array(256);

i(w);

const b = new Array(29);

i(b);

const g = new Array(s);

function m(t, e, a, i, n) {

    this.static_tree = t,

    this.extra_bits = e,

    this.extra_base = a,

    this.elems = i,

    this.max_length = n,

    this.has_stree = t && t.length

}

let p, k, v;

function y(t, e) {

    this.dyn_tree = t,

    this.max_code = 0,

    this.stat_desc = e

}

i(g);

const x = t=>t >> 7)]

  , z = (t,e)=>{

    t.pending_buf[t.pending++] = 255 & e,

    t.pending_buf[t.pending++] = e >>> 8 & 255

}

  , A = (t,e,a)=>{

    t.bi_valid > 16 - a ? (t.bi_buf |= e > 16 - t.bi_valid,

    t.bi_valid += a - 16) : (t.bi_buf |= e {

    A(t, a[2 * e], a[2 * e + 1])

}

  , R = (t,e)=>{

    let a = 0;

    do {

        a |= 1 & t,

        t >>>= 1,

        a  0);

    return a >>> 1

}

  , Z = (t,e,a)=>{

    const i = new Array(16);

    let n, r, s = 0;

    for (n = 1; n {

    let e;

    for (e = 0; e {

    t.bi_valid > 8 ? z(t, t.bi_buf) : t.bi_valid > 0 && (t.pending_buf[t.pending++] = t.bi_buf),

    t.bi_buf = 0,

    t.bi_valid = 0

}

  , D = (t,e,a,i)=>{

    const n = 2 * e

      , r = 2 * a;

    return t[n] {

    const i = t.heap[a];

    let n = a {

    let i, r, s, l, d = 0;

    if (0 !== t.last_lit)

        do {

            i = t.pending_buf[t.d_buf + 2 * d] {

    const a = e.dyn_tree

      , i = e.stat_desc.static_tree

      , n = e.stat_desc.has_stree

      , r = e.stat_desc.elems;

    let s, o, h, d = -1;

    for (t.heap_len = 0,

    t.heap_max = 573,

    s = 0; s > 1; s >= 1; s--)

        O(t, a, s);

    h = r;

    do {

        s = t.heap[1],

        t.heap[1] = t.heap[t.heap_len--],

        O(t, a, 1),

        o = t.heap[1],

        t.heap[--t.heap_max] = s,

        t.heap[--t.heap_max] = o,

        a[2 * h] = a[2 * s] + a[2 * o],

        t.depth[h] = (t.depth[s] >= t.depth[o] ? t.depth[s] : t.depth[o]) + 1,

        a[2 * s + 1] = a[2 * o + 1] = h,

        t.heap[1] = h++,

        O(t, a, 1)

    } while (t.heap_len >= 2);

    t.heap[--t.heap_max] = t.heap[1],

    ((t,e)=>{

        const a = e.dyn_tree

          , i = e.max_code

          , n = e.stat_desc.static_tree

          , r = e.stat_desc.has_stree

          , s = e.stat_desc.extra_bits

          , o = e.stat_desc.extra_base

          , h = e.stat_desc.max_length;

        let d, _, f, c, u, w, b = 0;

        for (c = 0; c  h && (c = h,

            b++),

            a[2 * _ + 1] = c,

            _ > i || (t.bl_count[c]++,

            u = 0,

            _ >= o && (u = s[_ - o]),

            w = a[2 * _],

            t.opt_len += w * (c + u),

            r && (t.static_len += w * (n[2 * _ + 1] + u)));

        if (0 !== b) {

            do {

                for (c = h - 1; 0 === t.bl_count[c]; )

                    c--;

                t.bl_count[c]--,

                t.bl_count[c + 1] += 2,

                t.bl_count[h]--,

                b -= 2

            } while (b > 0);

            for (c = h; 0 !== c; c--)

                for (_ = t.bl_count[c]; 0 !== _; )

                    f = t.heap[--d],

                    f > i || (a[2 * f + 1] !== c && (t.opt_len += (c - a[2 * f + 1]) * a[2 * f],

                    a[2 * f + 1] = c),

                    _--)

        }

    }

    )(t, e),

    Z(a, d, t.bl_count)

}

  , N = (t,e,a)=>{

    let i, n, r = -1, s = e[1], l = 0, o = 7, h = 4;

    for (0 === s && (o = 138,

    h = 3),

    e[2 * (a + 1) + 1] = 65535,

    i = 0; i {

    let i, n, r = -1, s = e[1], l = 0, o = 7, h = 4;

    for (0 === s && (o = 138,

    h = 3),

    i = 0; i {

    A(t, 0 + (i ? 1 : 0), 3),

    ((t,e,a,i)=>{

        S(t),

        i && (z(t, a),

        z(t, ~a)),

        t.pending_buf.set(t.window.subarray(e, e + a), t.pending),

        t.pending += a

    }

    )(t, e, a, !0)

}

;

var C = (t,e,a,i)=>{

    let r, s, l = 0;

    t.level > 0 ? (2 === t.strm.data_type && (t.strm.data_type = (t=>{

        let e, a = 4093624447;

        for (e = 0; e >>= 1)

            if (1 & a && 0 !== t.dyn_ltree[2 * e])

                return 0;

        if (0 !== t.dyn_ltree[18] || 0 !== t.dyn_ltree[20] || 0 !== t.dyn_ltree[26])

            return 1;

        for (e = 32; e {

        let e;

        for (N(t, t.dyn_ltree, t.l_desc.max_code),

        N(t, t.dyn_dtree, t.d_desc.max_code),

        L(t, t.bl_desc),

        e = 18; e >= 3 && 0 === t.bl_tree[2 * _[e] + 1]; e--)

            ;

        return t.opt_len += 3 * (e + 1) + 5 + 5 + 4,

        e

    }

    )(t),

    r = t.opt_len + 3 + 7 >>> 3,

    s = t.static_len + 3 + 7 >>> 3,

    s {

        let n;

        for (A(t, e - 257, 5),

        A(t, a - 1, 5),

        A(t, i - 4, 4),

        n = 0; n {

        B || ((()=>{

            let t, e, a, i, n;

            const _ = new Array(16);

            for (a = 0,

            i = 0; i >= 7; i (t.pending_buf[t.d_buf + 2 * t.last_lit] = e >>> 8 & 255,

    t.pending_buf[t.d_buf + 2 * t.last_lit + 1] = 255 & e,

    t.pending_buf[t.l_buf + t.last_lit] = 255 & a,

    t.last_lit++,

    0 === e ? t.dyn_ltree[2 * a]++ : (t.matches++,

    e--,

    t.dyn_ltree[2 * (w[a] + n + 1)]++,

    t.dyn_dtree[2 * x(e)]++),

    t.last_lit === t.lit_bufsize - 1),

    _tr_align: t=>{

        A(t, 2, 3),

        E(t, 256, f),

        (t=>{

            16 === t.bi_valid ? (z(t, t.bi_buf),

            t.bi_buf = 0,
```
