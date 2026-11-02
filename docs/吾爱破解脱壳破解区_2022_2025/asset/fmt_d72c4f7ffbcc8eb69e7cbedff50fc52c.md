
# [CTF] 【2022春节】解题领红包windows（二、三）和web解题过程

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1588476
- 抓取时间: 2026-11-01T23:36:22

## 解题领红包之二 {Windows 初级题}

查壳，无壳

![](https://attach.52pojie.cn/forum/202202/16/085904fq2lo613jqjlejl2.png)

`0.PNG` (120.08 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjQ5NTA3OXwwMWM5YTRkYXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA3OXwwMWM5YTRkYXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

运行程序看一下

![](https://attach.52pojie.cn/forum/202202/16/085907frrf33qo3qhxhfrc.png)

`1.png` (8.52 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA4MHw3YzZiZWJlNHwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4MHw3YzZiZWJlNHwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

拖进od，搜字符串，找到please input password，以及附近的tip: flag长度为23

![](https://attach.52pojie.cn/forum/202202/16/085909j3wgf0m2mrvzifrx.png)

`2.png` (17.74 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjQ5NTA4MXxjYWE5NGNiZnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4MXxjYWE5NGNiZnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

双击please input password，在该处硬件断点，运行

![](https://attach.52pojie.cn/forum/202202/16/085911s4555qlm5kmk5ykg.png)

`3.PNG` (11.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjQ5NTA4MnxlMTYzMzUwM3wxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4MnxlMTYzMzUwM3wxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

堆栈窗口看到字符串，长度也为23，可能就是flag

![](https://attach.52pojie.cn/forum/202202/16/085914pic4tkc63t26t84k.png)

`4.PNG` (8.86 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjQ5NTA4M3wwMzI4YTBmNnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4M3wwMzI4YTBmNnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

试输入，正确，得到flag为2022HappyNewYear52PoJie

![](https://attach.52pojie.cn/forum/202202/16/085916zijezjkcoqfktpjg.png)

`5.PNG` (4.92 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA4NHwxZjRjYTAxNXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4NHwxZjRjYTAxNXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

## 解题领红包之三 {Windows 中级题}

查壳，可能是有修改的upx，要先[//www.52pojie.cn/forum-5-1.html" target="_blank" class="relatedlink"](https://www.52pojie.cn/forum-5-1.html:href="https)

![](https://attach.52pojie.cn/forum/202202/16/085918okkykv4ywkferusv.png)

`6.PNG` (117.23 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA4NXwwMjAzZTQzYnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4NXwwMjAzZTQzYnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

先用52[//www.52pojie.cn/thread-661779-1-1.html" target="_blank" class="relatedlink"](https://www.52pojie.cn/thread-661779-1-1.html:href="https)脱壳，拖进od，走一步，ESP定律，在ESP寄存器上右键，数据窗口中跟随

![](https://attach.52pojie.cn/forum/202202/16/085921jq81qxm2ljl9221p.png)

`7.PNG` (33.73 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjQ5NTA4NnxiYTc0MGUzZHwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4NnxiYTc0MGUzZHwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

然后右键，断点-硬件访问-byte

![](https://attach.52pojie.cn/forum/202202/16/085923yf6t7lnkn3t84frl.png)

`8.PNG` (20.56 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA4N3w0ZTZmYzA1M3wxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4N3w0ZTZmYzA1M3wxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

然后运行程序，停在4240C6，上面是popad，下面不远处一个大jmp，推测接近OEP

![](https://attach.52pojie.cn/forum/202202/16/085925urps2sykudxpye76.png)

`9.PNG` (11.91 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjQ5NTA4OHxiNTZmNDI3ZXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4OHxiNTZmNDI3ZXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 08:59 上传

运行到4240D3该jmp处，然后单步一次，到40B37B的push处，应为OEP

![](https://attach.52pojie.cn/forum/202202/16/090015pz0p5a38ldd9sm3k.png)

`10.PNG` (14.86 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjQ5NTA4OXxiZTk4MmYxNXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA4OXxiZTk4MmYxNXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:00 上传

Ollydump脱壳

![](https://attach.52pojie.cn/forum/202202/16/090017lcwjpwqpj69qquys.png)

`11.PNG` (21.29 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjQ5NTA5MHxkNGY2NTFhYnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5MHxkNGY2NTFhYnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:00 上传

脱壳结果ExeinfoPE看不出，用PEID看应该是脱了

![](https://attach.52pojie.cn/forum/202202/16/090019r8jijoy00loljuiz.png)

`12.PNG` (127.27 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA5MXw4ZmUzZmUwZHwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5MXw4ZmUzZmUwZHwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:00 上传

运行程序看一下

![](https://attach.52pojie.cn/forum/202202/16/090021zn6fvvfs640ljsoa.png)

`13.PNG` (11.23 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjQ5NTA5MnxhMmYyZDBiN3wxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5MnxhMmYyZDBiN3wxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:00 上传

拖回实机上的IDA，看流程，首先是Input your UID:，然后到Input your Key:，最后根据某种判断条件跳到Success或Error。

![](https://attach.52pojie.cn/forum/202202/16/090023g0o9k3x4p3oy6kxl.png)

`14.PNG` (41.01 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjQ5NTA5M3xiMDI1OGQyNXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5M3xiMDI1OGQyNXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:00 上传

![](https://attach.52pojie.cn/forum/202202/16/090130e0a03311axatmizp.png)

`15.PNG` (46.59 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA5NHw5ODg5OTE5NnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5NHw5ODg5OTE5NnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:01 上传

在Success处F5得到伪代码，显然地，`sub_401520`内有判定flag是否正确的条件

![](https://attach.52pojie.cn/forum/202202/16/090240pc57wyxzfr6sw97o.png)

`16.PNG` (14.9 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjQ5NTA5NnwzNmY4NmY1OXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5NnwzNmY4NmY1OXwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:02 上传

双击点进`sub_401520`，它的返回值以`v8`变量为判断条件决定返回值为0或1

![](https://attach.52pojie.cn/forum/202202/16/090350hppu5857ejr8bmcv.png)

`17.PNG` (13.03 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjQ5NTA5N3w2MmQxN2YxMnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5N3w2MmQxN2YxMnwxNzkzNTc2MTgwfDE2ODA4NnwxNTg4NDc2&nothumb=yes)

2022-2-16 09:03 上传

[C++] 纯文本查看 复制代码
