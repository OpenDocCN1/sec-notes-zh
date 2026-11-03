
```javascript
var _0x5dd0c4 = _0x1b0f61[_0x3e3f('0x11f', 'yfbZ') + 'b'](_0x1b0f61[_0x3e3f('0x5b', 'Z74&') + 'b'](_0x1b0f61[_0x3e3f('0xce', 'yo%1') + 'b'](_0x1b0f61[_0x3e3f('0xfd', '[OoV') + 'b'](_0x4321b4['tn'], '='), _0x3f0c4b[0x0]) + _0x1b0f61[_0x3e3f('0x40', 'yVRw') + 'l'], _0x4321b4['vt']), _0x3e3f('0x9b', 't[Yh') + _0x3e3f('0xbb', 'CN]8') + '\x20/');

_0x5dd0c4 = _0x5dd0c4 + _0x1b0f61[_0x3e3f('0x93', 'ftCA') + 'G'];

jsl = _0x5dd0c4;

console.log(jsl);   // 成功出值
```

注意到`go`函数传入的对象是随机的。

```javascript
{
    "bts": ["1760151418.93|0|yoME", "vzEV2XwBkaFP%2BxJzN0yvU%3D"],
    "chars": "tIRMbkDXMigbVIAizFVRzN",
    "ct": "98640d7578769f51e99a2fba2739d6ece6200fd38fe010843b0c16dbe5ff8211",
    "ha": "sha256",
    "is": true,
    "tn": "__jsl_clearance_s",
    "vt": "3600",
    "wt": "1500"
};
```

`ha`字段标记了哈希算法种类，目前发现有`md5`、`sha1`和`sha256`三种哈希算法，我们多刷新请求几次，采用笨办法，不同的哈希算法，扣下对应的 3 份代码。

![](https://attach.52pojie.cn/forum/202510/11/213424b56n91zdhoh1tt6h.png)

`jsl3.png` (82.78 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNzc4MHwzMzkwN2FiN3wxNzkzNTcwNTA2fDE2ODA4NnwyMDY1Mjg3&nothumb=yes)

2025-10-11 21:34 上传

后续使用正则拿到这个对象，判断`ha`字段，调用相应的逻辑即可。

## 测试

关闭油猴脚本，清空 cookie，刷新

![](https://attach.52pojie.cn/forum/202510/11/213525d22rsl66j76r6sms.png)

`jsl4.png` (35.49 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNzc4MXw0OTZkNGE5OHwxNzkzNTcwNTA2fDE2ODA4NnwyMDY1Mjg3&nothumb=yes)

2025-10-11 21:35 上传

此时接口返回的动态对象为

![](https://attach.52pojie.cn/forum/202510/11/213548ft3qjp33zqrmj37j.png)

`jsl5.png` (46.87 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNzc4MnxmNmVlZjQ5ZnwxNzkzNTcwNTA2fDE2ODA4NnwyMDY1Mjg3&nothumb=yes)

2025-10-11 21:35 上传

本地执行

![](https://attach.52pojie.cn/forum/202510/11/213612fzjywj02hrvpu0tv.png)

`jsl6.png` (56.81 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNzc4M3wyNGRkNDNjOHwxNzkzNTcwNTA2fDE2ODA4NnwyMDY1Mjg3&nothumb=yes)

2025-10-11 21:36 上传

与上篇联动测试

![](https://attach.52pojie.cn/forum/202510/11/213642suwvu1bud639dzzt.png)

`jsl7.png` (77.46 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNzc4NHxlNTNmMzgyN3wxNzkzNTcwNTA2fDE2ODA4NnwyMDY1Mjg3&nothumb=yes)

2025-10-11 21:36 上传

完美

## 总结

- 加速乐 cookie 防护，关键字段为`__jsl_clearance_s`
- 生成 cookie 的 js，会通过两个 521 状态码的请求获得
- 哈希算法随机，可以分别扣下各哈希算法的逻辑

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNzc4NXxiYzdkMTNkOXwxNzkzNTcwNTA2fDE2ODA4NnwyMDY1Mjg3)

(27.11 KB, 下载次数: 17)

2025-10-11 21:37 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB
