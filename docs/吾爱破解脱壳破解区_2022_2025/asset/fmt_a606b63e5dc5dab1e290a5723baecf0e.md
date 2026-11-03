
![](https://attach.52pojie.cn/forum/202407/05/145852qwk7bws1bqprupcr.png)

`image-20240620230549418.png` (48.92 KB, 下载次数: 6)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM1N3xlM2RjYzVlNXwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 14:58 上传

这样调试就非常舒服了

### 2.4.3 开始逆向

如果不能断住，可能是找不到 `LIuDu`，可以在函数头上添加 `bobo = LIuDu.$_AD`。

可能有几个 `break` 没删干净，手动删一下就可以。

```
var t = this;
        var n = t["$_EI_"];
        if (!n["gt"] || !n["challenge"]) return G(I("config_lack", t));
        var e = t["$_BJDj"]["$_BIBN"]();
        t["$_CCF_"] = e;
        t["$_EJK"]["cc"] = n["cc"];
        t["$_EJK"]["ww"] = n["supportWorker"];
        t["$_EJK"]["i"] = e;
        var r = t["$_CCGr"]();
        var o = $_BFc()["encrypt1"](de["stringify"](t["$_EJK"]), t["$_CCHC"]());
        var i = p["$_HET"](o);
        var s = {
          "gt": t["$_EJK"]["gt"],
          "challenge": t["$_EJK"]["challenge"],
          "lang": n["lang"],
          "pt": t["$_BJHm"],
          "client_type": t["$_BJIk"],
          "w": i + r
        };
```

我们可以很舒服的看到，`w` 是 `i` 和 `r` 的值拼接形成的。

`r` 是直接调用 `t["$_CCGr"]` 函数。

`i` 是通过执行 `p["$_HET"]` 函数，传进去参数 `o` 得到的，而参数 `o` 则是上面某个加密函数生成的。

而其它的值可以给它还原一下，比如那些 `this` 啥的不好改写。

![](https://attach.52pojie.cn/forum/202407/05/145859rdgqgzff5tjj5kdd.png)

`image-20240621112652490.png` (46.03 KB, 下载次数: 8)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM1OHw1MDllMDRhMHwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 14:58 上传

`var n = t["$EI"];` 这个是一系列大的字典，里面一大堆的值，先全部复制下来，然后看看有啥需要替换的吧。

```
var n = {
    "$_BJBl": 1718940058915, // 时间戳
    "protocol": "https://",
    "gt": "019924a82c70bb123aae90d483087f94", // 这两个是需要返回的，动态的
    "challenge": "a6eddf7c0bc30c8325c1ad94e703b736",
    "offline": false,
    "new_captcha": true,
    "product": "float",
    "width": "300px",
    "https": true,
    "api_server": "apiv6.geetest.com",
    "type": "fullpage",
    "static_servers": [
        "static.geetest.com/",
        "static.geevisit.com/"
    ],
    "beeline": "/static/js/beeline.1.0.1.js",
    "voice": "/static/js/voice.1.2.4.js",
    "click": "/static/js/click.3.1.0.js",
    "fullpage": "/static/js/fullpage.9.1.9-devcs9.js",
    "slide": "/static/js/slide.7.9.2.js",
    "geetest": "/static/js/geetest.6.0.9.js",
    "aspect_radio": {
        "slide": 103,
        "click": 128,
        "voice": 128,
        "beeline": 50
    },
    "cc": 20,
    "supportWorker": true,
    "$_FFW": {
        "pt": 0
    }
};
```

好像除了时间戳和 `gt`、`challenge` 需要变，其它的都是固定的。

![](https://attach.52pojie.cn/forum/202407/05/145926z02yqrxjqxe0br22.png)

`image-20240621112549286.png` (22.74 KB, 下载次数: 6)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM1OXxjY2MwZTU0OXwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 14:59 上传

这个 `if` 判断返回 `false`，可以直接删掉。

![](https://attach.52pojie.cn/forum/202407/05/145933w97sww8t8zrwwlrb.png)

`image-20240621113031751.png` (60.35 KB, 下载次数: 5)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2MHxlOTRjMTRkNHwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 14:59 上传

下面这个 `e` 值，似乎也是固定的，多试几次发现确实如此，那就直接写死就🆗

下面就是需要咱逆向的东西了

### 2.4.4 `r`值进行逆向

`var r = t["$_CCGr"]`

![](https://attach.52pojie.cn/forum/202407/05/150021mikubb8gmgubattz.png)

`image-20240621113236990.png` (22.63 KB, 下载次数: 7)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2MXw5YTI5NWYxY3wxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:00 上传

进入到 `t["$_CCGr"]` 函数当中，我们知道，外部调用的时候，没有带任何参数，所以 `e=undefined`，先是调用函数，得到一个 `aeskey`，然后执行 `encrypt()` 函数进行 `AES` 加密，返回得到结果。

![](https://attach.52pojie.cn/forum/202407/05/150029axzsyvrazykjtwxk.png)

`image-20240621113525476.png` (21.13 KB, 下载次数: 7)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2Mnw0NTMyMjUzNnwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:00 上传

注意：这是第一个坑，如果出现这种情况，说明此时，该网站有缓存之类的，记得清空

![](https://attach.52pojie.cn/forum/202407/05/150035dpyp1zop3ouophyf.png)

`image-20240621114622091.png` (92.56 KB, 下载次数: 7)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2M3w2YzI0MmJjZXwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:00 上传

![](https://attach.52pojie.cn/forum/202407/05/150042w6vj5jbni0nifj30.png)

`image-20240621114746653.png` (38.75 KB, 下载次数: 7)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2NHxhYzdkZWMyN3wxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:00 上传

这样才对，我们的 `aeskey` 其实是 `te()` 函数返回的，所以可以直接将 `this["$_CCHC"]` 改写为 `te()` 函数即可。

来到 `te()` 函数，发现是四个随机数拼接返回的值，直接扣下来就可以。

![](https://attach.52pojie.cn/forum/202407/05/150050l9qz0uv8b8q8d886.png)

`image-20240621125126701.png` (41.43 KB, 下载次数: 6)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2NXxmM2RmNjg1NnwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:00 上传

```
function e() {
    return (65536 * (1 + Math["random"]()) | 0)["toString"](16)["substring"](1);
}

function te() {
    return e() + e() + e() + e();
};
```

`key` 值搞定后，跟进加密的地方看看，看到熟悉的 `RSA`，可恶，它的 `aeskey` 竟然是迷惑人的，实际上是 `RSA`，差点被骗

![](https://attach.52pojie.cn/forum/202407/05/150055siahfcz6el515yy1.png)

`image-20240621191951256.png` (40.22 KB, 下载次数: 7)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2Nnw1MTczM2U1Y3wxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:00 上传

两种方式，一种是直接用 `RSA` 的加密库 `node-jsencrypt`，当然你要改一下它的源码，因为它里面的加密函数没有暴露出来

也就是说要在最后把这个 `module.exports` 改成以下格式才行

```
module.exports = {
    'JSEncrypt':JSEncryptExports.JSEncrypt,
    'RSAKey':JSEncryptExports.RSAKey
};
```

另一种的话就是硬扣 + 稍微一点的补环境 + 删除无效代码，比如那些检测 `MouseEvent` 事件的，之前没删干净的 `LIuDu.$_DV()[0]` 的东西

然后补上 `window` 和 `navigator`，最后调用内部函数

这里也有很多坑，比如说最容易弄错的地方吧

![](https://attach.52pojie.cn/forum/202407/05/150102rz2olyxd1nqka7wd.png)

`image-20240622112849808.png` (16.13 KB, 下载次数: 6)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2N3xmYjJmMmM2MHwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:01 上传

就是这里，对应原来的这个地方，可能 `AST` 还原的时候不精准，把 `v(0)` 变成了 `"stringfy"`，`v(1)` 变成了 `"prototype"`，肯定不止这一个地方，记得网站上也改一下

![](https://attach.52pojie.cn/forum/202407/05/150110kj46664r5dhj9scd.png)

`image-20240622113030029.png` (19.26 KB, 下载次数: 7)

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNjM2OHw3M2NlMmJlMXwxNzkzNTcyMDczfDE2ODA4NnwxOTQwNTY2&nothumb=yes>

2024-7-5 15:01 上传

诶，找了一上午原来是这个错误，然后差不多弄弄就应该出来了，扣了500多行

![](https://attach.52pojie.cn/forum/202407/05/150117fv7jvhvzjmqqzq5q.png)

`image-20240622113220884.png` (246.46 KB, 下载次数: 5)
