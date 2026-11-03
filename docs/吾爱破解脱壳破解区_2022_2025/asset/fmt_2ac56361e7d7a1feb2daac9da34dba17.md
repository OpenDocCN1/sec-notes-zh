
[href="forum.php?mod=attachment&aid=MjY5NzA1NXw1MGI1MzdjN3wxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA1NXw1MGI1MzdjN3wxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传

上面那几个虽然可以跳过`debugger`，但是可能会有一些奇奇怪怪的bug【不是】（我踩过的坑，希望有人也踩一下），所以，最好改一下`_0x17a01d(0xc2)('debu' + 'gger')_0x17a01d(0x100)('action')`中的`('debu' + 'gger')`，比如去掉一个“g”

## 找到启动的js

![](https://attach.52pojie.cn/forum/202405/21/130303azojs4sl4e4gwrsk.png)

19.png (205.31 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjY5NzA1Nnw0MGFjM2Y3NHwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA1Nnw0MGFjM2Y3NHwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传

## 开始调试

![](https://attach.52pojie.cn/forum/202405/21/130306z0scooy0ra9cc3zs.png)

20.png (309.37 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjY5NzA1N3w5MDNjMDg0ZXwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA1N3w5MDNjMDg0ZXwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传

![](https://attach.52pojie.cn/forum/202405/21/130309yt4gowttrm7umc4j.png)

21.png (299.85 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5NzA1OHw4MDgzM2QwM3wxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA1OHw4MDgzM2QwM3wxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传

![](https://attach.52pojie.cn/forum/202405/21/130311mzudei4hpmzcqhwm.png)

22.png (358.09 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY5NzA1OXxjN2UyMzI2OHwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA1OXxjN2UyMzI2OHwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传

## 分析代码

[JavaScript] 纯文本查看 复制代码

```
document.addEventListener('keydown', function(event) {    if (event.ctrlKey && event.shiftKey && event.key === 'I') {        event.preventDefault()    }});document.addEventListener('keydown', function(event) {    if (event.key === 'F12') {        event.preventDefault()    }});var fl1 = "flag{";var fl2 = "pj52}";var pa1 = "collinchen1218";var pa2 = "crackme_";var nu1 = 5555 * 5555;var pa3 = nu1 + "_";var nu2 = 8888 * 8888;var pa4 = nu2 + "_";var nu3 = 5222 * 5222;var pa5 = nu3 + "_";var pa6 = "cm_";var pa7 = "horry_";var pa8 = "52pojie_";var pa9 = "magic_";var rd1 = fl1 + pa2 + pa3 + fl2;var rd2 = fl1 + pa7 + pa2 + fl2;var rd3 = fl1 + pa8 + pa6 + fl2;var rd4 = fl1 + pa9 + pa7 + fl2;var rd5 = fl1 + pa8 + pa1 + fl2;var rd6 = fl1 + pa4 + pa8 + fl2;var rd7 = fl1 + pa6 + pa5 + fl2;var rd8 = fl1 + pa8 + pa1 + fl2;var rd9 = fl1 + pa4 + pa9 + fl2;var rd10 = fl1 + pa5 + pa4 + fl2;function verifyPassword() {    var inputPassword = document.getElementById("inputPassword").value;    var correctPassword = "flag{52pojie_Ha5py_M8y_cr6ckme_qwer56uiop_ht01_N9w@2024}";    var correctPassword2 = "flag{52p0ji5_Ha58y_M8y_cr6c1me_qwer56ulkp_ht01_N9w@2024}";    if (inputPassword === "flag{asdedfgh_cm_9999}") {}    if (inputPassword === rd7 + rd3) {        document.getElementById('editable-div').innerHTML = "密码正确"    } else {        document.getElementById('editable-div').innerHTML = "密码错误"    }}
```

看到以下代码就可以知道判断逻辑了

[JavaScript] 纯文本查看 复制代码

```
if (inputPassword === rd7 + rd3) {        document.getElementById('editable-div').innerHTML = "密码正确"    } else {        document.getElementById('editable-div').innerHTML = "密码错误"    }
```

在控制台输入 `rd7 + rd3`,就可以看到flag了

![](https://attach.52pojie.cn/forum/202405/21/130314m0t99d8fkfwgd9lh.png)

23.png (330.31 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5NzA2MHwwZDVkNzYwYXwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA2MHwwZDVkNzYwYXwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传

输入`flag{cm_27269284_pj52}flag{52pojie_cm_pj52}`验证一下，密码正确

![](https://attach.52pojie.cn/forum/202405/21/130316p9z9uc9gcipc99ce.png)

24.png (57.1 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY5NzA2MXwxNThkNjU0OXwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzA2MXwxNThkNjU0OXwxNzkzNTcyMTYxfDE2ODA4NnwxOTI2NTc0&nothumb=yes)

2024-5-21 13:03 上传
