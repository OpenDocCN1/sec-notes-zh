
# [原创] 给Win10的资源管理器治疗啰嗦病症

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1700704
- 抓取时间: 2026-11-01T23:07:55

## 问题描述

![](https://attach.52pojie.cn/forum/202210/18/105519k8zzvytfgztawlea.png)

`image.png` (182.75 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIxNnxkODZkYmE2ZnwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIxNnxkODZkYmE2ZnwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 10:55 上传

如图所示，当你在Windows资源管理器重命名时，是不是总有个啰了叭嗦的对话框出来？

询问你是否要改名？今天，我们就是调试修改一下它。

## 调试思路

当你在Win9x系统时，使用`SoftICE`或`TRW2000`时，任意时刻你都能用`Ctrl+D`唤出控制台来调试目标程序。

这个框框出来，你再去附加我们的调试器，比如`x64dbg`就已经晚了。

所以呢，我们就得先找一个假想敌，过渡一下来间接的来调试它。

## 从 `notepad.exe` 开始

开始以为是不是`exeplor.exe`？一切从怀疑开始。

用`x32dbg`来打开吧，结果提示需要64位的来

就重换`x64dbg`来走起吧。

当你一按运行时，就发现进程已经结束了。

。。。

所以呢，几经失败，我们就换成了调试`notepad.exe`

先`Alt+E`来到模块窗口中，尝试搜索`message`，下好断点，发现并不好使啊。。。

于是想。。。是不是位于`shell32.dll`模块中？

结果发现设断也好，搜索字符串也好，并不好使啊。。。

然后一想自从win8以后，不少人都说windows是印度人搞的。。。越来越臃肿，没用的功能与来越多，蛋痛的功能也不少。

以前你一个`notepad.exe`只需要一个主程序就行了，如今你去搜索还会发现多余的语言文件。。。

所以呢？搜索中文是多么的愚蠢行为。。。

那么，我们换成英文`renname`吧。

```asm
00007FFE5F485152 | 48:8D15 F73A0500 | lea rdx,qword ptr ds:[7FFE5F4D8C50] | 00007FFE5F4D8C50:L"RenameEdit"
```

就是就发现了多个，在这上断到了。

位于`explorerframe.dll`

![](https://attach.52pojie.cn/forum/202210/18/111558ayj44tzdd6dt2jyd.png)

`image.png` (81.29 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIxN3xlNGM1MWY3MnwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIxN3xlNGM1MWY3MnwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:15 上传

## 回溯调用来源

然后呢，我们试图使用堆栈窗口，慢慢向上回溯，似乎发现这样是无功而返的，因为它出现的地方更早。

所以，我再`Alt+M`搜索下内存窗口`Ctrl+B`

![](https://attach.52pojie.cn/forum/202210/18/111944pclma98ol317e6a5.png)

`image.png` (91.56 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIyMHwwZmVkYjBhNnwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyMHwwZmVkYjBhNnwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:19 上传

就会发现2处

我们来到DUMP窗口中按下`Ctrl+Shift+1`，设置1字节硬件访问断点

断下后

![](https://attach.52pojie.cn/forum/202210/18/112121v39x00btl99lzunb.png)

`image.png` (373.22 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIyMXw0MWIzMTEwZHwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyMXw0MWIzMTEwZHwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:21 上传

就会来到这下面附近

![](https://attach.52pojie.cn/forum/202210/18/112210gsuy7s5iyhmiypyo.png)

`image.png` (354.23 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIyMnw0Zjg5YzdmY3wxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyMnw0Zjg5YzdmY3wxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:22 上传

然后不断的`Ctrl+A`向上查看段首 是哪里调用的。。。

![](https://attach.52pojie.cn/forum/202210/18/112407lteajtz84m410w70.png)

`image.png` (422.91 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIyM3w5YTg0YzkxNHwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyM3w5YTg0YzkxNHwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:24 上传

不断的重复向上查看，并配合堆栈窗口

![](https://attach.52pojie.cn/forum/202210/18/112530se8ej0iit6l658i5.png)

`image.png` (457.39 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIyNHwwOTBiYjJjYXwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyNHwwOTBiYjJjYXwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:25 上传

![](https://attach.52pojie.cn/forum/202210/18/112555oxi6q9i58qpc6d6q.png)

`image.png` (386.62 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjU2MTIyNXwwYTg3MjlmMHwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyNXwwYTg3MjlmMHwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:25 上传

![](https://attach.52pojie.cn/forum/202210/18/112825fghp7vjv1j7xd3sp.png)

`image.png` (178.89 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU2MTIyNnwzZmVhYTNlOXwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU2MTIyNnwzZmVhYTNlOXwxNzkzNTc0NDcyfDE2ODA4NnwxNzAwNzA0&nothumb=yes)

2022-10-18 11:28 上传

## 修改与替换

最终，我们就来到了上面的这个地方！！！ 照图那样修改那个烦人的框框就再也不会出来了。。。

补丁后的文件，需要到WinPE环境下去替换。。。

[Asm] 纯文本查看 复制代码
