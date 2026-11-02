
## Thunderbird 数据窃取
利用 `RtlExpandEnvironmentStrings` 展开 `%AppData%\Thunderbird\Profiles`，之后通过 `NtQueryAttributesFile` 查看该文件夹是否存在。如果存在则会利用通配符 `*` 遍历所有文件夹，并从中获取以下内容：

- `key4.db`（账户密码）
- `cert9.db`（数字证书）
- `formhistory.sqlite`（历史记录）
- `cookies.sqlite`（cookie）
- `logins.json`（登录凭据）
- `places.sqlite`（书签）

最后写入 Thunderbird 文件夹中并压缩。

![](https://attach.52pojie.cn/forum/202410/22/162302hxxxmvkbb9bzzi9d.png)

Image-30.png (28.75 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4NXw4ZDMwMTA3MnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4NXw4ZDMwMTA3MnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

压缩包

## 获取 Steam Token
通过上述遍历进程名的方法查看 `steam.exe` 是否存在，如果存在则利用 `NtOpenProcess` 打开进程句柄并通过 `NtReadVirtualMemory` 读取内存查看是否存在 `eyAidHlwIjogIkpXVCIsICJhbGciOiAiRWREU0EiIH0` 字符串，如果有则获取该 Token 并解密 `Base64` 提取出 `sub` 的值，相加保存到 `Applications/Steam/Tokens.txt` 压缩。

![](https://attach.52pojie.cn/forum/202410/22/162304ibbexavudhsnajbe.png)

Image-31.png (58.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4NnwyZWFhM2JiMHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4NnwyZWFhM2JiMHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

压缩包和 Tokens.txt

获取 Steam Token 的流程与 2024 年 4 月火绒安全实验室发布的[//www.52pojie.cn/thread-1912507-1-1.html" target="_blank"](https://www.52pojie.cn/thread-1912507-1-1.html:href="https)文章中提到的盗取 Token 逻辑相同，具体细节可查看往期分析报告。

## 获取 Discord Token
Discord 有三种版本，分别对应路径：

- `%AppData%\Discord`（稳定版本）
- `%AppData%\DiscordCanary`（测试版本）
- `%AppData%\DiscordPTB`（介于中间的版本）

三者都会读取 `%AppData%\[DiscordName]\Local State` 文件，获取其中的 `os_crypt.encrypted_key` 并通过 `Base64` 解密，然后调用 `CryptUnprotectData` 函数解密后将数据写入 `[DiscordName]\dp.txt` 文件并压缩。

## 获取 NotePad++ TXT 文件
利用 `RtlExpandEnvironmentStrings` 展开 `%AppData%\Notepad++\session.xml`，利用 `NtOpenFile` 打开句柄，利用 `NtQueryInformationFile` 获取文件大小，之后通过 `NtReadFile` 读取文件内容，从中读取 `filename="` 开头和 `"` 结尾的 TXT 文件，并存储到 `Important Files/Notepad++/` 压缩。

![](https://attach.52pojie.cn/forum/202410/22/162306un75c2vj7nc74sn7.png)

Image-32.png (21.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4N3xiZjJlODkwZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4N3xiZjJlODkwZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

压缩包

## 获取已安装列表
利用 `NtOpenKeyEx` 打开注册表句柄 `\REGISTRY\MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` 并利用 `NtQueryKey` 获取键的数量。

之后利用 `NtEnumerateKey` 遍历注册表，通过 `NtQueryValueKey` 获取 `DisplayName`，最后写入 `Software.txt` 并压缩。

![](https://attach.52pojie.cn/forum/202410/22/162309i66i26e6a6i96u8h.png)

Image-33.png (179.83 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4OHw1M2M0ZjkwMnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4OHw1M2M0ZjkwMnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

压缩包和 Software.txt

## 获取进程列表
利用 `NtQuerySystemInformation` 函数传入参数 `SystemProcessInformation` 获取进程信息，将进程名写入到 `Processes.txt` 中并压缩。

![](https://attach.52pojie.cn/forum/202410/22/162311ye3mwijyamn2ngtm.png)

Image-34.png (79.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4OXw3MDhkNWM0ZnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4OXw3MDhkNWM0ZnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

压缩包和 Processes.txt

## 获取系统信息
以下是获取到的系统信息以及大致方法。

- `Path`：通过 `fs:[30]` 获取 `PEB`，通过偏移 `PEB->ProcessParameters->ImagePathName.Buffer` 获取当前进程路径
- `OS Version`/`Local Date`/`Time Zone`/`Install Date`：通过 `WMI` 命名空间 `ROOT\CIMV2`，使用 `WQL` 查询 `SELECT * FROM Win32_OperatingSystem`，分别获取 `Caption`（版本名）、`LocalDateTime`（时间）、`CurrentTimeZone`（时区）、`InstallDate`（安装时间）
- 位数：`mov eax, cs` 判断
- `Elevated`：`NtQueryInformationToken` 传入参数 `TokenElevation` 获取是否已提权
- `Computer`：`GetComputerNameA` 获取计算机名
- `User`：`GetUserNameA` 获取用户名
- `Domain`/`Hostname`/`NetBIOS`：`GetComputerNameExA` 分别传入参数 `ComputerNamePhysicalDnsDomain`/`ComputerNamePhysicalDnsHostname`/`ComputerNamePhysicalNetBIOS`
- `Language`：`GetUserDefaultLocaleName` 获取系统语言
- `Anti Virus`：通过 `WMI` 命名空间 `ROOT\SecurityCenter2`，使用 `WQL` 查询 `SELECT * FROM AntiVirusProduct` 获取 `displayName`（防病毒软件的名称）
- `HWID`：通过 `WMI` 命名空间 `ROOT\SecurityCenter2`，使用 `WQL` 查询 `SELECT * FROM Win32_BIOS` 获取 `SerialNumber`（序列号），再经过计算获得
- `RAM Size`：`GetPhysicallyInstalledSystemMemory` 获取物理内存大小
- `CPU` 相关：通过 `cpuid` 获取
- `GPU`：`EnumDisplayDevicesW` 获取设备信息
- `Display resolution`：`EnumDisplaySettingsW` 获取分辨率

下图是包含了系统信息的 `System.txt` 文件，可以看出该 Lumma Stealer 是 2024 年 9 月 2 号编译的。

![](https://attach.52pojie.cn/forum/202410/22/162313oe7etn77ee4b729s.png)

Image-35.png (42.52 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY5MHxiMGViOTc0M3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY5MHxiMGViOTc0M3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

系统信息

## 获取剪切板数据
利用 `GetClipboardData` 获取当前剪切板数据，将数据写入到 `Clipboard.txt` 文件后压缩。

![](https://attach.52pojie.cn/forum/202410/22/162315u2t7fwqk8pp8ifce.png)

Image-36.png (61.73 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY5MXxiOTc1ZjkxM3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY5MXxiOTc1ZjkxM3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

剪切板数据

## 附录

### C&C

### HASH
![](https://attach.52pojie.cn/forum/202410/22/162319tf8k2hr5bh0h08b8.png)

Image-38.png (12.84 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY5M3wwZjc5OGZhNXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY5M3wwZjc5OGZhNXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

### 讲点大白话
有的小伙伴表示没有学过计算机知识，看不太懂这篇文章，那么你可以参考如下说明。

![](https://attach.52pojie.cn/forum/202410/22/162154n2cn0uopcn3dlg51.gif)

GIF表情包.gif (9.2 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY1NHwxODExZDIzY3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY1NHwxODExZDIzY3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:21 上传

简单来说，网络世界存在一个对你个人信息图谋不轨的“小偷”，盗窃木马Lumma Stealer就像是它的“偷窃工具”，这个“小偷”很狡猾，它用一种叫做Go语言的编程语言来充当自己的伪装工具，伪装“偷窃工具”的“合法身份”，并通过一些加密手法来隐藏工具的行踪，使得它难以被发现和抓住。同时“偷窃工具”会悄悄潜入你的电脑，偷看你的个人信息，比如你的邮箱、Steam账户、Discord账户，甚至包括你电脑里的TXT文件、剪切板、截屏等数据。它还会根据“小偷”的指令来决定要偷哪些数据。

但是别担心，目前火绒已经可以拦截和清除这种木马了。就像有警察保护我们不受“小偷”侵害一样，所以，记得更新你的火绒病毒库，守护你的电脑安全哦~
