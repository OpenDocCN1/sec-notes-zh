
# [CTF] 2024国城杯Reverse方向wp

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1989657
- 抓取时间: 2026-11-01T22:18:22

## 前言：

一共4个题

XXTEA（Crush's_secret）

魔方套迷宫（FunMz）

驱动题（easy_key）

JavaBase64+爆破(round)

有两个做出来了，剩下两个是赛后复现，呜呜呜，其实驱动感觉也挺简单的没想出来，赛后才恍然大悟......

魔方+迷宫就算了。似乎有佬做出来了且是唯一解，还是太强了！！

## Reverse

### Crush's_secret

`IDA`加载进去，`Shift+F12`找到疑似flag的字符串，双击进去

![](https://attach.52pojie.cn/forum/202412/09/150104bm5kcjzzmg8mkp1j.png)

image.png (30.93 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc0MjQ2OHw2MjU5YmVkMXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ2OHw2MjU5YmVkMXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:01 上传

查找交叉引用（`X`），来到主函数入口

![](https://attach.52pojie.cn/forum/202412/09/150144iypu6l2t0l2tux21.png)

image.png (54.21 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=Mjc0MjQ2OXw4OGZjYjg1MXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ2OXw4OGZjYjg1MXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:01 上传

发现`v5`数组就四个值，十六进制后

![](https://attach.52pojie.cn/forum/202412/09/150217kn9i11ie1gn4g9nn.png)

image.png (3.25 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3MHxmOTJlZDdkN3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3MHxmOTJlZDdkN3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:02 上传

这下闭着眼睛都能知道是`TEA`系列了，然后注意到`sub_411122`函数双击进去

![](https://attach.52pojie.cn/forum/202412/09/150237g3jr04paxpje31ir.png)

image.png (65.22 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3MXxiNmY1N2I3M3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3MXxiNmY1N2I3M3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:02 上传

这里的`v11`数组就是密文了，具体加密方法在`sub_41110E`处，进入函数查看

![](https://attach.52pojie.cn/forum/202412/09/150258ottw11trthctzkwr.png)

image.png (14.71 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3MnxlZjY1MTZlNnwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3MnxlZjY1MTZlNnwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:02 上传

我超，红了

![](https://attach.52pojie.cn/forum/202412/09/150327mw3mdlbo0zyovnqv.png)

image.png (113.23 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3M3wzOWExZDQ0M3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3M3wzOWExZDQ0M3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:03 上传

一看是`SMC`，直接动态调试

![](https://attach.52pojie.cn/forum/202412/09/150349qirpylwwfgyhzghr.png)

image.png (13.28 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3NHxiMDNlMjQwYnwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3NHxiMDNlMjQwYnwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:03 上传

在这个地方下断点，`F9`运行程序

![](https://attach.52pojie.cn/forum/202412/09/150408y7hff7rlrrsslqxj.png)

image.png (15.96 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3NXxiZjQzZDc4NXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3NXxiZjQzZDc4NXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:04 上传

`F7`进入函数

![](https://attach.52pojie.cn/forum/202412/09/150421k80gaaam082mm98u.png)

image.png (206.15 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3Nnw1OGU3NzU0MnwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3Nnw1OGU3NzU0MnwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:04 上传

运行到标记地点，会出现弹窗点Yes，然后在`00418000`处按`P`，函数就出来了

![](https://attach.52pojie.cn/forum/202412/09/150434s3oy7cykkqci5jy3.png)

image.png (51.09 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjQ3N3wxMDE3ZjM2OXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ3N3wxMDE3ZjM2OXwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:04 上传

进去之后，一眼`XXTEA`，找个脚本直接写
