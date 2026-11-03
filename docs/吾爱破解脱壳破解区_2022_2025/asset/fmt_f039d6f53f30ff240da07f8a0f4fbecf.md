
```
const CryptoJS = require('crypto-js')
function decrypt(e, t){
    var n = CryptoJS.enc.Hex.parse(e)
      , a = CryptoJS.enc.Base64.stringify(n);
    var aeskey = CryptoJS.enc.Utf8.parse('之前的key');
    var aesiv = CryptoJS.enc.Utf8.parse("16-Bytes--String")
    var r = CryptoJS.AES.decrypt(a, aeskey, {
        iv: aesiv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    });
    return r.toString(CryptoJS.enc.Utf8).toString()
}
```

前文一些操作的谜底也终于揭开了，`o.Z.commit("setEncrypt", {enckey: ee(t),aeskey: n,aesiv: a})`其实就是将`key`和`iv`保存起来，然后解密的时候也用同样的`key`和`iv`。

好了，加密和解密都解决了，直接模拟请求。

![](https://attach.52pojie.cn/forum/202412/14/172207fx28nnza2png8dxn.png)

`16.png` (72.81 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MzcxNXxmMzQ4ZWEzNHwxNzkzNTcxNDc5fDE2ODA4NnwxOTkxMjY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzcxNXxmMzQ4ZWEzNHwxNzkzNTcxNDc5fDE2ODA4NnwxOTkxMjY4&nothumb=yes)

2024-12-14 17:22 上传

成功！！！

网站多请求几次会触发某象的验证码，以后有机会再分析怎么解决。
