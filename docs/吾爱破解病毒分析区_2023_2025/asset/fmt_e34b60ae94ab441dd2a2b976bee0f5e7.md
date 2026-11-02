
## 命令行参数检查与进程注入

![](https://attach.52pojie.cn/forum/202412/04/185645ghechdvv1likk03q.png)
Image-17.png (75.78 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTMzOXw5Y2Y0NDI3NnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzOXw5Y2Y0NDI3NnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:56 上传

执行命令行参数检查，对比命令行是否存在 `/aut` 参数，如果存在则通过 `fn_regedit` 函数将白文件 `down.exe` 路径写入注册表 `HKEY_CURRENT_USER\SOFTWARE\\DICKEXEPATH` 位置。

![](https://attach.52pojie.cn/forum/202412/04/185747o7nvtvnvtsdrtnj7.png)
Image-18.png (25.23 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0MHw5ZGRiNTdkM3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0MHw5ZGRiNTdkM3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:57 上传

![](https://attach.52pojie.cn/forum/202412/04/185749l5xadx68ovoxo7va.png)
Image-19.png (52.36 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0MXwxZjA3NWNkNnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0MXwxZjA3NWNkNnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:57 上传

随后通过 `fn_persistent` 函数内存加载 `aut.png` 中的恶意代码实现持久化操作。

![](https://attach.52pojie.cn/forum/202412/04/185814kpjwt2twwnzffllf.png)
Image-20.png (22.47 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0MnwzMjBkN2Y0YXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0MnwzMjBkN2Y0YXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:58 上传

`aut.png` 图片文件(辅助数据块的格式中存储PE恶意代码)。

![](https://attach.52pojie.cn/forum/202412/04/185833n2aakm3kzazmv6md.png)
Image-21.png (224.39 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0M3xmYjk3YzBjNHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0M3xmYjk3YzBjNHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:58 上传

![](https://attach.52pojie.cn/forum/202412/04/185835y74qv40qvk3zzc20.png)
Image-22.png (53.95 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0NHxmMzBkNTgwMXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0NHxmMzBkNTgwMXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:58 上传

如果命令行参数不存在 `/aut`，则调用 `fn_exec_aut` 函数增加 `/aut` 参数再次启动进程，执行权限维持操作。

![](https://attach.52pojie.cn/forum/202412/04/185853dkwuzase7awwep0u.png)
Image-23.png (43.42 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0MTM0NXw5ZjZmOGNlMnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0NXw5ZjZmOGNlMnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:58 上传

通过判断是否存在杀软路径，决定注入方式：注入自身进程或注入系统 `colorcpl.exe` 进程中。

![](https://attach.52pojie.cn/forum/202412/04/185917zyu9ye2ygkocyssz.png)
Image-24.png (19.05 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0MTM0NnxkM2RmMmU2NXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0NnxkM2RmMmU2NXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:59 上传

其中注入自身与权限维持操作是相同的，以下是线程注入系统 `colorcpl.exe` 进程过程。

![](https://attach.52pojie.cn/forum/202412/04/185940e1pvodca8pcpx81q.png)
Image-25.png (86.03 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0N3w2OGE1MTIwYXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0N3w2OGE1MTIwYXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 18:59 上传

## 二阶段：COM组件权限维持持久化

样本加载 `aut.png` 中的恶意代码，通过恶意代码利用 COM 组件创建多个快捷方式，并释放 Registry Workshop 注册表管理工具。接着，样本利用该注册表管理工具创建reg格式注册表配置文件并通过快捷方式附加参数reg配置文件路径，实现注册表权限的维持，从而确保恶意代码能够在系统中持久化存在。

对 `libcef.dll` 分析发现，该样本通过图片隐写技术，将恶意代码嵌入到 `aut.png` 图片文件中的辅助数据块区段，并隐藏了一个完整的 PE 文件。将 `aut.png` 中的恶意代码 dump 出来后发现恶意代码是一个 PE 文件，对其进行静态分析发现，恶意代码首先采用 com 组件进行创建快捷方式，随后通过修改注册表 `HKEY_CURRENT_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run` 添加注册表启动项。

![](https://attach.52pojie.cn/forum/202412/04/190008izcyf3marrs1vz1e.png)
Image-26.png (68.35 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0OHwwYzQxZDQyZHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0OHwwYzQxZDQyZHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:00 上传

恶意代码通过 COM 组件 `{00021401-0000-0000-C000-000000000046}` 创建快捷方式。该 COM GUID 是 Windows 系统中常用的 Shell COM 组件，用于操作文件夹和快捷方式。

![](https://attach.52pojie.cn/forum/202412/04/190027qsbnt2usrcnb3oor.png)
Image-27.png (78.65 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM0OXwxOWEwZmQyOHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM0OXwxOWEwZmQyOHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:00 上传

![](https://attach.52pojie.cn/forum/202412/04/190029p4doxn664jnp4on3.png)
Image-28.png (62.4 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0MTM1MHwzMWVhMTE5ZHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM1MHwzMWVhMTE5ZHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:00 上传

将内存中的 Registry Workshop 注册表管理工具写入临时目录后，恶意代码通过 com 组件创建 Registry Workshop（注册表管理工具）的快捷方式。随后，恶意代码生成一个 `.reg` 格式的注册表文件，并利用该文件通过快捷方式的形式附加参数，利用 Registry Workshop 写入启动项。

![](https://attach.52pojie.cn/forum/202412/04/190111ygaic5ngv8aa78ga.png)
Image-29.png (73.33 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0MTM1MXw2Y2NlZWQwNnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM1MXw2Y2NlZWQwNnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:01 上传

![](https://attach.52pojie.cn/forum/202412/04/190113ompumyr7u64ba6hu.png)
Image-30.png (55.21 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0MTM1MnxkODZkNGRhOHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM1MnxkODZkNGRhOHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:01 上传

![](https://attach.52pojie.cn/forum/202412/04/190115vhpz2l2ysp72sx4y.png)
Image-31.png (28.82 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM1M3xlN2EyOWVkZnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM1M3xlN2EyOWVkZnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:01 上传

通过快捷方式 `C:\Users\Admin\AppData\Local\Temp\{899066F6-659B-4985-A1B4-FE00017F3CAE}.exe /s "C:\Users\Admin\AppData\Local\Temp\{31917437-6BE4-4b67-8741-131B63BDDD0D}"` 启动 `{31917437-6BE4-4b67-8741-131B63BDDD0D}` (reg格式文件）。

![](https://attach.52pojie.cn/forum/202412/04/190138zpwiyw6ce166zis7.png)
Image-32.png (69.38 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM1NHwxZjZjOTZhY3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM1NHwxZjZjOTZhY3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:01 上传

以同样的方式添加注册表启动项 `GoogleUpdata_Service` -> `C:\Users\Public\Documents\down.exe`。

![](https://attach.52pojie.cn/forum/202412/04/190157mcypnyn4psdqcb2o.png)
Image-33.png (64.01 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MTM1NXwyYWNhM2Y1ZXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTM1NXwyYWNhM2Y1ZXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes)
2024-12-4 19:01 上传

## 三阶段：后门功能

样本利用 `WinOS` 实现远程控制，通过加载 `view.png` 中的恶意代码，向 `colorcpl.exe` 系统进程中远程线程注入病毒，从而实现后门功能。后门的功能包括：
- 插件加载：支持内存加载与进程注入。
- 屏幕捕获：获取目标系统的屏幕信息。
- 文件操作：文件上传与下载执行。
- 监控与控制：记录备注信息、分组信息，检测并过滤进程，监控屏幕状态。
- 系统管理：清除系统日志、重启系统、结束进程、修改系统配置。
- 其他功能：开启剪贴板监控、发送心跳包。
