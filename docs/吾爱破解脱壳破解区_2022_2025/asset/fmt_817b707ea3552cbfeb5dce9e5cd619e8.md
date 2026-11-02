
# [.NET逆向] 分析一个.NET CM

- 原帖：`https://www.52pojie.cn/forum.php?mod=viewthread&tid=1609112`
- 抓取时间：2026-11-01T23:31:55

原文链接：[https://www.52pojie.cn/thread-1608459-1-1.html](https://www.52pojie.cn/thread-1608459-1-1.html)

## 一、准备工作

目标软件是一个加了`VMP`并开启虚拟化及代码变异，[论坛链接](https://www.52pojie.cn/forum-5-1.html)过程略，直接看脱壳的结果。

![11.png](https://attach.52pojie.cn/forum/202203/23/133212y8ixdxshxuu1uze8.png)

11.png (111.68 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNjYxM3xhMzNhYjViY3wxNzkzNTc1OTEyfDE2ODA4NnwxNjA5MTEy&nothumb=yes)

2022-3-23 13:32 上传

可以看到关键地方全部被虚拟化了，关于虚拟化处理起来我还没有很好的解决方案。

## 二、去除反调试

`timer1`里有进程检测反调试，每2秒检测是否包含`{“dnSpy”，“de4dot”，“64”，“32”，“ExtremeDumper”}`字样，将`timer1`时钟设置为`flase`即可关闭检测。

![22.png](https://attach.52pojie.cn/forum/202203/23/133642o7lf47zl4s54vit1.png)

22.png (16.43 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNjYxNXxiMzI4OTJkZnwxNzkzNTc1OTEyfDE2ODA4NnwxNjA5MTEy&nothumb=yes)

2022-3-23 13:36 上传

`de4dot`直接修改会清除`vmp`区段，这里我直接跳转到16禁止去`c32`修改的，将`0xC98F6`这里修改为0就可以啦。

![33.png](https://attach.52pojie.cn/forum/202203/23/134043p30vozqfz102f2lz.png)

33.png (21.91 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNjYxNnwzOWQzYTdhY3wxNzkzNTc1OTEyfDE2ODA4NnwxNjA5MTEy&nothumb=yes)

2022-3-23 13:40 上传

## 三、追码

解决反调试后我找了个软件来帮助解决虚拟化代码的问题，下面贴软件生成的log。

[Asm] 纯文本查看 复制代码
