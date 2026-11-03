
# [原创] 【原创】探秘在高通平台苹果和三星手机的eSIM的交互过程及其工具研究

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=2040512](https://www.52pojie.cn/forum.php?mod=viewthread&tid=2040512)
- 抓取时间：2026-11-01T22:07:54

## 背景

eSIM 作为一种将来非常灵活的运营商业务，其中无需换卡就可以更换运营商。目前一些大品牌的手机已经将其中一张卡从物理实体SIM卡变成eSIM卡。eSIM的写卡过程通过运营商的网络下发数据。这个过程到底是怎样的呢，本文将采用IPhone13 IOS 5.2和 Samsung Galaxy Z Fold4的eSIM写卡过程中抓取`QXDM`的日志，解密如何分析eSIM的交互过程，以及如何进行相关PDU的解码过程。

## 准备工作

采用特定手段切出`diag`口，连接`qxdm`抓取日志，然后放到`QCAT`中过滤，`0x1987` log id，另存为`txt`作为后续分析的输入。如下图是IPhone 13的日志示例，搜索`Slot Id`的所有字段，可以看到这个log中包含了两个卡槽的日志。其中`slot_2`为eSIM，`slot_1`是实体SIM。

![](https://attach.52pojie.cn/forum/202506/21/135541m8n22ggo4ddvssow.png)

`image.png`（70.73 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM2OXwzMTkyMDBmZXwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:55 上传

我们重点关注的是`Slot2`的数据，所以需要制作一个工具将两个卡槽的日志进行分类整理。并且将其中关键的数据交互提取出来。

## 算法设计过程

程序设计思路如下：

1. 采用`Slot Id = SLOT_1`和`Slot Id = SLOT_2`作为过滤不同卡槽日志的关键词。
2. 根据`ETSI TS 102 221 V17.4.0`协议找到对应命令字节转换为相应的命令含义，在日志中找到5字节长度的`command APDU`，取出第二个字节作为`Command`（协议称为`Instruction`）字节，这个字节代表了此命令的含义。比如`E2`就是“`STORE DATA`”指令。

![](https://attach.52pojie.cn/forum/202506/21/135618w1vzc38lrl3rrcdy.png)

`image.png`（34.49 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3MHxjOWM2MzNiZnwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:56 上传

根据协议将常用的字节指令用`python`字典翻译如下:

![](https://attach.52pojie.cn/forum/202506/21/135655rdoc3o1r0h944e3n.png)

`image.png`（161.11 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3MXw3ZmM2NTlhZHwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:56 上传

![](https://attach.52pojie.cn/forum/202506/21/135717ieocbhd7py77ccc8.png)

`image.png`（38.06 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3MnxmMTI3MmRlOXwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:57 上传

将直到遇到下一个已知指令之前的所有`RX`和`TX`交互过程进行整合为一个`Session`。核心代码如下:

![](https://attach.52pojie.cn/forum/202506/21/135743xt1bpt0z0t6o4mo0.png)

`image.png`（85.27 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3M3xmMjhmMzViMHwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:57 上传

![](https://attach.52pojie.cn/forum/202506/21/135807fnnlo8x63ooibtti.png)

`image.png`（84.71 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3NHwzMGIyM2QxN3wxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:58 上传

![](https://attach.52pojie.cn/forum/202506/21/135824rgyny9nynnhryywp.png)

`image.png`（78.94 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3NXxjMjlhMDE4M3wxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:58 上传

对于常见的文件单元内容进行解析，示例如下:

![](https://attach.52pojie.cn/forum/202506/21/135848dgulfifmqvvglmue.png)

`image.png`（93.14 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3NnxiNGI2NzBiYXwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:58 上传

![](https://attach.52pojie.cn/forum/202506/21/135904m7auqnvuvnqenqgz.png)

`image.png`（147.54 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3N3w1MTFmZTcxZXwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:59 上传

## 实际验证

加载之前存储人IPhone13的eSIM进行OTA的过程如下，可以看到基本的分析过程和下载的eSIM关键参数:从工具分析来看，前面有一系列的数据下载过程`STORE DATA`。随后就是按照UICC的APDU协议进行从参数的更新。

![](https://attach.52pojie.cn/forum/202506/21/135938ppf0ksksssisiscs.png)

`image.png`（80.61 KB，下载: 1）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM3OHw0YWVmMWFkN3wxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 13:59 上传

以`Terminal Profile`为例，点击`Terminal Profile`，我们可以看到终端向eSIM发送的数据如红色圈所示:

![](https://attach.52pojie.cn/forum/202506/21/140103hi2uun0r0msfete2.png)

`image.png`（51.26 KB，下载: 0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM4MHw5NDVmMGU4NXwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 14:01 上传

```
80 10 00 00 2D FF FF FF FF 7F 9D 00 DF BF 00 00 1F E2 00 00 00 C3 F0 00 07 00 01 68 00 51 01 00 00 00 08 02 80 07 70 06 03 00 00 00 00 00 00 00 00 A0
```

将红圈的部分拷贝出来，放到更专业的SIM APDU解码器中进行解码，可以解码出此款手机的支持对于SIM卡方面功能的支持能力如下. 这就相当于 [www.52pojie.cn](https://www.52pojie.cn) 出了针对这张eSIM的话，手机都支持什么能力。

```
1000 .... = Class Coding: ETSI TS 102.221 (0x8)

.... 00.. = Secure Messaging Indication: No SM used between terminal and card (0x0)

.... ..00 = Logical Channel number: 0

Instruction: TERMINAL PROFILE (0x10)

Terminal Profile Byte 1 (Download): 0xff, Profile Download, SMS-PP Data Download, CB Data Download, Menu Selection, SMS-PP data download is supported, Timer expiration, Call Control by USIM is supported, Call Control by USIM is supported

.... ...1 = Profile Download: Supported

.... ..1. = SMS-PP Data Download: Supported

.... .1.. = CB Data Download: Supported

.... 1... = Menu Selection: Supported

...1 .... = SMS-PP data download is supported: Yes

..1. .... = Timer expiration: Supported

.1.. .... = Call Control by USIM is supported: Yes

1... .... = Call Control by USIM is supported: Yes

Terminal Profile Byte 2 (Other): 0xff, Command result, Call Control by USIM, Call Control by USIM is supported, MO SMS control by SIM, Call Control by USIM is supported, UCS2 Entry, UCS2 Display, Display of Extension Text

.... ...1 = Command result: Supported

.... ..1. = Call Control by USIM: Supported

.... .1.. = Call Control by USIM is supported: Yes

.... 1... = MO SMS control by SIM: Supported

...1 .... = Call Control by USIM is supported: Yes

..1. .... = UCS2 Entry: Supported

.1.. .... = UCS2 Display: Supported

1... .... = Display of Extension Text: Supported

Terminal Profile Byte 3 (Proactive SIM): 0xff, Proactive SIM: DISPLAY TEXT, Proactive SIM: GET INKEY, Proactive SIM: GET INPUT, Proactive SIM: MORE TIME, Proactive SIM: PLAY TONE, Proactive SIM: POLL INTERVAL, Proactive SIM: POLLING OFF

.... ...1 = Proactive SIM: DISPLAY TEXT: Supported

.... ..1. = Proactive SIM: GET INKEY: Supported

.... .1.. = Proactive SIM: GET INPUT: Supported

.... 1... = Proactive SIM: MORE TIME: Supported

...1 .... = Proactive SIM: PLAY TONE: Supported

..1. .... = Proactive SIM: POLL INTERVAL: Supported

.1.. .... = Proactive SIM: POLLING OFF: Supported

1... .... = Proactive SIM: REFRESH: Supported

[鈥�]Terminal Profile Byte 4 (Proactive SIM): 0xff, Proactive SIM: SELECT ITEM, Proactive SIM: SEND SHORT MESSAGE, Proactive SIM: SEND SS, Proactive SIM: SEND USSD, Proactive SIM: SET UP CALL, Proactive SIM: SET UP MENU, Proactive SIM: PR

.... ...1 = Proactive SIM: SELECT ITEM: Supported

.... ..1. = Proactive SIM: SEND SHORT MESSAGE: Supported

.... .1.. = Proactive SIM: SEND SS: Supported

.... 1... = Proactive SIM: SEND USSD: Supported
```
