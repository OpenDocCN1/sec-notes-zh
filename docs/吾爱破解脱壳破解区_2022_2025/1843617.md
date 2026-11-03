
# [Web逆向] 某XX自考小程序的AES加密分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1843617
- 抓取时间: 2026-11-01T22:44:42

## 前言

本人主要是报了自考在这个小程序上面做题，就研究了一下这个接口本文仅供学习交流使用，请勿随意传播。
如有侵犯你的权益及时联系我删除。

工具：主要用到这位小哥的 `devtools` 工具。

![](https://static.52pojie.cn/static/image/smiley/default/17.gif)

https://www.52pojie.cn/forum.php?mod=viewthread&tid=1842912&highlight=%D0%A1%B3%CC%D0%F2

## 一、抓包分析

打开小程序，打开`devtools`工具，这里就不啰嗦，直接上过程。

![](https://attach.52pojie.cn/forum/202310/12/160038ge77nhk46hpu003e.png)

image.png (65.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODYyNHw0Yzk0OGMxZXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODYyNHw0Yzk0OGMxZXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes)

2023-10-12 16:00 上传

![](https://attach.52pojie.cn/forum/202310/12/155821n0566ma2j4zje5m6.png)

image.png (55.17 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODYyM3w2YjYxNGNiYXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODYyM3w2YjYxNGNiYXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes)

2023-10-12 15:58 上传

点击`initiator`，

![](https://attach.52pojie.cn/forum/202310/12/160305nm8pdkdpg3rpr88n.png)

image.png (148.16 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODYyNnwyZDQ1OWVlM3wxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODYyNnwyZDQ1OWVlM3wxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes)

2023-10-12 16:03 上传

点击红框进入 。

一般的加解密的关键字都是`decrypt`，`encrypt` ，反正各种关键字使劲搜就完事了 。
我这里搜的是解密的关键词 `decrypt` 。。找到好多处。下断点测试 。

![](https://attach.52pojie.cn/forum/202310/12/160600nkw8ibjrkh21l1ir.png)

image.png (138.25 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODYyN3w1NjI4MDU4MXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODYyN3w1NjI4MDU4MXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes)

2023-10-12 16:06 上传

此处下断点 。

![](https://attach.52pojie.cn/forum/202310/12/160726uw87hr5w17uefh7t.png)

image.png (160.3 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODYyOXw3MjAxODAwOXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODYyOXw3MjAxODAwOXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes)

2023-10-12 16:07 上传

可以看到加密方式是`AES`加密的 。其实他这里用到的是 `CryptoJS` 的前端解密的 。加密方式是 `CBC-128` 。

## 二、结束

`CryptoJS` 是一个三方的`JS`库。写一段代码证明一下。

[JavaScript] 纯文本查看 复制代码

```
 function cryptoDecrypt_(e) {
var t = n.crpytoConfig.AES_IV
, o = n.crpytoConfig.AES_KEY
, c = e.replace(/\_/g, "/").replace(/\-/g, "+")
, a = CryptoJS.enc.Utf8.parse(o)
, u = CryptoJS.enc.Utf8.parse(t)
, p = CryptoJS.enc.Base64.parse(c)
, f = CryptoJS.enc.Base64.stringify(p);
return CryptoJS.AES.decrypt(f, a, {
iv: u,
mode: CryptoJS.mode.CBC,
padding: CryptoJS.pad.Pkcs7
}).toString(CryptoJS.enc.Utf8).toString()
}

console.log(cryptoDecrypt_(str));
```

得到了明文。

![](https://attach.52pojie.cn/forum/202310/12/161423sqeheiiqmzt3sqff.png)

image.png (105.46 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODYzMnxhZjFlYzAzNXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODYzMnxhZjFlYzAzNXwxNzkzNTczMDgwfDE2ODA4NnwxODQzNjE3&nothumb=yes)

2023-10-12 16:14 上传

加密的方式也是一样的。整体来说还是比较简单的。

第一次发逆向`JS`的帖子。都不知道咋编辑的好。各位看官见谅哈。

如果有违规，可以删除。
