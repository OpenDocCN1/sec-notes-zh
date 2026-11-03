
[href="forum.php?mod=attachment&aid=MjY3NzIyOXwwZmY0ODNiOHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIyOXwwZmY0ODNiOHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:36 上传

![](https://attach.52pojie.cn/forum/202402/25/003641e8f8iie83ytaaca5.png)

`吾爱破解【2024年春节】解题领红包之Web题[0]000177.png` (83.21 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzIzMHxlM2Q4OTdlMnwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzMHxlM2Q4OTdlMnwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:36 上传

![](https://attach.52pojie.cn/forum/202402/25/003644s6aradrchmoqzw0r.png)

`吾爱破解【2024年春节】解题领红包之Web题[0]000201.png` (83.68 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzIzMXwwN2NkZWEwYXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzMXwwN2NkZWEwYXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:36 上传

![](https://attach.52pojie.cn/forum/202402/25/003646ibm8rrr99pr8r89z.png)

`吾爱破解【2024年春节】解题领红包之Web题[0]000241.png` (77.98 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzIzMnwzMmEwYmIyMnwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzMnwzMmEwYmIyMnwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:36 上传

可以用华清大学连夜购买的 Photoshop 拼起来：

![](https://attach.52pojie.cn/forum/202402/25/003717ungrzq60ggapoqa2.png)

`QQ图片20240217101356.png` (88.35 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzIzNHw4ZmViYzkwZnwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzNHw4ZmViYzkwZnwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:37 上传

目测宽度及阅读格式信息可以知道这个 QR 码是 Version 4 Level H Mask 0 的。H 级容错，有文字没关系，直接扫就行。

这是个混合型的 QR 码。

```
Version: 4 (33×33)
Error correction level: H
Data mask pattern: 000 (0)
(Begin data)
Byte mode: https
Alphanumeric mode: ://2024
Byte mode: challenge.52pojie.cn/
(End data)

https://2024challenge.52pojie.cn/
```

可惜一开始因为 DNS 的问题没能成功打开。

顺带复现一下这个 QR 码。

![](https://attach.52pojie.cn/forum/202402/25/003739sa6n6n6nnqqa8rcn.png)

`QQ图片20240217102031.png` (8.43 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzIzNXwxMWE5MmMzY3wxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzNXwxMWE5MmMzY3wxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:37 上传

打开时 Header 有 Flag 2：

```
HTTP/1.1 302 Found

X-Flag2: flag2{xHOpRP}
```

登录时的 Set-Cookie 藏着一点东西：

```
POST /auth/login HTTP/1.1
HTTP/1.1 302 Found
Date: Sat, 17 Feb 2024 03:57:15 GMT
Set-Cookie: flagA=jSFoeEAv9TLOh5Vu9v2rPpwibER05UMT/7sgK93Vonp3anYy9XMq/782NA==; expires=Sat, 17 Feb 2024 04:00:00 GMT; path=/; SameSite=Lax
Set-Cookie: uid=45Wru7lYfiPf6Sei5KBK231Dr6jwzZMNPUNP9CZrzT+sWhQ=; path=/; SameSite=Lax
```

但是我不会解密啊。

等等，如果 Cookie 只有加密的 UID，那，网页是怎么显示 UID 的呢……

![](https://attach.52pojie.cn/forum/202402/25/003815pj1i21y85e65a5n8.png)

`QQ图片20240217115944.png` (16.65 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY3NzIzNnxiNTdhYTQ1NHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzNnxiNTdhYTQ1NHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:38 上传

原来，把加密后的 UID 用 Cookie 传入 `https://2024challenge.52pojie.cn/auth/uid` 便能得到明文 UID……嗯？如果我狸猫换太子，把 UID 改成 Flag 呢？

Fiddler，启动！

![](https://attach.52pojie.cn/forum/202402/25/003816p7v0rao6rsicorvl.png)

`QQ图片20240217120126.png` (11.75 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY3NzIzN3w4NWQ3YzJiZXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzN3w4NWQ3YzJiZXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

![](https://attach.52pojie.cn/forum/202402/25/003818h38r83zrl8yzdr11.png)

`QQ图片20240217120147.png` (77.72 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzIzOHw4MDEyYjgzZXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzIzOHw4MDEyYjgzZXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

![](https://attach.52pojie.cn/forum/202402/25/003831vzpgp04qmoggfwlf.png)

`QQ图片20240217120234.png` (21.86 KB, 下载次数: 1)

やったぜ！

在源代码赫然发现一些东西。

```
background: url("flag4_flag10.png") white center center no-repeat;
```

![](https://attach.52pojie.cn/forum/202402/25/003845lky44tfsftksl7vv.png)

`flag4_flag10.png` (52.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY3NzI0MXw1YmMxMmU4MHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzI0MXw1YmMxMmU4MHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:38 上传

肉眼就能看到 Flag 4。当然是白字，所以白背景下看不见。

![](https://attach.52pojie.cn/forum/202402/25/003858k2z5915q8ga50apy.png)

`QQ图片20240217181517.png` (61.17 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY3NzI0Mnw5ZWNhMWExY3wxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzI0Mnw5ZWNhMWExY3wxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:38 上传

```
flag4{YvJZNS}
```

把 Alpha 通道去掉，只留下 RGB 部，就能看到 Flag 10。

![](https://attach.52pojie.cn/forum/202402/25/003918o1p3cqqps3cvi9hz.png)

`QQ图片20240217181702.png` (57.85 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzI0M3xkYzZhZDg1OHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzI0M3xkYzZhZDg1OHwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:39 上传

秒杀两题的解法是同时展示两部。

![](https://attach.52pojie.cn/forum/202402/25/003921w2j76gz2kqlsp0hr.png)

`QQ图片20240217181746.png` (72.57 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3NzI0NHxjMjk3NWNhYXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzI0NHxjMjk3NWNhYXwxNzkzNTcyNjU1fDE2ODA4NnwxODkzMDcx&nothumb=yes)

2024-2-25 00:39 上传

```
flag10{6BxMkW}
```

现在已经完成初级题了。

```
flag1{52pj2024} flag2{xHOpRP} flag3{GRsgk2} flag4{YvJZNS} flagA{daffd801}
```

三血。

Flag 5 同样是藏在源代码里面。

```
