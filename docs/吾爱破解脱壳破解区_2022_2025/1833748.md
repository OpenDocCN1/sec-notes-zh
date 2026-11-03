
# [Web逆向] 某e通的m3u8文件解密

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1833748
- 抓取时间: 2026-11-01T22:45:22

菜鸡一个，研究某利威失败，就找个网站研究一下重拾信心，然后看到有人说某e通的m3u8加密了，然后就研究了一下

感觉没有任何难度

直接搜索 `[xiaoe]` 就可以找到解密的位置，我就不贴图了

跟进去之后有个`L`函数，调试了一下，32位，应该是`md5`，那就随便找个`md5`调戏一下，结果一致，

那就复制粘贴，本地调戏，把`L`替换成`md5`，完事，然后发现最后面的`k.decode`要解码`u`，感觉`u`像`base64`

然后本地调戏和他`k.decode`对比下，一样，那就替换成`atob`搞定

懒汉一个，就不贴图了

`appid`是网址最前面那个`app`开头的字符串

注意，仅做学习使用，不要乱搞，乱搞我不负责

```
function r(r, n) {
    var t = (65535 & r) + (65535 & n);
    return (r >> 16) + (n >> 16) + (t >> 16) >> 32 - a, e);
    var f, a;
}

function t(r, t, e, u, o, c, f) {
    return n(t & e | ~t & u, r, t, o, c, f);
}

function e(r, t, e, u, o, c, f) {
    return n(t & u | e & ~u, r, t, o, c, f);
}

function u(r, t, e, u, o, c, f) {
    return n(t ^ e ^ u, r, t, o, c, f);
}

function o(r, t, e, u, o, c, f) {
    return n(e ^ (t | ~u), r, t, o, c, f);
}

function c(n) {
    for (var c = 1732584193, f = -271733879, a = -1732584194, i = 271733878, h = 0; h > 2] >> t % 4 * 8 + 4 & 15) + "0123456789abcdef".charAt(r[t >> 2] >> t % 4 * 8 & 15);
    return n;
}

function a(r) {
    for (var n = 1 + (r.length + 8 >> 6), t = new Array(16 * n), e = 0; e > 2] |= (255 & r.charCodeAt(e)) > 2] |= 128  1 && void 0 !== arguments[1] ? arguments[1] : "appbgzjnopv1917";
    e = hexMD5(e);
    var r = 3
        , n = +(t = decodeURIComponent(t)).substr(-r)[1]
        , i = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/="
        , a = i.split("").reduce((function(t, e, r) {
            return t[e] = r,
                t
        }
    ), {})
        , o = t[0]
        , s = i.indexOf(o)
        , c = hexMD5(e + o).substr(s % 8, s % 8 + 7).split("").map((function(t) {
            return t.charCodeAt()
        }
    ));
    t = t.substr(1, t.length - r - 1);
    for (var u = "", l = 0, f = 0, h = 0; h 

```
