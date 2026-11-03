
# [原创] IDA Pro 9.0临时解决Local Windows debugger结束调试时出现崩溃问题

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1956001
- 抓取时间: 2026-11-01T22:24:56

## 1、IDA Pro 9.0的Local Windows debugger调试模式在结束调试时大概率会出现崩溃
![](https://attach.52pojie.cn/forum/202408/19/144921b4je3bmkrbfjpejf.png)
剪贴板 2024-08-19 下午 2.38.59.png (43.07 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjcxODYyMnxkYmM4N2I5Y3wxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxODYyMnxkYmM4N2I5Y3wxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes)
2024-8-19 14:49 上传
经分析，问题出在`ida64.exe`的`0x14010B949`附近，有一个意义不明的条件决定是否将`r8`赋值给`rbx`，若未成功赋值则后续代码触发异常导致IDA崩溃
![](https://attach.52pojie.cn/forum/202408/19/144918n89hrp4s9s5qgw7s.png)
剪贴板 2024-08-19 下午 2.36.48.png (95.61 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjcxODYyMXw5ZGY5OTVjOHwxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxODYyMXw5ZGY5OTVjOHwxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes)
2024-8-19 14:49 上传
将该部分代码改为类似如下即可
![](https://attach.52pojie.cn/forum/202408/19/144924ss1obbb8oen1vbvj.png)
剪贴板 2024-08-19 下午 2.44.08.png (181.61 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjcxODYyM3xlOTJmNjI3Y3wxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxODYyM3xlOTJmNjI3Y3wxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes)
2024-8-19 14:49 上传

## 2、此时可以正常退出调试了，但是从第二次启动调试时有小概率弹出“IDA has encountered a problem”，但只是启动调试失败，不会导致IDA崩溃
解决方法是将`0x1400A39F4`的`jz`改为`jmp`即可
![](https://attach.52pojie.cn/forum/202408/19/151948qnrutixbilis8hwq.png)
剪贴板 2024-08-19 下午 3.19.51.png (60.54 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjcxODYyN3w4ZTdhYTMxYnwxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxODYyN3w4ZTdhYTMxYnwxNzkzNTcxODk0fDE2ODA4NnwxOTU2MDAx&nothumb=yes)
2024-8-19 15:19 上传

## 3、此方法为临时解决方法，可能存在一些额外问题，等待论坛大神提供更好的方法
