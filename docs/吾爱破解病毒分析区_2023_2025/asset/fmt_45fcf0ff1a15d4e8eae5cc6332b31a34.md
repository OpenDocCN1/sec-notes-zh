
# [PC样本分析] 回顾：后门病毒Gh0st近年变种演化

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1823611
- 抓取时间: 2026-11-01T23:49:14

网络安全的核心本质是攻防对抗。当防病毒技术在不断完善的同时，病毒也在不断设法予以对抗，夹缝求生。Gh0st后门病毒就是其中之一。Gh0st 是一种远程访问工具（RAT），最早出现在2001 年左右，通过远程控制受感染计算机来执行各种恶意活动。其发展历史与网络攻击、渗透测试和间谍活动紧密相连，并且相关代码已经开源，致使对应变种层出不穷，对用户造成了较大的威胁。据“火绒威胁情报系统”显示，Gh0st 在国内常见的后门病毒中占比最大，超过50%。

![](https://attach.52pojie.cn/forum/202308/21/095834oprkkccxr4bkzkxm.png)

Image-0.png (204.37 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4M3w5YWI2MGE5ZnwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:58 上传

数据统计图

传统的 Gh0st 由控制器和服务器组成，其功能模块多以插件形式下发，包含如下功能：

- 键盘记录
- 远程终端访问
- 远程音频和视频访问
- 文件管理
- 远程文件下载和执行
- 进程资源管理器和其他系统枚举功能
- 图形用户界面交互（远程控制）
- 自我更新
- 重置 SSDT 以删除现有挂钩

![](https://attach.52pojie.cn/forum/202308/21/095846xyw002v6t0l5yl2l.png)

Image-1.png (99.44 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4NHxjNzkyNjY1N3wxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:58 上传

Gh0st木马截图

火绒工程师在分析该病毒的对抗历程和变化特征时发现，该病毒不仅持续更新免杀对抗手段，而且传播途径花样百出，前后发生至少五次主要变种：

## 第一次变种

黑客通过攻击用户 SQL Server 服务器的方式，利用数据库相关进程下载并执行 Gh0st 后门病毒的早期变种—— “神农远控” ，火绒安全产品及时感知并拦截了该变种。

![](https://attach.52pojie.cn/forum/202308/21/095857ic4t4zwx8bwt912b.png)

Image-2.png (63.42 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4NXxmNmM4NzMzYXwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:58 上传

火绒查杀图

不同于原始 Gh0st 的单一执行方式和注册表的简单利用，该变种除了会在不同的操作系统（`InstallTime`、`WOW64` 等）中执行不同的病毒逻辑外，还用于感染后的信息交互。

![](https://attach.52pojie.cn/forum/202308/21/095907jxhkq484uxuuufhh.png)

Image-3.png (54.88 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4NnxjNzFkN2Y4MHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:59 上传

部分操作截图

在前期信息收集的过程中，该变种除了原有的硬件信息外，还添加了对各种杀软和敏感软件的检测，以供控制端调整执行策略。

![](https://attach.52pojie.cn/forum/202308/21/095916hszvtzrrnk5rouxw.png)

Image-4.png (78.3 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4N3xkY2Y3YjU3MHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:59 上传

软件遍历

在通信方面，一改传统 5 字节 `"Gh0st"` 默认开头和 13 字节特征数据头，通过添加固定的 `HTTP[/VX/WXWork/1688854992357587/Cache/File/2023-08/WWW.NCBUG.COM` 绕开 Gh0st 的流量端的关键字匹配。对于回传的数据部分，更是直接自定义了要执行的 shellcode 代码，用于指定各种自定义操作。

![](https://attach.52pojie.cn/forum/202308/21/095927tjo51ul1rojj7izl.png)

Image-5.png (45.81 KB, 下载次数: 1)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4OHwwN2Q3MzAzZHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:59 上传

代码部分截图及完全功能展示

相关文章链接：

https://www.huorong.cn/info/1556281170207.html

## 第二次变种

在供应链污染问题上，“火绒威胁情报系统”同样发现了 Gh0st 变种的利用踪迹。一款名为 `HellohaoOCR_V3.1` 的图像识别程序经分析携带着后门病毒 `Scvhost.exe`（Gh0st 的变种），该程序作者由于利用了第三方的易语言导致编译环境被感染，从而导致病毒被不断扩散。

![](https://attach.52pojie.cn/forum/202308/21/095940m7jsz4z44z4ajlr9.png)

Image-6.png (107.21 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM4OXxiY2FlNjI3OHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:59 上传

火绒查杀图

`Scvhost.exe` 作为 Gh0st 的一个变种，除了通过传统的 C&C 服务器接收上线消息通知外，还添加了可以通过 QQ 上线的方式。其原理是通过 QQ 一个公开接口来获取加密之后的其他 C&C 服务器地址和端口，以此变换服务器地址并绕过杀软 IP 封禁。

![](https://attach.52pojie.cn/forum/202308/21/095951rzsmp6axf69uu8af.png)

Image-7.png (136.66 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM5MHw0OGZjMDk5ZHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 09:59 上传

远控配置界面

该变种在执行层面上，继承了前面已提到过的杀软对抗，shellcode 代码执行等特点。但在通信操作上，除了前面提到的 QQ 上线外，该变种会通过第三方网站 `ip.cn` 来获取真实的 C&C 服务器地址，确保获取到的 C&C 服务器地址不会受到当前网络环境影响（比如 DNS 劫持）。

![](https://attach.52pojie.cn/forum/202308/21/100002ymzu6v3svxm48zxv.png)

Image-8.png (92.7 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM5MXxlNDc2ZjQwYXwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 10:00 上传

相关代码及功能展示

相关文章链接：

https://www.huorong.cn/info/1572352454382.html

## 第三次变种

黑灰产软件往往是病毒聚集的重灾区，Gh0st 变种也将目光瞄准了此处。火绒安全工程师根据用户反馈和“火绒威胁情报系统”监测，发现 Gh0st 的变种通过 “穿越火线” 等多款游戏外挂传播，并通过 QQ 群、网盘等渠道持续扩散。

![](https://attach.52pojie.cn/forum/202308/21/100014sfvt59ao3f5y442i.png)

Image-9.png (65.58 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM5MnxiMGZlMjRlNXwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 10:00 上传

火绒查杀图

在本次事件中 Gh0st 变种的执行依托于父文件 “白加黑” 的攻击组合，经过多层 PE 流调用和内层解密逻辑，最终通过动态库的导出函数调用内存中注入的 Gh0st 变种：

![](https://attach.52pojie.cn/forum/202308/21/100023xl87j43bf1x1iqyy.png)

Image-10.png (95.84 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM5M3xkMTZiOWRlNHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 10:00 上传

相关代码及功能展示

与上面例子中提到的后续操作中不同的是，该变种直接下发勒索病毒进行全盘文件加密，还会在遍历检测杀毒软件的过程中，通过控制端消息弹窗功能对中毒用户进行恐吓“别杀毒了，木有用”：

![](https://attach.52pojie.cn/forum/202308/21/100035r82w2nalnrnjgjg4.png)

Image-11.png (187.77 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM5NHwyYzE0NGU1ZHwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 10:00 上传

相关代码及功能展示

相关文章链接：

https://www.huorong.cn/info/1587129271457.html

## 第四次变种

Gh0st 变种不断在免杀对抗方面加强。火绒安全工程师在用户感染的电脑中发现了蠕虫病毒，根据相关威胁信息展开溯源分析，最终发现是以 Gh0st 变种后门为 "主"，蠕虫为 "辅" 的恶意控制行为。

![](https://attach.52pojie.cn/forum/202308/21/100046vtppzw27j0vu2eet.png)

Image-12.png (130.99 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNzM5NXwzZTY5ZTc4NnwxNzkzNTc2OTUxfDE2ODA4NnwxODIzNjEx&nothumb=yes

2023-8-21 10:00 上传

火绒查杀图

该 Gh0st 的变种在注册修改，杀软遍历以及自定义的后门控制功能上，与第一个例子（https://www.huorong.cn/info/1556281170207.html）文章中分析的变种相差无几，根据连接的域名的同源性以及 DDOS 模块的移除（DDOS 网站已过期），可以判断是同一样本的不断改进。

![](https://attach.52pojie.cn/forum/202308/21/100056yb21ua17nj57zn2u.png)

Image-13.png (91.01 KB, 下载次数: 0)
