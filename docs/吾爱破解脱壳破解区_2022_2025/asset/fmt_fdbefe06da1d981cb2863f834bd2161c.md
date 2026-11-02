
# [CTF] 2022祥云杯CTF 中 babyparser 的题解

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1718676>
- 抓取时间：2026-11-01T23:05:00

这道题是比赛结束前3小时放出来的，最后是0解。我有理由怀疑这是一道防ak题。

原题附件需要`glibc2.34`，但我的kali版本不够高，所以我`patchelf`成自编译的2.34，也因此搞丢了原题附件，现在上传的是`patchelf`以后的版本，如果有需要可以自己`patch`回去。

## 比赛时的逆向

比赛时候打开一看啊，发现这个程序用了`ollvm`的`deflat`来进行混淆，而且仔细一看是个全混淆，所有能混淆的地方它都混淆了。还好之前有所准备，拿起了之前备好的[`deflat.py`](https://github.com/cq674350529/deflat/blob/master/flat_control_flow/deflat.py)准备一把梭。梭完一看发现失败了，原来是这个`ollvm`改良过，出题人修改了`deflat`的`llvm pass`，分发逻辑增加了一步。其中真实块的样子大概是这样的

![](https://attach.52pojie.cn/forum/202211/25/204916n33g36zllb3e9gcs.png)

`1669376249002.png`（191.75 KB，下载次数：263）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3MzE2OHxmMTdkYTM5NHwxNzkzNTc0Mjk3fDE2ODA4NnwxNzE4Njc2&nothumb=yes)

2022-11-25 20:49 上传

这时候我以为出题人把打乱的函数都用上图中类似的`call rax`间接调用来实现。此时我心态平和，我以为还剩三小时放出来的题必不会很难，于是想尝试修改上面那个基于`angr`的`deflat`脚本。我低估了其中的难度，对于`angr`本身也不熟悉的我根本改不来。所以最后尝试动调加手动`patch binary`。但是只是尝试了一会我就发现这个做不到，因为量太大了。因此最后决定这题先放弃，跑去做别的了。

## 赛后复盘

### 去混淆

赛后当然还是想把这道题`deflat`出来，于是在之前的基础上学习了`angr`的用法，折腾了几天，发现`angr`作为符号执行的工具对于这种去混淆的处理其实并不是很合适。因为`deflat`部分的处理本质上是在模拟执行程序，并且记录真实的控制流，而这部分并不是`angr`所擅长的。那么谁更合适呢？当然是模拟执行的神器`unicorn`！

转头去尝试用`unicorn`完成`deflat`后，这题终于也走上了正轨。找了已有的`unicorn` `deflat`脚本，并在此基础上进行更改，写出了一个`deflat`脚本：
