
# [PC样本分析] 钓鱼网页散播银狐木马，远控后门威胁终端安全

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1988321
- 抓取时间：2026-11-01T23:46:28

当今网络环境下，许多人都有通过搜索引擎下载应用程序的习惯，虽然这种方式简单又迅速，但这也可能被不法分子所利用，通过设置钓鱼网站来欺骗用户。这些钓鱼网站可能会通过各种方式吸引用户点击，从而进行病毒的传播，危害个人或企业的信息安全。

我们期望本篇文章有助于帮助您提高网络安全防范意识，通过官方正规的渠道下载软件。

![Image-2.png](https://attach.52pojie.cn/forum/202412/04/185102hojbdyyz00bgpgld.png)

`Image-2.png` (318.25 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMyOXwxMTEzM2JlNXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:51 上传

近期，火绒威胁情报中心监测到一批更加活跃的“银狐”系列变种木马，该木马就是主要通过钓鱼网页进行传播。火绒安全产品能够有效识别通过网页下载的钓鱼恶意样本，保障用户系统的安全，请火绒的广大用户及时更新病毒库以提高防御能力。

![Image-0.png](https://attach.52pojie.cn/forum/202412/04/184624ywffvsfbjcivevdc.png)

`Image-0.png` (26.1 KB, 下载次数: 2) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMyN3w3YmJlNzczOXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:46 上传

**查杀图**

火绒工程师对样本的分析过程中，发现恶意攻击者会将钓鱼网页进行合法备案，伪装成正规网站，并借助搜索引擎的广告推广机制提高搜索结果排名，由于合法备案增加了网站的可信度，导致用户很容易对恶意网站产生信任。并且，恶意网页可通过支付推广费用获得较高的曝光率和排名，从而诱骗用户点击钓鱼网页，下载并运行病毒样本。

![Image-3.png](https://attach.52pojie.cn/forum/202412/04/203631kk7r2qe0r27y793i.png)

`Image-3.png` (26.64 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTQwMXw5OTBkN2ViY3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 20:36 上传

![Image-4.png](https://attach.52pojie.cn/forum/202412/04/203633wij4mmz6mqm9liji.png)

`Image-4.png` (25.44 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTQwMnw3Yzk1YzRkOHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 20:36 上传

该样本利用系统特性实现持久化，并通过“白加黑”技术释放恶意文件，从而部署 WinOs 远控后门。

样本执行流程图如下：

![Image-1-1000.png](https://attach.52pojie.cn/forum/202412/04/184937euh23cceei2enfhc.png)

`Image-1-1000.png` (158.35 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMyOHxiOTA5NmEyZXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:49 上传

**流程图**

## 一、样本分析

### 样本信息

该样本是一个恶意的 MSI 安装程序，内含 WinOs 后门病毒，其恶意代码通过加密方式嵌入 PNG 图片，并利用隐写技术隐藏在图片文件中。执行后，恶意代码会被解压并释放到公共文档之中，利用 COM 组件创建快捷方式，实现全白进程链，之后释放 Registry Workshop（注册表管理工具），利用 `.reg` 配置文件实现后门持久化，进而监控用户电脑的活动，在用户无感情况下，持续获取信息并执行恶意操作。

### 样本执行

#### 一阶段：loader释放病毒载体

初始样本利用 MSI 安装程序来释放包含病毒的加密压缩包和正常安装包。利用 `msiexec` 的 `CustomAction` 特性，加载并调用恶意 DLL 导出函数，从而解压并执行被隐藏的恶意文件：

```text
C:\Users\Public\Documents\down.exe
C:\Users\Public\Documents\libcef.dll
C:\Users\Public\Documents\aut.png
C:\Users\Public\Documents\view.png
```

首先，对初始样本 `demo.exe` 进行深入分析，样本是一个 MSI 安装程序，通过 `Orca` 工具查看 MSI 结构，发现其 `CustomAction` 表中存在一个名为 `CallFunctionFromDLL` 的执行动作，该动作指示安装程序将加载名为 `ziplib.dll` 的动态链接库，并调用 `zipcom` 导出函数。

![Image-8.png](https://attach.52pojie.cn/forum/202412/04/185212iqd1cuj876b8u657.png)

`Image-8.png` (70.6 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzMHxhZmJjYzM0Y3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:52 上传

在对该 MSI 安装程序进行解包后，发现以下几个文件：

- 正常安装包：包含 ToDesk 和好压压缩软件的正常安装包。
- 恶意 DLL 文件：`ziplib.dll`
- 加密病毒文件：`all.zip`，加密压缩包含加密的白加黑文件、图片恶意文件。

`ziplib.dll` 采用填充脏数据的方式扩大体积，将文件大小扩展至 `245MB`，防止文件被上传，并试图绕过云查杀的检测机制。在 MSI 安装过程中，恶意代码会默认将 `all.zip` 解压并释放到受害者的系统目录：`C:\Users\Public\Documents`。

![Image-9.png](https://attach.52pojie.cn/forum/202412/04/185248ugjedtigsg94njhe.png)

`Image-9.png` (50.38 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzMXwxYzZlZjFlMHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:52 上传

![Image-10.png](https://attach.52pojie.cn/forum/202412/04/185250nkq0zku90kzkuzwp.png)

`Image-10.png` (50.75 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzMnwyM2YzYjg2ZnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:52 上传

对 `ziplib.dll` 进行静态分析，发现其执行过程包含多个恶意行为和[检测](https://www.52pojie.cn/thread-661779-1-1.html)：

**1. 系统文件夹目录获取**

`ziplib.dll` 尝试获取系统文件夹目录。若获取失败，则使用 `GUID` 生成临时路径标识符，确保能够继续执行后续操作。

**2. 解压 all.zip 文件**

`ziplib.dll` 使用密码 `hello202411` 对 `C:\Users\Public\Documents\all.zip` 文件进行解压。

**3. 环境检测**

在解压后，`ziplib.dll` 会执行一系列环境检测：

- 父进程检查：检测当前进程的父进程是否是 `msiexec.exe`（即 MSI 安装程序的解释器）。
- 虚拟机检测：通过获取系统的 `CPU` 核心数，检测到的核心数是否大于 `2`。

**4. 执行恶意文件**

`ziplib.dll` 通过 `WinExec` 调用执行解压后的白加黑恶意文件，进一步发动恶意行为。

![Image-11.png](https://attach.52pojie.cn/forum/202412/04/185405p6m4zabpmomogp6a.png)

`Image-11.png` (43.94 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzM3xiMDliMmIwM3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:54 上传

![Image-12.png](https://attach.52pojie.cn/forum/202412/04/185407flor96hypk44ox36.png)

`Image-12.png` (14.68 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzNHw2YWE3NDI0YXwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:54 上传

![Image-13.png](https://attach.52pojie.cn/forum/202412/04/185409wooa4tk17a5lqfkz.png)

`Image-13.png` (65.43 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzNXxlYzEyZWY3MHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:54 上传

![Image-14.png](https://attach.52pojie.cn/forum/202412/04/185411n4tr4y524zqxq528.png)

`Image-14.png` (306.81 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzNnw3YjEyN2Q5N3wxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:54 上传

对 `all.zip` 文件进行解压后，发现病毒利用“白加黑”技术执行恶意代码。解压后的文件包括以下关键组件：

**1. `libcef.dll`（恶意 DLL 文件）**

`libcef.dll` 是一个恶意动态链接库，同样采用填充脏数据增大体积的方式，将文件大小膨胀防止木马被上传，绕过云查杀的检测。

**2. `down.exe`（白文件）**

![Image-15.png](https://attach.52pojie.cn/forum/202412/04/185516mre0emlllhlblzmh.png)

`Image-15.png` (59.61 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzN3w3YmMzM2I1MnwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:55 上传

对 `libcef.dll` 进行静态分析，发现其初始时会通过 Patch 系统 `ntdll.dll` 修正其在 `DllMain` 函数中的线程死锁问题，随后会检查启动命令行参数是否包含 `/aut`：

- 若命令行包含 `/aut`，则通过 `aut.png` 图片隐写技术，将恶意代码加载到内存中，并执行持久化操作。
- 若命令行不包含 `/aut`，则会主动添加 `/aut` 参数并创建新的进程，以此方式启动白文件 `down.exe`，并继续执行持久化操作。

在启动过程中，`libcef.dll` 还会检测目标系统是否存在杀毒软件：

- 若检测到杀软，则通过 `view.png` 图片隐写恶意代码，将后门功能加载到内存中，执行后门功能。
- 若未检测到杀软，则通过远程线程注入到 `colorcpl.exe` 系统进程，实现后门功能。

![Image-16.png](https://attach.52pojie.cn/forum/202412/04/185624hm4khh229yslee02.png)

`Image-16.png` (50.45 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MTMzOHwzY2YwODM4OHwxNzkzNTc2Nzg2fDE2ODA4NnwxOTg4MzIx&nothumb=yes) · 2024-12-4 18:56 上传

详细分析如下：

`libcef.dll` 首先通过调用 `RtlLeaveCriticalSection` 函数对 `LoaderLock` 临界区进行释放，同时通过特征码搜索的方式对系统的加锁机制进行 patch 修改，解决 `DllMain` 线程死锁问题。
