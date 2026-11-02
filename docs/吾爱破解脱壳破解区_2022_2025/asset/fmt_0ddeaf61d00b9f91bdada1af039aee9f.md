
# 代码片段

```javascript
                if (a.extra) {

                    for (R = a.extra; d >>= a.extra,

                    d -= a.extra,

                    a.back += a.extra

                }

                a.was = a.length,

                a.mode = 23;

            case 23:

                for (; z = a.distcode[h & (1 >> 24,

                g = z >>> 16 & 255,

                m = 65535 & z,

                !(b > p)],

                    b = z >>> 24,

                    g = z >>> 16 & 255,

                    m = 65535 & z,

                    !(p + b >>= p,

                    d -= p,

                    a.back += p

                }

                if (h >>>= b,

                d -= b,

                a.back += b,

                64 & g) {

                    t.msg = "invalid distance code",

                    a.mode = Re;

                    break

                }

                a.offset = m,

                a.extra = 15 & g,

                a.mode = 24;

            case 24:

                if (a.extra) {

                    for (R = a.extra; d >>= a.extra,

                    d -= a.extra,

                    a.back += a.extra

                }

                if (a.offset > a.dmax) {

                    t.msg = "invalid distance too far back",

                    a.mode = Re;

                    break

                }

                a.mode = 25;

            case 25:

                if (0 === o)

                    break t;

                if (c = f - o,

                a.offset > c) {

                    if (c = a.offset - c,

                    c > a.whave && a.sane) {

                        t.msg = "invalid distance too far back",

                        a.mode = Re;

                        break

                    }

                    c > a.wnext ? (c -= a.wnext,

                    u = a.wsize - c) : u = a.wnext - c,

                    c > a.length && (c = a.length),

                    w = a.window

                } else

                    w = n,

                    u = s - a.offset,

                    c = a.length;

                c > o && (c = o),

                o -= c,

                a.length -= c;

                do {

                    n[s++] = w[u++]

                } while (--c);

                0 === a.length && (a.mode = 21);

                break;

            case 26:

                if (0 === o)

                    break t;

                n[s++] = a.length,

                o--,

                a.mode = 21;

                break;

            case 27:

                if (a.wrap) {

                    for (; d {

        if (!t || !t.state)

            return ve;

        let e = t.state;

        return e.window && (e.window = null),

        t.state = null,

        me

    }

    ,

    inflateGetHeader: (t,e)=>{

        if (!t || !t.state)

            return ve;

        const a = t.state;

        return 0 == (2 & a.wrap) ? ve : (a.head = e,

        e.done = !1,

        me)

    }

    ,

    inflateSetDictionary: (t,e)=>{

        const a = e.length;

        let i, n, r;

        return t && t.state ? (i = t.state,

        0 !== i.wrap && 11 !== i.mode ? ve : 11 === i.mode && (n = 1,

        n = M(n, e, a, 0),

        n !== i.check) ? ye : (r = Ie(t, e, a, a),

        r ? (i.mode = 31,

        xe) : (i.havedict = 1,

        me))) : ve

    }

    ,

    inflateInfo: "pako inflate (from Nodeca project)"

};

var He = function() {

    this.text = 0,

    this.time = 0,

    this.xflags = 0,

    this.os = 0,

    this.extra = null,

    this.extra_len = 0,

    this.name = "",

    this.comment = "",

    this.hcrc = 0,

    this.done = !1

};

const Me = Object.prototype.toString

  , {Z_NO_FLUSH: Ke, Z_FINISH: je, Z_OK: Pe, Z_STREAM_END: Ye, Z_NEED_DICT: Ge, Z_STREAM_ERROR: Xe, Z_DATA_ERROR: We, Z_MEM_ERROR: qe} = Y;

function Je(t) {

    this.options = Mt({

        chunkSize: 65536,

        windowBits: 15,

        to: ""

    }, t || {});

    const e = this.options;

    e.raw && e.windowBits >= 0 && e.windowBits = 0 && e.windowBits  15 && e.windowBits  0 && r === Ye && a.state.wrap > 0 && 0 !== t[a.next_in]; )

            Ce.inflateReset(a),

            r = Ce.inflate(a, s);

        switch (r) {

        case Xe:

        case We:

        case Ge:

        case qe:

            return this.onEnd(r),

            this.ended = !0,

            !1

        }

        if (l = a.avail_out,

        a.next_out && (0 === a.avail_out || r === Ye))

            if ("string" === this.options.to) {

                let t = Xt(a.output, a.next_out)

                  , e = a.next_out - t

                  , n = Gt(a.output, t);

                a.next_out = e,

                a.avail_out = i - e,

                e && a.output.set(a.output.subarray(t, t + e), 0),

                this.onData(n)

            } else

                this.onData(a.output.length === a.next_out ? a.output : a.output.subarray(0, a.next_out));

        if (r !== Pe || 0 !== l) {

            if (r === Ye)

                return r = Ce.inflateEnd(this.strm),

                this.onEnd(r),

                this.ended = !0,

                !0;

            if (0 === a.avail_in)

                break

        }

    }

    return !0

}

,

Je.prototype.onData = function(t) {

    this.chunks.push(t)

}

,

Je.prototype.onEnd = function(t) {

    t === Pe && ("string" === this.options.to ? this.result = this.chunks.join("") : this.result = Kt(this.chunks)),

    this.chunks = [],

    this.err = t,

    this.msg = this.strm.msg

}

;

var Ve = {

    Inflate: Je,

    inflate: Qe,

    inflateRaw: function(t, e) {

        return (e = e || {}).raw = !0,

        Qe(t, e)

    },

    ungzip: Qe,

    constants: Y

};
```

