
# [PC样本分析] 广告位变窃密通道：IDM破解工具作者或成为窃密帮凶

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2081845
- 抓取时间: 2026-11-01T23:43:17

近期，火绒安全工程师监测发现一个危险陷阱：很多人想找的“IDM破解程序”，其下载网站的显眼广告位里藏着恶意下载按钮，下载运行就会中招窃密病毒！这类病毒专门盯着你的重要信息—— 例如浏览器保存的密码、Steam 游戏账号、加密货币钱包、Github 账号，甚至会偷偷替换加密货币收款地址、获取邮箱验证码，把你的财产和隐私一键偷走，最终全部上传到窃密者的服务器。

该病毒为StealC 窃密病毒，或是会安装恶意浏览器插件的窃密病毒。

![](https://attach.52pojie.cn/forum/202512/23/131823mb4is3352jiirur5.png)

广告位中病毒下载按钮与相应下载页面.png (212.45 KB, 下载次数: 1) 2025-12-23 13:18 上传

下载链接位于文章的最底部，用户难以察觉，若未查看评论区中他人的提醒，则极易中招。

![](https://attach.52pojie.cn/forum/202512/23/131910b7p0t5ttjjtmk3gi.png)

评论区提醒.png (45.5 KB, 下载次数: 1) 2025-12-23 13:19 上传

此次病毒系通过IDM 破解程序下载网站的广告位进行传播。即制作IDM 破解程序的作者将下载页面中的广告栏目予以出租，之后病毒作者通过购置该广告位，并将自身的病毒嵌入下载按钮。如此一来，受害者点击下载并执行病毒程序，从而完成窃密操作。

流程图如下：

![](https://attach.52pojie.cn/forum/202512/23/131927p0fs61skms8n64wz.png)

流程图.png (160.18 KB, 下载次数: 2) 2025-12-23 13:19 上传

目前，火绒安全产品已经能精准拦截和查杀这类恶意程序。

![](https://attach.52pojie.cn/forum/202512/23/132015i5yk22y23kkei3uk.png)

查杀图1.png (26.88 KB, 下载次数: 2) 2025-12-23 13:20 上传

![](https://attach.52pojie.cn/forum/202512/23/132027bhiuhaujc4cumnxp.png)

查杀图2.png (30.14 KB, 下载次数: 2) 2025-12-23 13:20 上传

![](https://attach.52pojie.cn/forum/202512/23/132040tuh595qc733i2tah.png)

查杀图3.png (32.22 KB, 下载次数: 2) 2025-12-23 13:20 上传

## 一、样本分析

此次分析的样本为 `File_Open v.5.2.7z` 与 `file_{pass}.rar`，分析后期发现 `file_{pass}.rar` 中的最终载荷 StealC 窃密病毒家族被用于 `File_Open v.5.2.7z` 病毒，且属于三种窃密方式之一，其三种窃密方式为类 StealC 窃密、浏览器插件窃密、StealC 窃密，且都会利用云控下发窃密配置，并通过窃密配置进行窃密。

下面主要以 `File_Open v.5.2.7z` 样本为主线进行分析，该样本总共有六个阶段，分别是白加黑并解密出 ShellCode 执行、获取 ShellCode 中的 ShellCode 数据后执行、用于规避沙箱或调试进程阶段、重复第二阶段、云控配置获取阶段、最终是窃密阶段。

### 1.1 `File_Open v.5.2.7z`

#### 1.1.1 第一阶段—解密文件

从广告位下载恶意程序到解压执行：从广告位可以下载到 `239.135.126.83.zip` 文件，解压后为 `File_Open v.5.2.7z`，其解压包中包含以下文件。其中用户会被误导点击白文件 `setup.exe` 进行安装，此时会加载黑文件 `sqlite.dll`，该 DLL 会读取 `data.bin` 文件并解密并执行其中 ShellCode。

![](https://attach.52pojie.cn/forum/202512/23/132152fxri8k1zzgbg1khz.png)

解压包文件构成.png (42.86 KB, 下载次数: 2) 2025-12-23 13:21 上传

下面是 `data.bin` 文件解密后的数据部分偏移：

![](https://attach.52pojie.cn/forum/202512/23/132218uqnfwg8qfnfcc9wc.png)

data.bin 偏移表.png (98.24 KB, 下载次数: 2) 2025-12-23 13:22 上传

调用函数并执行 ShellCode：随后通过 `STRING_MARKERS` 标记定位到 `ntdll.dll` 字符串并获取其基址，并利用特征获取到调用号，随后手动组装 `NtAllocateVirtualMemory`、`NtWriteVirtualMemory`、`NtProtectVirtualMemory` 调用方法，并分别用于申请内存、写入 ShellCode、修改内存权限，最终执行 ShellCode。

![](https://attach.52pojie.cn/forum/202512/23/132445dsh0dgdg119g3y55.png)

执行 ShellCode.png (166.69 KB, 下载次数: 2) 2025-12-23 13:24 上传

#### 1.1.2 第二阶段—获取数据

该模块中通过 PEB 的模块枚举并利用动态链接库名称与函数名称的哈希计算获取到匹配的函数地址，例如 `VirtualAlloc`、`VirtualFree`、`RtlExitUserProcess`、`LoadLibraryA` 等函数，随后通过解密算法解密偏移 `0x23C` 处的加密数据。

![](https://attach.52pojie.cn/forum/202512/23/132546eclw4mcllmedxu4i.png)

动态获取函数地址.png (199.13 KB, 下载次数: 2) 2025-12-23 13:25 上传

通过获取函数地址后修改函数开头数据的方式使函数失效，包含 `amsi.dll` 中 `AmsiScanBuffer` 和 `AmsiScanString`，以及 `ntdll.dll` 中 `EtwEventWrite` 函数。

![](https://attach.52pojie.cn/forum/202512/23/132622xdeii32mdw3edzdi.png)

使函数失效.png (61.66 KB, 下载次数: 2) 2025-12-23 13:26 上传

获取数据的方式存在三种标记，分别为 1、2、3，它们依次对应直接赋值（数据中已存在）、下载数据以及直接退出程序（即不获取数据）。在此过程中，数据将通过系统函数（`RtlDecompressBuffer`）或自行实现的解压函数进行解压处理。

![](https://attach.52pojie.cn/forum/202512/23/132653hzj9lcdajdlzahdy.png)

获取数据的方式.png (77.66 KB, 下载次数: 2) 2025-12-23 13:26 上传

其中获取到的代码有三种格式，执行时通过数字 2、4、6 控制。

- 2-.NET 加载：实现 Native CLR Host，其中利用了 `ICLRMetaHost`、`ICLRRuntimeInfo`、`ICorRuntimeHost` 等 COM 接口，从内存加载并执行托管程序集，例如 C# 程序等。
- 4-DLL 模块加载：直接手动加载并调用入口点。
- 6-脚本执行：利用 `IActiveScript` / `IActiveScriptParse` 等 COM 接口实现 VBS 等脚本执行。

![](https://attach.52pojie.cn/forum/202512/23/132756e55go5bfaif2qg45.png)

三种执行方式.png (18.64 KB, 下载次数: 2) 2025-12-23 13:27 上传

![](https://attach.52pojie.cn/forum/202512/23/132824ildwmoxk0piikjm0.png)

脚本和 C#代码加载.png (262.63 KB, 下载次数: 2) 2025-12-23 13:28 上传

#### 1.1.3 第三阶段—规避阶段

该阶段开始会检测当前目录下 `12345.txt` 文件是否存在、检测危险进程是否存在、检测虚拟显卡是否存在。

![](https://attach.52pojie.cn/forum/202512/23/133121bxzx6br5an00x5tr.png)

检测方式.png (118.14 KB, 下载次数: 2) 2025-12-23 13:31 上传

下面的字符串为样本中解密出来的进程列表，总共 67 个进程名。

- `apdagent.exe`
- `apimonitor.exe`
- `azurearcsystray.exe`
- `binaryninja.exe`
- `burpsuite.exe`
