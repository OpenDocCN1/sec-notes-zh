
# [原创] 高通芯片平台PCAT使用限制破解思路

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1725411
- 抓取时间：2026-11-01T23:03:53

郑重声明：本帖只供交流学习，为了避免不必要的麻烦，不提供源文件和处理后的程序。并且只给出最主要的步骤。

## 背景

PCAT是高通平台进行NV设置、MBN参数设置等很重要的工具。它依赖于高通的很多license。尤其是QUTS的license会间接影响PCAT的使用。那么今天就重点给出当QUTS和PCAT的license都无效时，应该如何解除这种限制的思路。

## 定位关键分支

采用`IDA`打开`PCATAPP.EXE`。

根据`PCATAPP.EXE`的错误提示，搜索如下关键词：

找到如下分支，将`brfalse.s`编码为`brtrue.s`。

![](https://attach.52pojie.cn/forum/202212/11/113146peci9398o5iie7o5.png)

`2.png`（429.72 KB，下载次数：1）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODM0Mnw3NzU3ODZjN3wxNzkzNTc0MjMwfDE2ODA4NnwxNzI1NDEx&nothumb=yes)

2022-12-11 11:31 上传

运行`patch`，然后尝试运行被`patch`后的程序，我们惊喜发现程序主界面出来了。

但是别高兴太早，你可以看到下面的提示，所有的license还是无效的。

找到如下关键点：

![](https://attach.52pojie.cn/forum/202212/11/113646mbbva1ikb7u1yeec.png)

`5.png`（364.02 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODM0NnxjOGRiNmYyYXwxNzkzNTc0MjMwfDE2ODA4NnwxNzI1NDEx&nothumb=yes)

2022-12-11 11:36 上传

把`brtrue`编码为`brfalse`，也就是改为让license无效的时候继续工作。

`patch`程序后重新运行，可见license有效了。但是仍然不识别调试端口。

## QUTS 与调试端口

剩下的工作就是无论你怎么[//www.52pojie.cn](https://www.52pojie.cn)，都无法获得调试口的链接信息，这是为什么呢，这是因为你的思路错了，此时是因为QUTS没有破解。当你完成了QUTS的破解，就可以得到如下的结果。QUTS将在后续的帖子中再探讨。

如下是整体破解后的运行图。可见，所有的功能都能脱离license使用。

![](https://attach.52pojie.cn/forum/202212/11/115651ng8ajx8xqqfaakqd.png)

`11111.png`（56.55 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODM1NnxmNWE2MzdlMnwxNzkzNTc0MjMwfDE2ODA4NnwxNzI1NDEx&nothumb=yes)

2022-12-11 11:56 上传

## 扩展思路

以上给出的是基础版的破解思路，其实可以有更多可以发挥的地方，比如与其将`brfalse`和`brtrue`进行调换，就不如把某些你不需要达到代码进行`nop`掉。当然是要注意`stack`平衡的。如果你把不需要的代码`nop`掉，就可以保证不管你有没有license都可以正常运行。否则你可能会遇到如果你加载了license反而无法工作的情况。