里面写了几种算法，扣下来可以学习，备着以后用。看大站，可以学习很多。

# 数据处理

因为使用 `python`，我这里做了点处理，先把 `byte` 转 `base64` 然后再处理

![](https://s2.loli.net/2023/05/18/WuBb5fDMmAxUY24.png#id=xaJnA&originHeight=774&originWidth=1711&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

所以 `js` 这样处理

![](https://s2.loli.net/2023/05/18/ezXnAtIHjYLFa9p.png#id=W7ePM&originHeight=256&originWidth=686&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

`python`：

![](https://s2.loli.net/2023/05/18/8siUArKvwfOq7xo.png#id=AfvJG&originHeight=728&originWidth=1806&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

现在数据都处理好了，就剩 `r.Response` 这个 `proto` 文件了，就能解析了。

![](https://s2.loli.net/2023/05/18/oYPxilv1fJ69DyM.png#id=SHefD&originHeight=553&originWidth=901&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

跟进去，发现是一个地方。我们刚刚不是处理过吗，找一下文件里面的，拿出来：

```proto
message webcast_im_Response{
repeated webcast_im_Message messages=1;
 string cursor=2;
 int64 fetchinterval=3;
 int64 now=4;
 string internalext=5;
 int32 fetchtype=6;
 int64 heartbeatduration=8;
 bool needack=9;
 string pushserver=10;
 string livecursor=11;
 bool historynomore=12;
}
message webcast_im_Message{
 string method=1;
 bytes payload=2;
 int64 msgid=3;
 int32 msgtype=4;
 int64 offset=5;
 bool needwrdsstore=6;
 int64 wrdsversion=7;
 string wrdssubkey=8;
}
```

编译一下，看效果：

网站上：

![](https://s2.loli.net/2023/05/18/7cUuRtz4NdBaoQJ.png#id=WkhWb&originHeight=718&originWidth=1077&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

`python`：

![](https://s2.loli.net/2023/05/18/ySr4mq7MGQox3zC.png#id=aVfNf&originHeight=743&originWidth=1794&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

一致，本来第一项列表是有数据，可以看到里面消息类型的，刚刚不小心跳过断点了，然后直播结束了，没消息了....尴尬。但是暂时不影响分析，我们把 `ack` 也完成就换个直播间。

# ack 处理

因为这里 `needack` 是 `true`，所以需要发一个包：

![](https://s2.loli.net/2023/05/18/8iexz5jtBT3R9Sv.png#id=uXQ5t&originHeight=760&originWidth=1631&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

可以看到，我们构造没问题。

代码：

```python
#ack处理

internalext=webcast_im_Response.internalext
```
