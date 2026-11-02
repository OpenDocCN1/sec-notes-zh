
# [PC样本分析] BT索引站藏陷阱！私人矿池借色情游戏偷算力

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2083340
- 抓取时间: 2026-11-01T23:43:13

近期，火绒安全团队接到用户反馈称，发现一个后台程序会占用大量CPU资源，随后，火绒安全团队溯源查明，该病毒源自BitTorrent索引站中的日本色情游戏，病毒作者采用“白加黑”手段，使游戏在启动时加载恶意模块。该恶意模块会依次检测和逆向工具，最终通过创建傀儡进程注入挖矿木马以实施挖矿操作。该挖矿木马借助`XMRig`工具连接私人矿池，并通过计算`RandomX`哈希值来提供算力。在挖矿期间，该模块会长时间占用CPU性能和内存资源。若检测到任务管理器运行，挖矿行为将自动停止。目前，火绒安全产品可对上述木马进行拦截和查杀。

![](https://attach.52pojie.cn/forum/202512/30/181032x9zl1luv90h8p1de.png)
查杀图

![](https://attach.52pojie.cn/forum/202512/30/181058toatraa1lt9fsfrl.jpg)
最终图片 拷贝

下图显示，2025年12月18日，用户`hentaigames**`于Tokyo Toshokan BitTorrent索引站发布了该游戏的种子。此后，自2025年12月20日起，该种子内的游戏文件陆续被其他色情游戏论坛转载分享。另外，从用户`hentaigames**`所发布的游戏中随机选取4款进行测试，发现其中3款游戏存在挖矿病毒。基于此，推测该用户为病毒作者，而其他规模不一的色情游戏论坛仅起到传播作用。

![](https://attach.52pojie.cn/forum/202512/30/181142y4q3zqrluzd73z39.png)
Tokyo Toshokan BitTorrent 索引站_副本

![](https://attach.52pojie.cn/forum/202512/30/181156tbb1ehhifbewbie5.png)
大小论坛分享该游戏_副本

![](https://attach.52pojie.cn/forum/202512/30/181216ag61iwhhdd6oxgno.png)
论坛分享游戏的大小与 BT 种子索引站中的大小相近

下图显示，从BT种子索引站与论坛下载的压缩包都存在病毒文件`version.dll`，对比图左侧为论坛中下载，右侧为BT种子索引站下载。

![](https://attach.52pojie.cn/forum/202512/30/181235hqxyxywl61uxoqmq.png)
BT 种子索引站与论坛下载的压缩包内含文件对比图

经实测发现，该挖矿病毒会占用6%CPU性能与2GB左右内存用于计算哈希。

![](https://attach.52pojie.cn/forum/202512/30/181251uun9q89bn49t8r22.png)
实测 CPU 与内存占用

## 一、样本分析

溯源发现其中被动手脚的恶意动态链接库名有 `version.dll`、`cryptbase.dll`、`libEGL.dll` 等，在不同游戏中会选取其中一种执行恶意代码。其中 `version.dll` 为释放器、`cacheapp64.exe` 为注入器，最终目的为注入挖矿木马，期间会检测虚拟机、沙箱、杀毒软件等环境。而挖矿木马会连接私人矿池，接收矿池下发的任务（`job`）并计算哈希给矿池提供算力。

![](https://attach.52pojie.cn/forum/202512/30/181317yaakat3mxapikzvb.png)
流程图

下面主要分析名为 `version.dll` 的恶意动态链接库。

### 1.1 `version.dll`（释放器）

`version.dll` 被 `Kaiju Princess 2.exe` 加载：游戏主程序为 `Kaiju Princess 2.exe`，该主程序会依赖 `UnityPlayer.dll` 动态链接库，该动态链接库会依赖 `VERSION.dll`，从而加载第一个恶意文件 `version.dll`。该 `version.dll` 文件本该是 Windows 提供的 `C:\Windows\System32\version.dll` 文件，然而该病毒作者将插入了恶意代码的 `version.dll` 放在游戏目录下，从而使程序优先加载同目录下文件。
互斥体：该程序先通过互斥体（`Global\DECOY_MUTEtgtggttgX`）保证单实例运行，并创建新线程继续执行恶意代码。

![](https://attach.52pojie.cn/forum/202512/30/181355kgux3emmlnez05og.png)
互斥体与创建线程

检测 TPM（可信平台模块）：打开 TPM 设备确认是否为真机。

![](https://attach.52pojie.cn/forum/202512/30/181416hyienyvv45izjj2g.png)
打开 TPM 设备

随后休眠 1~3 秒，随后删除 `CSIDL_APPDATA\Microsoft\\servicing64\\rescache\\pcuapp64.exe`，创建 `CSIDL_LOCAL_APPDATA\syscacheapp` 目录。
检查文件 `cacheapp64.exe`（注入器）：随后开始检查 `cacheapp64.exe` 文件是否存在，并且检查其是否大于或等于750MB，同时检查是否存在`cacheapp64.exe`的自启动项等情况。若其中任意一项不满足条件，则会继续执行。

![](https://attach.52pojie.cn/forum/202512/30/181445hmpumv122ofpo5cv.png)
释放注入器前的逻辑

特征定位与解密：随后遍历主游戏程序同目录并找出 `*.pak` 文件，通过特征码 `-starttextures` 与 `-endtextures` 定位 `Base64` 数据之后进行提取，`Base64` 解码后利用 `RC4` 流密码算法进行异或解密，其中密钥为 `12345678901234567890123456789012`。
随机填充：随后会解密出 `cacheapp64.exe` 程序，还会往里面再次填充 750MB 减去当前 `cacheapp64.exe` 程序大小的随机数据，从而使最终文件大小来到 750MB。

![](https://attach.52pojie.cn/forum/202512/30/181501rq14e4udlqb5ea48.png)
通过特征码定位

![](https://attach.52pojie.cn/forum/202512/30/181527znloochwe6dodueo.png)
解密和随机写入填充

![](https://attach.52pojie.cn/forum/202512/30/181547l993zy72jb2e7bbx.png)
RC4 流密码算法

设置持久化：随后创建注册表 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell` 键值对，设置值为 `explorer.exe, C:\Users\Administrator\AppData\Local\syscacheapp\cacheapp64.exe`，从而完成持久化，使用户每次登录都会执行一次 `cacheapp64.exe`。

![](https://attach.52pojie.cn/forum/202512/30/181604zi8zezixexx6cpr7.png)
设置持久化
