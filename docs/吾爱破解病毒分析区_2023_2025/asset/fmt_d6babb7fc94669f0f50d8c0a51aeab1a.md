
# [PC样本分析] 毒鼠 后门病毒再升级 通过伪造官网传播

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1845790
- 抓取时间: 2026-11-01T23:49:02

近期，火绒威胁情报系统监测到一批盗版软件安装包正在通过伪造的官网进行传播，其中包含后门病毒，该病毒被激活后，黑客可以执行截取屏幕图像、远程控制等恶意行为。目前，火绒安全产品可对上述病毒进行拦截查杀，请用户及时更新病毒库以进行防御。

![](https://attach.52pojie.cn/forum/202310/18/132817gehlvpweo53vx7ez.png)

Image-0.png (114.44 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyMXw0OGRhNWM3OHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyMXw0OGRhNWM3OHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:28 上传

火绒查杀图

此次披露的后门病毒为 [//www.huorong.cn/info/16897574401040.html" target="_blank"](https://www.huorong.cn/info/16897574401040.html:rel="nofollow noopener" href=")的升级版本。火绒工程师通过对该批样本进行溯源分析和公开资料的整合对比，发现该病毒背后基于 "银狐" （去中心化的黑产工具）的变种趋势，并与 "谷堕大盗"、"游蛇"、"Xidu" 等黑产组织的远控工具有关联。此外，鉴于该团伙的大量攻击行为和持续活跃状态，火绒将其命名为 "毒鼠"，以便后续追踪分析。

![](https://attach.52pojie.cn/forum/202310/18/132844bcfcbki3nzbsob85.png)

Image-1.png (130.18 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyMnw3YTZjYzcxYXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyMnw3YTZjYzcxYXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:28 上传

溯源图示

该黑客团伙将伪造的网站投放到搜索引擎中，并通过购买搜索引擎排名提高曝光度。其通过复制官网的界面，诱导用户下载盗版应用，以 Telegram 中文版为例：

![](https://attach.52pojie.cn/forum/202310/18/132901oz3o29reipozercz.png)

Image-2.png (81.97 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyM3xhNGRiMTQ2M3wxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyM3xhNGRiMTQ2M3wxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:29 上传

伪造的官网

当用户点击安装后，该病毒会释放正常安装程序和病毒文件，随后黑客可执行包括截取受害者屏幕图像、远程控制受害者电脑等各种恶意操作，该病毒执行流程如下所示：

![](https://attach.52pojie.cn/forum/202310/18/132920k0jwja5baur309ka.png)

Image-3.png (398.91 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyNHwwYTAzNTFlOHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyNHwwYTAzNTFlOHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:29 上传

病毒的执行流程图

目前，火绒收集到的盗版软件列表如下图所示：

![](https://attach.52pojie.cn/forum/202310/18/132937zrpudwbj3laa4lib.png)

Image-4.png (28.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyNXwyMjhiOTIyMXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyNXwyMjhiOTIyMXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:29 上传

盗版软件列表

## 一、样本分析

以 `x64-china.msi` 程序为例，其为伪造的 `Telegram` 程序安装包，在正常安装 TG 后还会释放并执行恶意样本的主体 `显卡安全升级y.exe`

![](https://attach.52pojie.cn/forum/202310/18/132950xtuxo2xs2orxohxe.png)

Image-5.png (93.93 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyNnxjNWE2ZDA1NHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyNnxjNWE2ZDA1NHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:29 上传

执行过程

该 EXE 经过代码和控制流混淆处理，参杂大量冗余代码，试图干扰分析：

![](https://attach.52pojie.cn/forum/202310/18/133015prw9t2wrwirqoqqx.png)

Image-6.png (65.06 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyN3xkMGM2M2MyMnwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyN3xkMGM2M2MyMnwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:30 上传

代码混淆

样本首先通过连接硬编码写入的 `url` 获取第一个数据文件 `DU_2.dat`，将其写入到 `C:\Users\Public\Music` 目录下创建的随机文件名中，并为其设置系统保护和隐藏属性。

![](https://attach.52pojie.cn/forum/202310/18/133035uwffd7l5l9fd43i6.png)

Image-7.png (34.69 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAyOHw5MDViMDY0MHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAyOHw5MDViMDY0MHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:30 上传

下载链接及文件

该文件以 base64 加密的形式存储着后续操作 payload 所在的 `url` 列表，该列表以一个统一的主体文件 `zip` 包和不同 `shellcode` 文件 `zip` 包所在的 `url` 链接作为一个分组，末尾部分还伴随着链接中存储的压缩包的文件名、压缩包内文件夹名以及 `shellcode` 将要存放的目录。

![](https://attach.52pojie.cn/forum/202310/18/133048f8cch4clhlb8ldfd.png)

Image-8.png (274.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAzMHw2NWQ4NWE4ZHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAzMHw2NWQ4NWE4ZHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:30 上传

加密 url 列表

### 第一阶段：（主体压缩包及其关联操作）

样本随后会解密并遍历 `DU_2.dat` 中的链接，首先下载统一的主体文件 `zip` 包 `bai12.zip`，并以随机文件名的形式写入到 `C:\Users\Public` 目录下，解压密码为 `lalala123%`

![](https://attach.52pojie.cn/forum/202310/18/133109c5l12q2xjbt11l12.png)

Image-9.png (273.04 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAzMXwzZjkzOTBkZnwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAzMXwzZjkzOTBkZnwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:31 上传

压缩包下载

该压缩包中有 4 个文件夹，每个文件夹名对应着不同的功能，`package` 下是后续执行主体，`static` 下是静态的文本信息，`txtCode` 下的文本形式的 `shellcode`，`winzipper` 下的是解压缩包用的正常程序。

![](https://attach.52pojie.cn/forum/202310/18/133127v173b1khclk7z7b7.png)

Image-10.png (19.96 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAzM3wyZDg3YmU5NXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAzM3wyZDg3YmU5NXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:31 上传

文件分析

在后续的操作中，样本会陆续解压出内部文件夹到指定目录，解压细节如下：

#### （1）
把 `package` 包下的 `NXYBankAssist.trg`、`captcommBase.dll` 解压到 `C:\Users\Public\Videos\fqcvhj`（随机文件夹名）中，并在后续的操作中重命名为 `fjqggg.exe`（随机文件名）

![](https://attach.52pojie.cn/forum/202310/18/140453h25zz1t6ybzyvp5p.png)

Image-11.png (13.19 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDA2N3w3MGZiNDUwOXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDA2N3w3MGZiNDUwOXwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 14:04 上传

package文件夹

#### （2）
把 `static` 下的 `png.1413131` 解压到 `C:\ProgramData\` 目录中，并在后续操作中重命名为 `SHELL.txt`

![](https://attach.52pojie.cn/forum/202310/18/133157snpk1j88zw097dpz.png)

Image-12.png (23.35 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAzNXwwODY4ODYzNHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAzNXwwODY4ODYzNHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:31 上传

static文件夹

#### （3）
把 `winzipper` 下的 `fdafvdav.fdafda` 解压到 `C:\Users\ \AppData\Roaming\gcdks`（随机文件夹名）中，并在后续操作中重命名为 `qqvw.exe`（随机文件名）

![](https://attach.52pojie.cn/forum/202310/18/133210kaf66ootqoinn7i7.png)

Image-13.png (5.4 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAzNnxhMjg3MmNmNHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAzNnxhMjg3MmNmNHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:32 上传

winzipper文件夹

#### （4）
把 `txtCode` 下的 `out.bin` 读取到内存中解密后执行，解密密钥为 `KPT[^JFXWPQ` 解密算法如下：

![](https://attach.52pojie.cn/forum/202310/18/133227dbpnpnbtkdcbfcbu.png)

Image-14.png (109.71 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MDAzN3w2ODQ1MWEwNHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDAzN3w2ODQ1MWEwNHwxNzkzNTc2OTM5fDE2ODA4NnwxODQ1Nzkw&nothumb=yes)

2023-10-18 13:32 上传

txtCode文件夹

其为一个 `DLL`，有两个导出函数 `TestFibo` 和 `Fibo`，其中 `TestFibo` 是空函数，`Fibo` 则用以实现后续的相互 `lnk` 文件和 `url` 文件的关联设置和 `fjqggg.exe`（原 `NXYBankAssist.trg`）的进程启动操作：

![](https://attach.52pojie.cn/forum/202310/18/133239yqfwg15ffffg1sgq.png)

Image-15.png (16.98 KB, 下载次数: 0)
