
# [原创] CrackMe-008

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1897449>
- 抓取时间：2026-11-01T22:36:38

## CrackMe-008

这个功能的 [www.52pojie.cn](https://www.52pojie.cn)，由于是刚学习破解，先从简单的开始练习。

先打开页面。

![img_1.png](https://attach.52pojie.cn/forum/202403/06/173115ntnftb8t69yibl85.png)

`img_1.png`（78.84 KB，下载次数：2）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2MHwzZjA0OWE0NXwxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

然后在运行下，到底是个啥玩意。

![img_2.png](https://attach.52pojie.cn/forum/202403/06/173124nuuclqcjuuahu0au.png)

`img_2.png`（8.54 KB，下载次数：2）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2MXw2NjU5YzAwNXwxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

长这个样子，挺丑的，为了学习没办法。

![img_3.png](https://attach.52pojie.cn/forum/202403/06/173133kylz5ys67ezrnops.png)

`img_3.png`（14.27 KB，下载次数：3）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2MnxmZDgyZmU3MHwxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

糟糕，输入错误了，开始搞事情。

对于我这种新手来说，肯定是要查找这个字符串的。

于是我打开。

![img_4.png](https://attach.52pojie.cn/forum/202403/06/173140nkd1ev56v5vwv55o.png)

`img_4.png`（75.87 KB，下载次数：3）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2M3wwYjY2NjIwM3wxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

点击智能搜索，然后出来了一个奇怪的东西。

![img_5.png](https://attach.52pojie.cn/forum/202403/06/173146siwjwwyajyhxhjby.png)

`img_5.png`（14.57 KB，下载次数：4）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2NHxmYzllNDcwMHwxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

没想到是明文，我们假装不知道点进去。

然后就到了这个地址的附近。

![img_6.png](https://attach.52pojie.cn/forum/202403/06/173150ekrrrww1orwcb1rk.png)

`img_6.png`（17.53 KB，下载次数：3）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2NXw1NTJmNzA5ZXwxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

往上找，发现有个可以的 `jnz` 比较跳转，是跳转到这个地址的。

上面那个可能是正确得。

那么我们修改下跳转地址，改到 `401597`。

![img_7.png](https://attach.52pojie.cn/forum/202403/06/173155kr4pwblnfri7p1wr.png)

`img_7.png`（34.14 KB，下载次数：3）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDA2NnxkODcxNzYwZXwxNzkzNTcyNTk2fDE2ODA4NnwxODk3NDQ5&nothumb=yes>

2024-3-6 17:31 上传

ok 成功，打卡 2024 年 3 月 6 日。

至于密码是什么，这个就不看了，都成功了，管那么多干啥！先开心开心。
