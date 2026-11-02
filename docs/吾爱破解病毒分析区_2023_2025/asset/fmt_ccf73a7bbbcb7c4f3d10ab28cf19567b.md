
## PSB 数据包

`PSB`数据包作为追踪包解码的同步标志，定义了追踪日志中的一个边界。在此处，解压缩过程可以独立进行，不会产生其他影响。在`libipt`库中，这个偏移量被称为“同步偏移量”（`sync offset`），因为它标志着追踪文件中的一个位置，从这个位置开始可以安全地解码后续的数据包。

## TIP（目标 IP）

`TIP`数据包表示目标 IP 地址。????????压缩省略地址高四字节。

## TNT（条件分支执行状态）

`TNT`数据包用于指示某个条件分支是否被执行。无条件跳转的分支不会被记录，因为这些流程控制可以从程序的状态中推导出来。

## 解包数据

解包数据如下：

![](https://attach.52pojie.cn/forum/202412/27/135625w019x54woz7ehtto.png)

Image-26.png (56.29 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NjYwN3xiOTg2MzE5OXwxNzkzNTc2NzY4fDE2ODA4NnwxOTk1MDk2&nothumb=yes)

2024-12-27 13:56 上传

通过解包数据，可以发现程序在执行过程中的`TIP`跳转到`7FFFFFFFF0a26c90`以及`0x405e0`偏移处存在的`TNT`跳转，这些信息有助于我们进一步借助代码实现更深入的分析。

## 数据包分析

解包后的数据包通过 IDA 进行分析，可使我们更方便地分析代码的执行流程及计算代码覆盖率。

![](https://attach.52pojie.cn/forum/202412/27/135708mqbb7qib0pbii2qg.png)

020.png (116.48 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NjYwOHwyODlkMzBkMnwxNzkzNTc2NzY4fDE2ODA4NnwxOTk1MDk2&nothumb=yes)

2024-12-27 13:57 上传

## 火绒内部工具

### 虚拟沙盒

火绒虚拟沙盒实现了对超过数万个 API 的模拟，覆盖了绝大多数操作系统的核心机制，并支持多个操作系统平台：

- Windows x86/x64
- Linux x86/x64
- MacOS x86/x64

虚拟沙盒支持病毒查杀并对病毒进行追踪。通过虚拟化执行引擎，能够为目标代码划分独立的地址空间，并通过接管中断和异常为目标代码分配私有时间片，从而使目标代码得以受控制，执行效率几乎可以达到与真实机相当，唯一的开销来自于执行环境的切换。且通过模拟目标代码的执行，可以快速进行通用脱壳和`dump`区段，免去了对强壳外壳的复杂分析工作。

### 火绒查杀引擎与调试器集合

火绒查杀引擎与调试器集合实现了对病毒的追踪，脱壳并自动修复 IAT，自动运行到调用 API，单步步过显示调用的 API。

![](https://attach.52pojie.cn/forum/202412/27/135833rtnaflapaf87c0d8.gif)

Image-27.1 6.引擎模拟api_2.gif (29.16 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NjYwOXxiODdmMWYzZnwxNzkzNTc2NzY4fDE2ODA4NnwxOTk1MDk2&nothumb=yes)

2024-12-27 13:58 上传

模拟输出调用 API 演示

![](https://attach.52pojie.cn/forum/202412/27/135837kx1g4aivfzvni746.gif)

Image-29.1 7.引擎追踪至跨节_2.gif (42.78 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NjYxMXw3MTFkYzAyNXwxNzkzNTc2NzY4fDE2ODA4NnwxOTk1MDk2&nothumb=yes)

2024-12-27 13:58 上传

引擎追踪至跨节区演示

![](https://attach.52pojie.cn/forum/202412/27/184109n1vtsyytchcl7248.gif)

8.gif (180.8 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NjY1OXw4NTM0ZThlZHwxNzkzNTc2NzY4fDE2ODA4NnwxOTk1MDk2&nothumb=yes)

2024-12-27 18:41 上传

Dump 脱壳内存映像自动修复 IAT 演示

今天的技术分享到这里就结束啦，希望本篇文章，可以对大家探索病毒分析领域起到一定帮助。绒绒相信，这些工具和技术不仅仅是我们对抗恶意软件的利器，更是我们解码溯源、守卫网络安全的法宝。

在这个信息安全越来越被重视的时代，每一次技术的进步和应用都显得尤为重要，网络安全也不仅仅是专业人士的责任，更是与我们每个人的生活都息息相关。让我们一起筑起防护之盾，为整个网络空间的和谐与安全贡献力量。

最后，绒绒也要感谢大家的陪伴和支持，正是因为有了你们的好奇心和探索欲，我们的分享才更有意义。未来，绒绒会继续为大家挖掘更多网络安全的“宝藏”，分享更多有价值的技术知识，敬请期待吧~
