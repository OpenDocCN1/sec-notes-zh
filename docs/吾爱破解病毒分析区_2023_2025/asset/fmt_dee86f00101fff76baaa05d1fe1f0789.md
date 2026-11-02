
# [PC样本分析] Telegram汉化暗藏玄机，悄无声息释放后门病毒

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1924008
- 抓取时间: 2026-11-01T23:48:05

近期，火绒安全实验室收到用户反馈称自己下载的 Telegram 汉化文件安装后造成系统异常，火绒安全工程师第一时间为用户提供技术支持，提取样本进行分析。分析过程中发现该程序在对 Telegram 程序进行汉化的同时还 “悄悄” 释放恶意后门文件，执行远控操作，危害极大。目前，火绒安全产品可对上述病毒进行拦截查杀，请广大用户及时更新病毒库以提高防御能力。

![](https://attach.52pojie.cn/forum/202405/14/131921r7fx417mtx5n54gx.png)
Image-0.png (31.34 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTcyN3w5MmZhZWFjMHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTcyN3w5MmZhZWFjMHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
火绒 6.0 查杀图

样本执行流程图如下所示：

![](https://attach.52pojie.cn/forum/202405/14/131926t00ozyerro00ae0t.png)
Image-1.png (342.12 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTcyOHwwM2I2Mzk5N3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTcyOHwwM2I2Mzk5N3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
执行流程图

在此，火绒工程师建议大家在下载软件时，尽量选择官方网站或正规可信的应用商店，同时安装可靠的安全软件保护设备免受恶意软件和病毒的侵害。目前，火绒 6.0 已上线公测，在沿袭核心功能的基础上，专注终端安全，精细化查杀，多重技术深入布局，安全防护再次升级；欢迎大家前往火绒官方网站下载体验。

## 一、样本分析

起始汉化安装包是一个捆绑了恶意文件的 `exe` 程序，附带 `vmp` 壳干扰分析：

![](https://attach.52pojie.cn/forum/202405/14/131929bs0eggq9xxjgzhdq.png)
Image-2.png (45.82 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTcyOXwxMDI4MjJjM3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTcyOXwxMDI4MjJjM3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
vmp 加壳

双击运行程序时，其会先执行正常的 telegram 汉化操作，链接到正常 telegram 程序并改变界面语言：

![](https://attach.52pojie.cn/forum/202405/14/131931j66h83s9jssvp32f.png)
Image-3.png (211.5 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczMHwzMGVjMmI0ZnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczMHwzMGVjMmI0ZnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
链接到 telegram

![](https://attach.52pojie.cn/forum/202405/14/131933iwxqbqwwjvbehbbe.png)
Image-4.png (50.58 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczMXwzYWM1ZTkyNHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczMXwzYWM1ZTkyNHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
汉化

但在执行正常操作的同时，会在内存中解密出第一阶段 `payload`，其为一个恶意 `dll`，用于在内存中加载执行：

![](https://attach.52pojie.cn/forum/202405/14/131935jujv8tovoj8tv8o8.png)
Image-5.png (122.13 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczMnxlYjA5ZGYyYXwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczMnxlYjA5ZGYyYXwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
内存解密dll

### Payload1：加载通用.dll

从内存中 dump 出 `dll` 进行分析，其名称为 "`加载通用.dll`"。有一个 `Hello` 的导出函数，但不执行任何操作，所有操作都在 `DllMain` 中进行：

![](https://attach.52pojie.cn/forum/202405/14/131937uxsk3eskpks3kxek.png)
Image-6.png (22.82 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczM3w1NzkwODYxMnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczM3w1NzkwODYxMnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
dll 概览

该 `dll` 以易语言编写并由黑月编译器编译，特征字符串如下：

![](https://attach.52pojie.cn/forum/202405/14/131939s1pv36kow31v2r11.png)
Image-7.png (44.77 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTczNHw3MzBjNDRlMnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczNHw3MzBjNDRlMnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
黑月编译器特征字符串

在执行过程中会先进行一些检测操作，通过判断 `SxIn.dll` 是否存在来检测 360 的虚拟沙盒。检查 `SystemTray_Main` 的窗口属性是为了避免重复操作，因为在后面的操作中会有将该窗口属性设置为隐藏的代码：

![](https://attach.52pojie.cn/forum/202405/14/131942kdjjbgdqqqjc9j9j.png)
Image-8.png (17.46 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTczNXxmZGZhNTQ2M3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczNXxmZGZhNTQ2M3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
检测代码

随后开始初始URL部分，计算文件自身 `MD5` 并获取前 16 字节部分作为URL路径进行拼接，并生成 `FPTOKEN` 用于后续 http 通讯：

![](https://attach.52pojie.cn/forum/202405/14/131944opdxl2slsf1nnlvv.png)
Image-9.png (74.54 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczNnxkYWJiNDcxYXwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczNnxkYWJiNDcxYXwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
拼接 URL

通信相关的函数是通过 `WinHTTP` 的 `COM` 对象接口来调的，数据传输时使用的域名是 `taobc.tv`，利用前面生成的 URL 路径及 `FPTOKEN` 以验证请求的有效性：

![](https://attach.52pojie.cn/forum/202405/14/131946ta0gtqqujqgw9d0q.png)
Image-10.png (70.23 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczN3xiYzM4ZTQxNHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczN3xiYzM4ZTQxNHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
外联操作

接收到的数据为加密数据，如下图所示：

![](https://attach.52pojie.cn/forum/202405/14/131948vdl918r8gkp7kl41.png)
Image-11.png (113.63 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczOHw5ZjI5Zjg2ZnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczOHw5ZjI5Zjg2ZnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
通信流量截图

在接收到回传的通信数据后，`dll` 会获取多个系统特定位置路径，作为后续投放第二阶段 `Payload` 使用：

![](https://attach.52pojie.cn/forum/202405/14/131950w8zjypieekpgga8k.png)
Image-12.png (46.93 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTczOXxlOTJiOTgzNnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTczOXxlOTJiOTgzNnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
获取系统关键路径

随后通过 `ResponseBody` 字段来提取加密部分数据，并解密出新的 `PE` 文件：

![](https://attach.52pojie.cn/forum/202405/14/131952x0ywzjcg0ozso7ng.png)
Image-13.png (58.53 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTc0MHxmYTlhYTg1M3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTc0MHxmYTlhYTg1M3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
调用解密函数

![](https://attach.52pojie.cn/forum/202405/14/131955zvhvdl7vbt2bdgdd.png)
Image-14.png (55.24 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTc0MXwzN2M1MDEwM3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTc0MXwzN2M1MDEwM3wxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
解密数据截图

第二阶段 `payload` 下发的路径选在 `Videos` 目录，`dll` 会创建 `VSTELEM` 文件夹，并继续创建随机文件夹用以投放第二阶段载荷：

![](https://attach.52pojie.cn/forum/202405/14/131957yqv3wdwmirq1qms4.png)
Image-15.png (32.84 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTc0MnwwODkxNDcwNHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTc0MnwwODkxNDcwNHwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
payload 投放位置

通过在循环中读取并定位解密数据中的 `PE` 文件，陆续投放 `rzrue.exe`（随机文件名，与文件夹同名）、`Language.dll`、`update.dll` 到前面指定的目录中：

![](https://attach.52pojie.cn/forum/202405/14/131959b4srs4lhuis4v4zs.png)
Image-16.png (59.78 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTc0M3w1Y2FiOWUwYnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTc0M3w1Y2FiOWUwYnwxNzkzNTc2ODgyfDE2ODA4NnwxOTI0MDA4&nothumb=yes)
2024-5-14 13:19 上传
释放文件代码

![](https://attach.52pojie.cn/forum/202405/14/132001x5nwh5hpmnmwotqq.png)
Image-17.png (17.67 KB, 下载次数: 0)
