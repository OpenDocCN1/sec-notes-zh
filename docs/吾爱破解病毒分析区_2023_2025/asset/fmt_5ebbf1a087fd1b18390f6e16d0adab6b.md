
# [PC样本分析] 警惕，后门病毒在chm/vbe内嵌入`GadgetToJScript`生成的恶意js/vbs下载白加黑远控木马

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1852696
- 抓取时间: 2026-11-01T23:48:53

## 一、背景

近期捕获到大量使用 chm/vbe 的后门病毒下载器广泛传播。

根据现有情报，该后门病毒最初使用 chm 和 exe 传播，目前捕获到最早的 chm 样本可能于 2023 年 9 月份出现，且在 2023 年 10 月下旬开始出现使用 vbe 传播的变种，目前存在 chm/vbe 同时传播的现象。该后门病毒异常活跃，更新频率频繁，仅在几个月间变化多次，国内多家安全厂商发布预警——

火绒安全：《[https://huorong.cn/safe/16981185211083.html](https://huorong.cn/safe/16981185211083.html)》

腾讯电脑管家：《[https://mp.weixin.qq.com/s/PqZfqtvZoeyOSqqnc_QFUA](https://mp.weixin.qq.com/s/PqZfqtvZoeyOSqqnc_QFUA)》

本文可以作为相关安全厂商报告的说明、补充与更正，将对第一层下载器样本所使用的开源工具进行更明确的确定，并将对该家族样本的变化始终贯穿其中。

## 二、样本分析

### I. 第一层下载器

#### （一）概况

传播的 chm/vbe 即第一层下载器

分析方式：

- chm：使用 7-Zip 等工具解压缩后得到嵌入了恶意 js 的 html 文件对其进行分析
- vbe：即编译后的 vbs 代码，可以使用开源工具进行解密得到 vbs 代码对其进行分析

以下将分别呈现得到的第一层下载器的 js、vbs（10月27日发现样本）、vbs（11月1日发现样本）代码：

![](https://attach.52pojie.cn/forum/202311/04/172310cjh6vcpkdq363ook.png)

[捕获.PNG (42.05 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYxN3w2Mzg5MDRlNXwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:23 上传

![](https://attach.52pojie.cn/forum/202311/04/172320jitypivv25twx32r.png)

[捕获3.PNG (35.73 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYxOHw4ZDIwNTIzMXwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:23 上传

![](https://attach.52pojie.cn/forum/202311/04/172334jvnjjta3hmmkazmh.png)

[捕获2.PNG (14.28 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYxOXxiM2I0Njc5Y3wxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:23 上传

![](https://attach.52pojie.cn/forum/202311/04/172354wdnmzpwjjcydjdxg.png)

[捕获1.PNG (33.76 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyMHwwYWRmODI3OHwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:23 上传

此处之所以分别呈现10月27日与11月1日发现样本的 vbs 代码，是因为不难发现第一层下载器的一处微不足道的更新之处：样本的 Base64 转换函数名发生了变化——由字符串 `Base64ToStream` 变化为 `Base64sTsosSstsrsesasms`，可能是为了绕过防病毒软件对其的静态检测特征。

我们可以发现，第一层样本的 `stage` 变量内容均为 Base64 字符串，并由样本的 Base64 转换函数进行处理。

### （二）第一层下载器使用的开源工具

在火绒安全的分析报告中提到：“其为开源工具 `DotNetToJScript` 的修改版本，用于从内存加载 .NET 程序集”，但此处需要说明的是准确而言第一层样本实际上使用的是基于开源工具 `DotNetToJScript` 的又一开源工具——`GadgetToJScript`。

因此，第一层下载器使用的是开源工具 `GadgetToJScript` 生成恶意 js/vbs，用于直接在内存当中加载 .NET 程序集。

以下进行详细说明：

首先，对 `stage_1` 变量内容进行 Base64 解码，可得到一个 XML 文件，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172420cjkmbnwmghvqkqbb.png)

[捕获4.PNG (90.41 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyMXw4OTQ3Y2VkZHwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:24 上传

这是用来绕过 .Net 4.8+ 阻止 `Assembly.Load` 的 XML 代码，也就是说 `stage_1` 的作用就是用来绕过 .Net 4.8+ 阻止 `Assembly.Load`，而这与 `GadgetToJScript` 的特性一致，使用 `DotNetToJScript` 生成的脚本并无该特征。相关介绍，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172431bprf0m0ui66fhuzf.png)

[捕获5.PNG (41.63 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyMnw4YmQ0ODAwYXwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:24 上传

其次，脚本内容存在 `GadgetToJScript` 独特的反序列化执行调用链。以生成 vbs 为例，分别使用 `DotNetToJScript` 与 `GadgetToJScript` 生成 vbs，并与样本使用的 vbs 代码进行对比，其反序列化执行调用链与 `GadgetToJScript` 并无太大差异，几乎无明显变化。

`DotNetToJScript` 生成的 vbs 代码，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172450kn4rmoo4tn4ttm0k.png)

[捕获6.PNG (68.26 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyM3w1OWE0MTEwZHwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:24 上传

`GadgetToJScript` 生成的 vbs 代码，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172500gadiaizabzeokhik.png)

[捕获7.PNG (66.36 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyNHxjMDUwOTJlNHwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:25 上传

通过对比可以发现，第一层下载器的 vbs 代码与 `GadgetToJScript` 生成的 vbs 代码在各方面均高度相似。

### II. 第二层下载器（`stage_2`、`stage_3`）

通过对 `stage_2`、`stage_3` 进行 Base64 解码后可得到第二层下载器样本。

继上文 I，在第二层样本的 pdb 路径内也可发现 `GadgetToJScript` 字符串，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172518mulriblkrqtyq7hu.png)

[捕获8.PNG (25.63 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyNXxiNDQxMDcxZnwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:25 上传

![](https://attach.52pojie.cn/forum/202311/04/172531z5yegh2hubbzor9w.png)

[捕获43.PNG (26.1 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyNnw1NWRjNDliNXwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:25 上传

上文中提到，目前捕获到最早的 chm 样本可能于 2023 年 9 月份出现。因此此处分别查看了 2023 年 9 月与 2023 年 10/11 月的 `stage_2` 第二层下载器的 pdb 路径，可以发现明显不断更新的痕迹——字符串从 `v4.1.1` 变化为 `v4.2.0`、新增目录 `CHM`（猜测可能与 `VBE` 相对）、由最初的 `TestAssembly.pdb` 变化为 `strage_1_dotnet40.pdb`（`stage_3` 的 pdb 路径则是 `strage_1_dotnet35.pdb`）。且与腾讯电脑管家的分析相一致，确实发现了 `Tianma` 字符串。

第二层下载器使用 .NET 编写，进行分析后可以发现下载“白加黑”的主要逻辑均在 `Program` 类的 `Program` 函数下，且多使用云服务器厂商对象存储作为服务器来下载。并且对比多个不同时间样本的第二层下载器，可以发现其在不断更新，不断切换、更换对象存储服务器及文件下载地址。

2023 年 9 月的第二层下载器，仍是 `TestAssembly.dll`，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172620dwylk9k42fz2krkf.png)

[捕获9.PNG (106.95 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyN3wzZDk0NzkxMnwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:26 上传

2023 年 10 月下旬的 `stage_2` 第二层下载器，变化为了 `strage_1_dotnet40.dll`；而 `stage_3` 第二层下载器则为 `strage_1_dotnet35.dll`，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172637exo8h488y4jch48r.png)

[捕获10.PNG (129.37 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyOHxhZjU3ZmYyOHwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:26 上传

![](https://attach.52pojie.cn/forum/202311/04/172659myp0r03g56zipiy1.png)

[捕获44.PNG (108.08 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzYyOXxjZmMwODgxNHwxNzkzNTc2OTMwfDE2ODA4NnwxODUyNjk2&nothumb=yes)

2023-11-4 17:26 上传

2023 年 11 月上旬的第二层下载器，更换了“白加黑”结构与分离式 ShellCode 的文件名，由原先的 `libcef` 更换为了 `Foolish`，如下图所示：

![](https://attach.52pojie.cn/forum/202311/04/172721lxkbauppt3ysmtt5.png)

捕获11.PNG (130.09 KB, 下载次数: 2)
