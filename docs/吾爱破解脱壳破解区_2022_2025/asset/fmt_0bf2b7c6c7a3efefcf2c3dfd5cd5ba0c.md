
```
)), new Tt(4,4,8,4,Dt), new Tt(4,5,16,8,Dt), new Tt(4,6,32,32,Dt), new Tt(4,4,16,16,Ot), new Tt(8,16,32,32,Ot), new Tt(8,16,128,128,Ot), new Tt(8,32,128,256,Ot), new Tt(32,128,258,1024,Ot), new Tt(32,258,258,4096,Ot)];
function Nt() {
    this.strm = null,
    this.status = 0,
    this.pending_buf = null,
    this.pending_buf_size = 0,
    this.pending_out = 0,
    this.pending = 0,
    this.wrap = 0,
    this.gzhead = null,
    this.gzindex = 0,
    this.method = ut,
    this.last_flush = -1,
    this.w_size = 0,
    this.w_bits = 0,
    this.w_mask = 0,
    this.window = null,
    this.window_size = 0,
    this.prev = null,
    this.head = null,
    this.ins_h = 0,
    this.hash_size = 0,
    this.hash_bits = 0,
    this.hash_mask = 0,
    this.hash_shift = 0,
    this.block_start = 0,
    this.match_length = 0,
    this.prev_match = 0,
    this.match_available = 0,
    this.strstart = 0,
    this.match_start = 0,
    this.lookahead = 0,
    this.prev_length = 0,
    this.max_chain_length = 0,
    this.max_lazy_match = 0,
    this.level = 0,
    this.strategy = 0,
    this.good_match = 0,
    this.nice_match = 0,
    this.dyn_ltree = new Uint16Array(1146),
    this.dyn_dtree = new Uint16Array(122),
    this.bl_tree = new Uint16Array(78),
    yt(this.dyn_ltree),
    yt(this.dyn_dtree),
    yt(this.bl_tree),
    this.l_desc = null,
    this.d_desc = null,
    this.bl_desc = null,
    this.bl_count = new Uint16Array(16),
    this.heap = new Uint16Array(573),
    yt(this.heap),
    this.heap_len = 0,
    this.heap_max = 0,
    this.depth = new Uint16Array(573),
    yt(this.depth),
    this.l_buf = 0,
    this.lit_bufsize = 0,
    this.last_lit = 0,
    this.d_buf = 0,
    this.opt_len = 0,
    this.static_len = 0,
    this.matches = 0,
    this.insert = 0,
    this.bi_buf = 0,
    this.bi_valid = 0
}
const Ft = t=>{
    if (!t || !t.state)
        return kt(t, nt);
    t.total_in = t.total_out = 0,
    t.data_type = ct;
    const e = t.state;
    return e.pending = 0,
    e.pending_out = 0,
    e.wrap {
    const e = Ft(t);
    var a;
    return e === at && ((a = t.state).window_size = 2 * a.w_size,
    yt(a.head),
    a.max_lazy_match = Lt[a.level].max_lazy,
    a.good_match = Lt[a.level].good_length,
    a.nice_match = Lt[a.level].nice_length,
    a.max_chain_length = Lt[a.level].max_chain,
    a.strstart = 0,
    a.block_start = 0,
    a.lookahead = 0,
    a.insert = 0,
    a.match_length = a.prev_length = 2,
    a.match_available = 0,
    a.ins_h = 0),
    e
}
  , It = (t,e,a,i,n,r)=>{
    if (!t)
        return nt;
    let s = 1;
    if (e === lt && (e = 6),
    i  15 && (s = 2,
    i -= 16),
    n  9 || a !== ut || i  15 || e  9 || r  _t)
        return kt(t, nt);
    8 === i && (i = 9);
    const l = new Nt;
    return t.state = l,
    l.strm = t,
    l.wrap = s,
    l.gzhead = null,
    l.w_bits = i,
    l.w_size = 1 It(t, e, ut, 15, 8, ft),
    deflateInit2: It,
    deflateReset: Bt,
    deflateResetKeep: Ft,
    deflateSetHeader: (t,e)=>t && t.state ? 2 !== t.state.wrap ? nt : (t.state.gzhead = e,
    at) : nt,
    deflate: (t,e)=>{
        let a, i;
        if (!t || !t.state || e > et || e > 8 & 255),
                Et(n, n.gzhead.time >> 16 & 255),
                Et(n, n.gzhead.time >> 24 & 255),
                Et(n, 9 === n.level ? 2 : n.strategy >= ht || n.level > 8 & 255)),
                n.gzhead.hcrc && (t.adler = j(t.adler, n.pending_buf, n.pending, 0)),
                n.gzindex = 0,
                n.status = 69) : (Et(n, 0),
                Et(n, 0),
                Et(n, 0),
                Et(n, 0),
                Et(n, 0),
                Et(n, 9 === n.level ? 2 : n.strategy >= ht || n.level = ht || n.level >> 16),
                Rt(n, 65535 & t.adler)),
                t.adler = 1
            }
        if (69 === n.status)
            if (n.gzhead.extra) {
                for (a = n.pending; n.gzindex  a && (t.adler = j(t.adler, n.pending_buf, n.pending - a, a)),
                zt(t),
                a = n.pending,
                n.pending !== n.pending_buf_size)); )
                    Et(n, 255 & n.gzhead.extra[n.gzindex]),
                    n.gzindex++;
                n.gzhead.hcrc && n.pending > a && (t.adler = j(t.adler, n.pending_buf, n.pending - a, a)),
                n.gzindex === n.gzhead.extra.length && (n.gzindex = 0,
                n.status = 73)
            } else
                n.status = 73;
        if (73 === n.status)
            if (n.gzhead.name) {
                a = n.pending;
                do {
                    if (n.pending === n.pending_buf_size && (n.gzhead.hcrc && n.pending > a && (t.adler = j(t.adler, n.pending_buf, n.pending - a, a)),
                    zt(t),
                    a = n.pending,
                    n.pending === n.pending_buf_size)) {
                        i = 1;
                        break
                    }
                    i = n.gzindex  a && (t.adler = j(t.adler, n.pending_buf, n.pending - a, a)),
                0 === i && (n.gzindex = 0,
                n.status = 91)
            } else
                n.status = 91;
        if (91 === n.status)
            if (n.gzhead.comment) {
                a = n.pending;
                do {
                    if (n.pending === n.pending_buf_size && (n.gzhead.hcrc && n.pending > a && (t.adler = j(t.adler, n.pending_buf, n.pending - a, a)),
                    zt(t),
                    a = n.pending,
                    n.pending === n.pending_buf_size)) {
                        i = 1;
                        break
                    }
                    i = n.gzindex  a && (t.adler = j(t.adler, n.pending_buf, n.pending - a, a)),
                0 === i && (n.status = gt)
            } else
                n.status = gt;
        if (n.status === gt && (n.gzhead.hcrc ? (n.pending + 2 > n.pending_buf_size && zt(t),
        n.pending + 2 > 8 & 255),
        t.adler = 0,
        n.status = mt)) : n.status = mt),
        0 !== n.pending) {
            if (zt(t),
            0 === t.avail_out)
                return n.last_flush = -1,
                at
        } else if (0 === t.avail_in && vt(e) {
                let a;
                for (; ; ) {
                    if (0 === t.lookahead && (St(t),
                    0 === t.lookahead)) {
                        if (e === Q)
                            return 1;
                        break
                    }
                    if (t.match_length = 0,
                    a = q(t, 0, t.window[t.strstart]),
                    t.lookahead--,
                    t.strstart++,
                    a && (At(t, !1),
                    0 === t.strm.avail_out))
                        return 1
                }
                return t.insert = 0,
                e === tt ? (At(t, !0),
                0 === t.strm.avail_out ? 3 : 4) : t.last_lit && (At(t, !1),
                0 === t.strm.avail_out) ? 1 : 2
            }
            )(n, e) : n.strategy === dt ? ((t,e)=>{
                let a, i, n, r;
                const s = t.window;
                for (; ; ) {
                    if (t.lookahead = 3 && t.strstart > 0 && (n = t.strstart - 1,
                    i = s[n],
                    i === s[++n] && i === s[++n] && i === s[++n])) {
                        r = t.strstart + wt;
                        do {} while (i === s[++n] && i === s[++n] && i === s[++n] && i === s[++n] && i === s[++n] && i === s[++n] && i === s[++n] && i === s[++n] && n  t.lookahead && (t.match_length = t.lookahead)
                    }
                    if (t.match_length >= 3 ? (a = q(t, 1, t.match_length - 3),
                    t.lookahead -= t.match_length,
                    t.strstart += t.match_length,
```
