
# [PC样本分析] 蠕虫病毒伪装传播，根目录文件遭神秘删除

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1940534
- 抓取时间: 2026-11-01T23:47:42

近期，火绒威胁情报中心监测到一款通过改名为`USB Disk`来欺骗用户执行的蠕虫病毒正在传播，火绒安全工程师第一时间提取样本进行分析。分析过程中发现该程序在后台自动感染每个插入的可移动磁盘，并将原文件移动到一个隐藏文件夹中。随后，程序创建一个名为“`USB Disk.exe`”的文件，诱导用户点击以执行病毒程序。同时，程序还会启动后门进程，试图控制用户主机成为自己的肉鸡。目前，火绒安全产品可对上述病毒进行拦截查杀，请广大用户及时更新病毒库以提高防御能力。

![000.png](https://attach.52pojie.cn/forum/202407/03/190815g3ey3ctcyc3x5f44.png)

000.png (28.56 KB, 下载次数: 0) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUwOXw1NGUwNDA4MXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 19:08 上传

火绒6.0查杀图

样本执行流程图如下所示：

![Image-1.png](https://attach.52pojie.cn/forum/202407/03/160517n9yamy8987gcl3u5.png)

Image-1.png (175.94 KB, 下载次数: 3) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxMHxhYWY3YTU2MXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:05 上传

流程图

在此，火绒工程师提醒大家对来历不明的文件应保持警惕，同时安装可靠的安全软件保护设备免受恶意软件和病毒的侵害。目前，火绒6.0已正式上线，综合用户的实际场景、使用感受和建议等多个方面，在形式、功能和操作等方面均进行了改进与创新，升级反病毒引擎等核心技术，精细化病毒查杀设置，增加更多威胁检测点，为“杀、防、管、控”增强壁垒。欢迎大家前往火绒官方网站下载体验。

## 一、样本分析

分析得出该病毒通过多个模块协同工作，通过白加黑的手段，具备较强的隐蔽性。其包含U盘传播、[检测](https://www.52pojie.cn/thread-661779-1-1.html)、自启动设置、文件管理和`CMD`后门等多种恶意行为。

其中白文件是`wwntray.exe`（第二启动器）、`Transfer.exe`（蠕虫启动器）、`Permissions.exe`（后门启动器），它们主要是通过调用dll导出函数的方式执行恶意代码。

样本`USB Disk.exe`在刚被制作出来时并不能完整运行，因为需要满足多种条件才可执行，所以需要通过`transfer.exe`启动蠕虫并感染第一个可移动磁盘，从而“帮助”它满足条件才可以真正开始传播。

传播的方式是将`USB Disk.exe`放入到可移动磁盘中的根目录下，同目录下其他六个文件放入到可移动磁盘的隐藏目录中。之后在其他主机上使用该可移动硬盘并运行`USB Disk.exe`时，总启动器`USB Disk.exe`就会执行同目录下病毒文件夹中的`wwntray.exe`实现从可移动磁盘逆感染主机的操作，此时被感染的主机就会在后台执行蠕虫模块和后门模块，使每一个插入该主机的可移动磁盘都会被感染，重复第一次`transfer.exe`完成的操作。

### 初始化

病毒在运行蠕虫模块和后门模块前会进行一系列检查、持久化设置、删除部分文件等操作。

样本会在感染U盘时，将程序改名为`USB Disk.exe`，以诱导用户点击并执行。所以当`USB Disk.exe`程序在用户的U盘中出现时，意味着该U盘可能已被该病毒感染。

#### `USB Disk.exe`

该程序首先通过`GetModuleFileName`检查当前目录是否为盘符的根目录，如果不是，则结束运行。

随后，通过`CreateProcess`启动`explorer.exe`，参数为藏有U盘原文件的目录，此操作的主要目的是打开并展示受害者U盘中的原始文件，从而试图让受害者误以为自己仅仅是打开了USB Disk文件夹而不是病毒。

![Image-2.png](https://attach.52pojie.cn/forum/202407/03/160519mf9909bw00eatb6f.png)

Image-2.png (26.24 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxMXw0OWU5MmYwMHwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:05 上传

打开原始文件目录

![Image-3.png](https://attach.52pojie.cn/forum/202407/03/160521bllwrzf8r3lhv6y8.png)

Image-3.png (44.4 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxMnwwZTUxZDNjYXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:05 上传

U 盘中原始文件目录

接着通过传入字符串参数`(C:)`来寻找磁盘根目录窗口并发送关闭窗口指令，目的是为了让受害者感受到打开的就是一个普通的文件夹。

![Image-4.png](https://attach.52pojie.cn/forum/202407/03/160523e6a66536aonwvcrg.png)

Image-4.png (26.54 KB, 下载次数: 1) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxM3w0OGVhYzAxNXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:05 上传

关闭磁盘根目录窗口

并通过`OpenEvent`参数`ConfigurantionDebug4`防止程序多开`wwnotray.exe`，随后执行`wwnotray.exe`，参数为`-debug`。

![Image-5.png](https://attach.52pojie.cn/forum/202407/03/160525p30qqwscxqoxc8b8.png)

Image-5.png (17.73 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxNHw1MzVmN2IxMHwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:05 上传

执行 wwnotray.exe

#### `wwnotray.exe`

`wwnotray.exe`加载`wweb32.dll`并执行该dll导出表中的`ShowModalWordWebWEx`函数，该函数包括持久化操作、执行蠕虫和后门模块等功能。

![Image-6.png](https://attach.52pojie.cn/forum/202407/03/160528gr0i1ikr9kfog4oy.png)

Image-6.png (54.82 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxNXxmNGI5N2M5OHwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:05 上传

执行 ShowModalWordWebWEx 函数

程序首先通过`CreateEvent`检测事件是否已经存在来防止多开。

![Image-7.png](https://attach.52pojie.cn/forum/202407/03/160604lur103c90icc71i7.png)

Image-7.png (59.1 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxNnxhODhmZWIxZXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

防止多开

动态获取函数地址：

![Image-8.png](https://attach.52pojie.cn/forum/202407/03/160606tskc74a4geke797q.png)

Image-8.png (23.36 KB, 下载次数: 1) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxN3w2ZDNjMDgyZHwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

动态获取函数地址

动态获取函数地址方法如下，主要原理是通过比对函数名Hash值来获取函数地址。

![Image-9.png](https://attach.52pojie.cn/forum/202407/03/160608juofvlrqovdsdzs5.png)

Image-9.png (63.78 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxOHwwZDEwNjE5OHwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

获取 Windows API 函数地址函数

其执行时会判断参数是否为`-debug`，以及路径中是否包含`$WinUpdate`，否则直接结束进程。

![Image-10.png](https://attach.52pojie.cn/forum/202407/03/160610uribykdby44qdyy6.png)

Image-10.png (20.83 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUxOXw5NDQxNTExYXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

判断逻辑

随后样本开始删除文件`C:\users\Public\GBPRSMBAL\WaveEditor.exe`和`C:\users\Public\GBPRSMBAL\WaveeditNero.exe`，根据相关样本推测，这可能是老版本病毒文件名，进行清除。

![Image-11.png](https://attach.52pojie.cn/forum/202407/03/160612dd7byzc6eh7bva2h.png)

Image-11.png (18.22 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUyMHw3NmNjNDAyYXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

删除文件

![Image-12.png](https://attach.52pojie.cn/forum/202407/03/160615v47kov3knkcqbc77.png)

Image-12.png (22.96 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUyMXwyODVhMTM2NnwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

删除路径列表

还会删除目录，推测可能也是清除老版本的操作。

![Image-13.png](https://attach.52pojie.cn/forum/202407/03/160617w6a3r6kuiabkilib.png)

Image-13.png (16.71 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUyMnw1NWI0ZjQ3MHwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:06 上传

删除目录

![Image-14.png](https://attach.52pojie.cn/forum/202407/03/160842unv50ni00qfh5ozz.png)

Image-14.png (15.29 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNTUyM3wzMzA1MDZjYXwxNzkzNTc2ODU5fDE2ODA4NnwxOTQwNTM0&nothumb=yes)

2024-7-3 16:08 上传

删除目录列表

删除目录的操作是通过`cmd`命令加上参数`/c rd/s/q`来完成的。

![Image-15.png](https://attach.52pojie.cn/forum/202407/03/160844ofzcpfgutnrpp8yy.png)

Image-15.png (40.21 KB, 下载次数: 2)

