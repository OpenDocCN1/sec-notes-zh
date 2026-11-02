
# [PC样本分析] 新型后门病毒伪装常用软件，正通过Google搜索引擎传播

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1850519
- 抓取时间: 2026-11-01T23:48:55

近期，火绒威胁情报系统检测到一种新型后门病毒伪装成常用软件，通过Google搜索引擎传播，主要针对中文用户。该病毒利用多种方式对抗杀软查杀，被运行后，黑客会立即控制受害者终端并进行任意恶意行为。目前，火绒安全产品可对上述病毒进行拦截查杀，请用户及时更新病毒库以进行防御。

![](https://attach.52pojie.cn/forum/202310/30/184538sbzm9400mdonvt8u.png)
Image-0.png (34.75 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY1MjY2MXw2MTczMWM4OXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2MXw2MTczMWM4OXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:45 上传
查杀图

以假冒"QQ音乐"为例，在Google中搜索“QQ音乐”，搜索结果的第一名就是该病毒，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184552oa3x04qxpb40x3ux.png)
Image-1.png (256.66 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY1MjY2MnwyMmIxYmFhMHwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2MnwyMmIxYmFhMHwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:45 上传
搜索引擎结果

该网站内容与QQ音乐原始官方网站高度相似，诱导用户下载，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184609hii0deevecviikr7.png)
Image-2.png (689.7 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY1MjY2M3xmYTc1MGZjN3wxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2M3xmYTc1MGZjN3wxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:46 上传
病毒伪装的QQ音乐官网

用户运行通过Google搜索引擎下载的伪装安装包之后，程序就会释放病毒文件，随后执行黑客下发的恶意行为，包括文件监控、远程控制、键盘记录、窃取QQ微信隐私信息等。该病毒执行流程，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184621xqfckkcricqcttcf.png)
Image-3.png (450.93 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY1MjY2NHwxNjg3NWExMnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2NHwxNjg3NWExMnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:46 上传
病毒执行流程

火绒工程师对该病毒进行详细分析后，发现该病毒是《后门病毒伪装成正常文件诱导点击 请保持警惕》报告中所提到的病毒的最新变种。该黑客团伙早期主要利用实时热点在国内的微信群聊中进行病毒的传播，现又以购买谷歌引擎传播病毒，预计后续还会持续更新传播手段。因此，火绒工程师建议大家尽量到官网下载软件，做到先使用安全软件扫描后再使用。

## 一、样本分析

黑客所伪造的安装包命名为`QQMusic_Music.msi`。当执行该安装包后，真正的QQ音乐安装包会被释放到`C:\Program Files (x86)\qqmusic\qqmusic\`路径中，同时在桌面生成一个快捷方式。此外，病毒相关的文件会被释放到路径`C:\Windows\SysWOW64\dazui2m`（随机文件夹名）并运行。火绒剑能够监控到的行为，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184634nckkenf5hhuhkeea.png)
Image-4.png (206.72 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY1MjY2NXxjNjY3OThkOXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2NXxjNjY3OThkOXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:46 上传
火绒剑监控到的行为图

该病毒采用了"白加黑"的技术策略，以规避安全软件的检测和清除，其隐蔽性表现突出。其中`libcef.dll`是经过黑客修改的病毒文件，而`webprocess.exe`则是一个带有正规签名“白”文件。相关的签名信息，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184646asz3hjjzw3o2j2ht.png)
Image-5.png (53 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY1MjY2NnxhODgwNTEzNnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2NnxhODgwNTEzNnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:46 上传
带签名的“白”文件

`libcef.dll`是被黑客修改过的 DLL 文件，其主要功能是对内层的后门模块进行解密和加载。为了对抗杀毒软件查杀，该 DLL 文件采用了混淆技术。它使用了 C++ 异常以打乱正常程序的执行顺序，这也有效地干扰了安全分析人员的分析工作。这种混淆方法能够阻止 IDA 插件的 "反编译"功能，混淆后的执行流程，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184700wzrcipw0oj7y7iff.png)
Image-6.png (118.42 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY1MjY2N3xlN2VhNzc2MXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2N3xlN2VhNzc2MXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:47 上传
执行流程混淆

内层的后门模块采用了异或加密进行混淆。相关的解密代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184714o4k74rss7j4j5nrj.png)
Image-7.png (36.94 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY1MjY2OHxiYzdjMjE1M3wxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2OHxiYzdjMjE1M3wxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:47 上传
解密后门模块

后门模块解密后将直接加载到内存中执行，而不会在磁盘上留下任何痕迹。这种方式使得其执行过程难以被传统的磁盘基础防御机制检测到，以此提高了其隐蔽性，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184726zkefnxksi8sf16c8.png)
Image-8.png (54.72 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY1MjY2OXxkZjFhMmI0YnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY2OXxkZjFhMmI0YnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:47 上传
内存加载恶意模块

后门模块被加载后会主动连接C&C服务器（`154.91.65.25`），接收并执行C&C服务器下发的恶意指令，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184738jw9wculo3c7boj01.png)
Image-9.png (98.11 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY1MjY3MHw2ODYzYWQ1ZHwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY3MHw2ODYzYWQ1ZHwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:47 上传
连接C&C服务器

该病毒还会将自身添加到任务计划中，以实施持久化策略，从而达到在受害者电脑上持续运行的目的，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184750fp101fyqfu0afaaf.png)
Image-10.png (130.93 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY1MjY3MXw3ZDRmNDlkN3wxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY3MXw3ZDRmNDlkN3wxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:47 上传
添加任务计划

添加完任务计划后，通过系统自带的任务计划程序来查看，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184804c0upunghlc1t20u0.png)
Image-11.png (42.45 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY1MjY3MnwzNGU2Yzk2MXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY3MnwzNGU2Yzk2MXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:48 上传
被添加的任务计划

相较之前，最新变种增加了一些新的功能，例如，窃取微信数据库解密密钥、窃取QQ隐私信息、窃取Chrome浏览器数据等恶意功能。利用微信数据库密钥，攻击者能够解密受害者计算机上的微信数据库，从而获取受害者与其他人的聊天记录中的全部内容。通过这些信息，攻击者可以更有效地执行诈骗操作。获取微信数据库解密key并上传至C&C服务器，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184912u91amj9yh1ccadjs.png)
Image-12.png (62.69 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY1MjY3M3w1NmMwOTkyOXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY3M3w1NmMwOTkyOXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:49 上传
获取微信数据库key

该变种还具有通过利用 QQ 快捷登录协议来获取本机登录的 QQ 相关信息的能力。这些信息包括但不限于 QQ 好友备注以及好友QQ 号、用户所加入的 QQ 群组等敏感信息。这些数据代表了用户的私人和社交生活的关键方面，其泄露可能会导致严重的隐私侵犯。与微信数据库解密密钥一样，攻击者可以利用这些信息来进行更有效的欺诈行为。获取qq隐私信息，并上传至C&C服务器，相关的代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184927mxxoh0hxlhxrsaes.png)
Image-13.png (433.24 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY1MjY3NHxmN2M4Zjk4NnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY3NHxmN2M4Zjk4NnwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:49 上传
获取QQ隐私信息并上传至C&C服务器

通过对病毒代码的分析，我们可以推测该病毒具有窃取Chrome浏览器数据的恶意模块。在安全人员的调试与分析过程中，尚未捕获到这个恶意模块，但不能排除病毒在后续会下发这个恶意模块的可能性。相关的调用代码，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/184939vtyuu79jauekk4k7.png)
Image-14.png (78.11 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY1MjY3NXwxZGQ4N2FkNXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjY3NXwxZGQ4N2FkNXwxNzkzNTc2OTMyfDE2ODA4NnwxODUwNTE5&nothumb=yes)
2023-10-30 18:49 上传
窃取Chrome浏览器数据

该后门模块具备各种恶意功能如：键盘记录、文件窃取、远程控制等恶意功能，相关功能相比之前变种，变化不大，通过对代码进行比较，我们可以确定该病毒是《后门病毒伪装成正常文件诱导点击，请保持警惕》中提到的病毒的最新变种。相关的功能代码对比，如下图所示：

![](https://attach.52pojie.cn/forum/202310/30/185000zba222p1hkobllgl.png)
Image-15.png (424.71 KB, 下载次数: 2)
