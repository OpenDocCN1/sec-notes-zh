
Image-17.png (387.3 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg0NXwyYTM1OGU3Y3wxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:42 上传

关闭窗口

如果安全软件不存在，则会将 `'C:\\'`、`'C:\\ProgramData'`、`'C:\\Users'`、`'C:\\Program Files (x86)'` 路径添加到 Windows Defender 白名单中，以躲避检测。

![](https://attach.52pojie.cn/forum/202409/29/144233x7rrtgg0rrzhrrrz.png)

Image-18.png (36.6 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg0NnwxYWIzYTAxNHwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:42 上传

添加白名单

下载 `PACqpC.exe`

完成上述一系列检查后，该样本会下载一些文件和 Shellcode ，比如：`TrueSightKiller` 驱动程序和用于创建计划任务的 Shellcode 等。

大致流程如下：解密链接，利用链接下载内容，并使用流密码进行解密，之后创建文件夹并隐藏文件夹。

![](https://attach.52pojie.cn/forum/202409/29/144243fw80gzfwlehgfl8l.png)

Image-19.png (84.66 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg0N3wzOTRkYjgyZXwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:42 上传

具体流程

样本在此流程中，利用 `InternetReadFile` API 函数下载数据。

![](https://attach.52pojie.cn/forum/202409/29/144252spu0qpppqz0yypbv.png)

Image-20.png (96.09 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg0OHwxY2JhMDZiMnwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:42 上传

下载数据

其中，下载链接和下载的数据分别利用凯撒密码和简单流密码进行解密。

![](https://attach.52pojie.cn/forum/202409/29/144307nrrnppluln1rxnln.png)

Image-21.png (104.89 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg0OXw4NzU4ZTZkZnwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:43 上传

解密算法

解密后的 URL 数据如图所示，包含下载链接和相对应的文件名。

![](https://attach.52pojie.cn/forum/202409/29/144320bfsilyirhqvh1lqd.png)

Image-22.png (46.06 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1MHw5ZmZkNjU2N3wxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:43 上传

URL 数据

随后检查是否有安全软件。

![](https://attach.52pojie.cn/forum/202409/29/144349go48nn9e4b4nc0n3.png)

Image-23.png (86.47 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1MXwwZjhiN2NhZXwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:43 上传

检查安全软件是否存在

如果检查到安全软件，将保持之后下载到的数据解密并写入文件后的文件句柄处于开启状态，以此阻止安全软件的读取。

![](https://attach.52pojie.cn/forum/202409/29/144402vlyxp3lma13rgbrx.png)

Image-24.png (58.94 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1MnwyYjFiZWM4MXwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:44 上传

写入文件

下载后开始分析数据，并提取出异或秘钥和数据开头地址。

![](https://attach.52pojie.cn/forum/202409/29/144414uiaj1t5ha5511z1h.png)

Image-25.png (35.15 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1M3xjNmU5MjdjN3wxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:44 上传

分析数据

开始下载 `a.gif`、`b.gif`、`c.gif`、`d.gif` ，它们分别对应 `install.exe`（即随机名 `PACqpC.exe` ）、`hccutils.dll`、`1.gif`、`2.jpg` 。这些文件中的最后一个字节用作异或解密的秘钥，倒数第五个字节至倒数第二个字节用于计算出数据开头偏移，其中 `b.gif` 的最后四位会写入随机数。

![](https://attach.52pojie.cn/forum/202409/29/144427gd9fpk319i1f1e6f.png)

Image-26.png (95.76 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1NHw1ZjkwOTRlYXwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:44 上传

整体逻辑

![](https://attach.52pojie.cn/forum/202409/29/144439v81f53tv35d7titv.png)

Image-27.png (116.98 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1NXw2MWM5ZTMzOHwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:44 上传

异或、随机数、获取秘钥和偏移

随后会读取自身内容，并利用 `*/&` 定位，将 32 位随机数+`*/&`之后的 36 位数据写入到 `C:\Users\InstallUP.ini` 文件中。这些数据是被加密的 IP 和端口，用于下载后门模块后的动态替换 IP 和端口。

![](https://attach.52pojie.cn/forum/202409/29/144450v95nlqyszdifyodd.png)

Image-28.png (63.51 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1Nnw3ZDFiY2VlZHwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:44 上传

`InstallUP.ini` 文件

接着通过链接 `https://197oss.oss-cn-beijing.aliyuncs.com/s.dat` 下载 `TrueSightKiller` 驱动，并将此驱动写入到 `C:\Users\TTruespanl.sys` 中，同时保持文件句柄处于开启状态。该驱动用于关闭杀毒软件。

![](https://attach.52pojie.cn/forum/202409/29/144502fvmlnx1omcvvdvde.png)

Image-29.png (47.12 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1N3xhMDk1NTA1NnwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:45 上传

下载驱动

之后通过链接 `https://197oss.oss-cn-beijing.aliyuncs.com/s.jpg` 下载 Shellcode ，通过利用 `VirtualProtect` 设置执行权限并执行。

![](https://attach.52pojie.cn/forum/202409/29/144517z76812lj66f7thhh.png)

Image-30.png (69.32 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1OHw2ZWI2NjM2MHwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:45 上传

下载执行 Shellcode

Shellcode 中存在被压缩的 Dll ，对其进行解压缩后手动加载，并从中获取 `RegisterTask` 函数地址。

![](https://attach.52pojie.cn/forum/202409/29/144529mhnnporg0878qq7h.png)

Image-31.png (155.71 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg1OXwzYzVmZmQ1NnwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:45 上传

获取 `RegisterTask` 函数地址

随后调用 `RegisterTask` 函数，创建计划任务。具体方法是创建并配置一个 RPC 绑定句柄，利用 `NdrClientCall3` 函数进行客户端 RPC 调用来创建计划任务，其中 XML 数据是计划任务的配置信息。

![](https://attach.52pojie.cn/forum/202409/29/144540g9bb3ea2agpopaab.png)

Image-32.png (305.99 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg2MHwzN2ZmOTViMnwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:45 上传

创建计划任务

计划任务配置信息如图所示，它将实现登录、创建和修改功能，并且每分钟都会自动触发启动。此外，该任务的工作目录被设置为 `C:\Users` 目录。

![](https://attach.52pojie.cn/forum/202409/29/144551hejw5z22r31kjaoa.png)

Image-33.png (169.08 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTg2MXxhMWU3N2I2ZHwxNzkzNTc2ODE0fDE2ODA4NnwxOTY4Njc0&nothumb=yes)

2024-9-29 14:45 上传

计划任务配置信息

`PACqpC.exe`

`PACqpC.exe` 文件为白文件，会加载 `hccutils.dll`，该文件为 VMP 壳。

![](https://attach.52pojie.cn/forum/202409/29/144604cawacdnjfsfbm8yc.png)

Image-34.png (47.89 KB, 下载次数: 0)
