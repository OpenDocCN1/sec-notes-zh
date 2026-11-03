
# [.NET逆向] 简单脱一个DNG_CM

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1766570
- 抓取时间: 2026-11-01T22:57:43

CM原帖地址：[//www.52pojie.cn/forum.php?mod=viewthread&tid=952601" target="_blank"](https://www.52pojie.cn/forum.php?mod=viewthread&tid=952601:href="https)

## 1、前期准备

下载后通过`dng`特有的`dll`可以看到文件为`dng3.8.4`版本，据楼主所说是旗舰版只加了`hvm`，试试威力
![](https://attach.52pojie.cn/forum/202303/29/134417b8o1mormnv8ocm2v.png)
`QQ截图20230329134335.png` (18.95 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYwMjkxOHw3MTQwOWY0ZnwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkxOHw3MTQwOWY0ZnwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 13:44 上传

## 2、jit[//www.52pojie.cn/forum-5-1.html" target="_blank" class="relatedlink"](https://www.52pojie.cn/forum-5-1.html:href="https)，这个搜索一下资料，这里我就略过了

## 3、脱壳修复

![](https://attach.52pojie.cn/forum/202303/29/135902u7vleu5nzidmm55j.png)
`QQ截图20230329135854.png` (97.5 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjYwMjkxOXxkYWQ0YzcxMnwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkxOXxkYWQ0YzcxMnwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 13:59 上传

首先可以看到脱壳的效果还是不错的，方法过程都被还原出来了，但是运行时提示
![](https://attach.52pojie.cn/forum/202303/29/140034gkxjlzxf22u9u2xc.png)
`QQ截图20230329140013.png` (17.22 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYwMjkyMXw4NjBmNGFiZHwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkyMXw4NjBmNGFiZHwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:00 上传

看来还是需要修复一下，转到`.cctor`下可以看到方法虽然被还原了但依旧没有剥离`dng`
![](https://attach.52pojie.cn/forum/202303/29/140208fe4rohijdher8rox.png)
`QQ截图20230329140121.png` (79.32 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjYwMjkyMnxmOTViNWFhZHwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkyMnxmOTViNWFhZHwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:02 上传

将`cctor`下的`ZYXGuarder.Startup()`；`nop`掉保存，打开发现依然运行不了，这时开始使用`dnspy`进行调试看看问题出现在哪里
![](https://attach.52pojie.cn/forum/202303/29/140543irr7i4v9xoaa4ftg.png)
`QQ截图20230329140534.png` (17.8 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjYwMjkyNXxhMjNjYjFmOHwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkyNXxhMjNjYjFmOHwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:05 上传

转到入口点后发现这里还有一处调用，看来问题就是这里了，继续`nop`
![](https://attach.52pojie.cn/forum/202303/29/140724utcno7ohhq6oht28.png)
`QQ截图20230329140714.png` (23.16 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjYwMjkyN3wxOTEzOWUyYXwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkyN3wxOTEzOWUyYXwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:07 上传

这里看到在加载`form1`时报错，将调用全部`nop`之后保存
![](https://attach.52pojie.cn/forum/202303/29/141031xgdnj1833dy8p66n.png)
`QQ截图20230329141022.png` (18.58 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjYwMjkyOXwzMWQ4ZGM1OXwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkyOXwzMWQ4ZGM1OXwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:10 上传

![](https://attach.52pojie.cn/forum/202303/29/140853g95t9t97533l8xrm.png)
`QQ截图20230329140843.png` (69.37 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYwMjkyOHw0MTIyODliMnwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkyOHw0MTIyODliMnwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:08 上传

这次可以正常运行了，测试通过
![](https://attach.52pojie.cn/forum/202303/29/141157zidqy59yjjs65i66.png)
`QQ截图20230329141133.png` (15.09 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYwMjkzMHxlYzg3NmRiZXwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjkzMHxlYzg3NmRiZXwxNzkzNTczODYwfDE2ODA4NnwxNzY2NTcw&nothumb=yes)
2023-3-29 14:11 上传

```
ZYXDNGuarder.Startup();
ZYXDNGuarder.Star
```
