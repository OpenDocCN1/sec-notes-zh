
# [PC样本分析] GitHub开源项目被投毒，后门病毒跟随开发流程传播蔓延

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2002104
- 抓取时间: 2026-11-01T23:45:54

现如今，代码的开放给软件开发提供了诸多便利，`GitHub`就是其中极具代表性的平台。然而随着“开放”逐渐广泛，其被恶意利用的风险也随之增加。近年来，一种隐蔽又危险的攻击手段——代码投毒，悄悄成了威胁开发者和用户安全的隐患。攻击者会在开源项目或代码库里植入有害代码，这些看似正常的代码，一旦被开发者无意中引入，就会在开发链条中蔓延，最终让用户也面临安全风险。
近期，火绒威胁情报中心监测到一批`GitHub`投毒的恶意样本。经火绒工程师分析，该木马利用多种编程语言（包括`JavaScript`、`VBS`、`PowerShell`、`C#`、`C++`）构建复杂的进程链，最终实现恶意后门木马功能。这种投毒攻击方式不仅隐蔽性强，还可能通过软件供应链传播到更广泛的用户群体中。目前，火绒安全产品已具备对该类`GitHub`投毒样本的精准识别能力，能够有效阻断其加载过程，为用户系统提供可靠的安全保障。为了进一步增强系统防护能力，火绒安全建议广大用户及时更新病毒库。这将确保您的系统能够抵御最新威胁，防范潜在安全风险。

![](https://attach.52pojie.cn/forum/202501/23/185720nls6zzzh6lqlahza.png)
Image-0.png (35.93 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQxN3w0ZjU1OGEwZnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQxN3w0ZjU1OGEwZnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
查杀图

## 一、GitHub投毒事件及潜在风险

尽管此类攻击看似主要针对开发者，但其潜在影响却远不止于此，甚至可能波及每一位普通用户。以下将详细介绍此类攻击的运作方式及其可能带来的风险，希望广大用户在了解后能够进一步提升终端安全防护意识。

### 攻击的起点——开发者

攻击者利用`GitHub`等开源平台的开放性，将恶意代码伪装成正常工具，诱骗开发者下载和使用。而近年来，`GitHub`投毒攻击已成为网络安全领域备受关注的热点话题。开发者作为软件的构建者，一旦不慎落入陷阱，这些恶意代码便会悄无声息地潜入他们的开发环境。而开发者编写的代码和使用的工具，最终将会转化为我们日常生活中所依赖的应用程序和服务。因此，一旦开发环境被污染，安全隐患便会顺着开发流程蔓延，进而影响到每一位普通用户。

### 供应链的连锁反应

在软件开发的生态系统中，开发者所使用的工具和库往往会通过软件供应链传播到更广泛的应用中，它们会通过层层传递，融入到更广泛的应用程序中。例如：
一个被篡改的开源库可能被集成到多个软件项目中。由于开源库的广泛使用和高度共享性，这种篡改行为很容易在软件开发过程中传播开来。
这些软件项目最终会发布到应用商店，或者通过更新机制推送到用户的设备上。在这个过程中，恶意代码可能会随着软件的分发而扩散到用户手中。
因此，即使我们从未直接接触过`GitHub`或开源代码，也可能因为使用了某个被感染的软件而成为受害者。

### 潜在风险

隐私泄露：恶意代码可能会窃取个人信息，如账号密码、浏览器Cookies等。
远程控制：攻击者可能通过远程访问工具控制设备，甚至监控使用者的一举一动。
经济损失：窃取的信息可能被用于欺诈或勒索，导致使用者遭受财产损失。
数据勒索：恶意代码可能导致设备运行缓慢、频繁崩溃，以及加密数据以勒索设备拥有者。
而火绒威胁情报中心监测到的这一批恶意样本，就存在盗取信息导致隐私泄露、远程控制等风险，以下为样本分析内容。

## 二、样本分析

![](https://attach.52pojie.cn/forum/202501/23/185728gagbhucux9hshbd8.png)
Image-1.png (90.73 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQxOHw0MmQwMGU5MXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQxOHw0MmQwMGU5MXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
执行流程图

### 2.1 样本信息

该样本通过利用`Visual Studio`的`PreBuildEvent`机制执行恶意命令，生成`VBS`脚本以下载`7z`解压工具和恶意压缩包`SearchFilter.7z`。解压后，样本加载一个基于`Electron`框架的程序，该程序具备反调试和[//www.52pojie.cn/thread-661779-1-1.html" target="_blank" class="relatedlink"](https://www.52pojie.cn/thread-661779-1-1.html:href="https)检测功能，能够规避安全分析环境。随后，程序从`GitHub`下载并解压第二个恶意压缩包`BitDefender.7z`，进一步释放多个恶意模块，包括后门工具（如`AsyncRAT`、`Quasar`、`Remcos`）、剪贴板劫持组件以及`Lumma Stealer`窃密木马。
这些模块共同构建了一个多层次、功能完备的恶意行为链，能够实现远程控制、剪贴板内容篡改、浏览器Cookie及其他敏感信息窃取等破坏性操作，对用户系统安全构成严重威胁。

### 2.2 Loader动静态分析

首先，病毒利用`GitHub`进行传播，通过`Visual Studio`的`PreBuildEvent`命令执行`cmd`命令。

![](https://attach.52pojie.cn/forum/202501/23/185735xegjzlewgahhdgxz.png)
Image-2.png (60.55 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQxOXwxZGEyYjI3MnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQxOXwxZGEyYjI3MnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
测试执行`calc`

![](https://attach.52pojie.cn/forum/202501/23/185740fw01sgwawhvq2wne.png)
Image-3.png (37.86 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyMHw0MmY4NjVmNnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyMHw0MmY4NjVmNnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
配置文件

以下是`GitHub`中`Apex-Legends-External-Cheat-Hack-Trigger-Glow-Aimbot-Skin-More-Hwid-Spoofer`外 挂作弊软件项目中的恶意命令。

![](https://attach.52pojie.cn/forum/202501/23/185745odo8ppbttq0oppx4.png)
Image-4.png (367.54 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyMXwxNzFkYTkyN3wxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyMXwxNzFkYTkyN3wxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
恶意命令

对其执行的命令进行分析，发现攻击者采用了`Base64`加密的`VBS`脚本。解密后，脚本内容被写入到名为`b.vbs`的文件中。

![](https://attach.52pojie.cn/forum/202501/23/185753vcn9ll3360f3lj3b.png)
Image-5.png (200.03 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyMnw0M2ViMGFlOXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyMnw0M2ViMGFlOXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
`Base64`加密

对其进行解密。

![](https://attach.52pojie.cn/forum/202501/23/185759qyrglym10shozpir.png)
Image-6.png (235.46 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyM3wwM2I1YWQzYXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyM3wwM2I1YWQzYXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:57 上传
解密

解密后的`VBS`脚本通过反转字符串、`Base64`解码等方式对混淆内容进行处理，并利用`Invoke-Expression`执行解码后的指令。此外，脚本通过调用`pWN $bnb`来运行隐藏的恶意逻辑。

![](https://attach.52pojie.cn/forum/202501/23/185805ftj0g000j9b89508.png)
Image-7.png (203.86 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyNHxhYmRhZDE1YnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyNHxhYmRhZDE1YnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:58 上传
解密`VBS`脚本

最终解密之后是一段`PowerShell`脚本，主要功能如下。
1.多层递归调用与`Base64`解码：脚本通过多层递归调用和`Base64`解码技术，逐步解析出隐藏的下载地址，确保其隐蔽性。
2.下载`7z`解压工具：脚本从指定地址下载`7z`解压工具，并将其保存到`C:\ProgramData\sevenZip`目录中。
3.执行`SearchFilter.exe`：在下载并解压相关文件后，脚本会执行`SearchFilter.exe`，以加载后续的恶意功能。
4.清理痕迹：脚本会在执行完成后删除下载的压缩包文件。

![](https://attach.52pojie.cn/forum/202501/23/185810zw4gttwwkg7kw7qj.png)
Image-8.png (93.65 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyNXxiZDhmYmUxOXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyNXxiZDhmYmUxOXwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:58 上传
下载`7z`

![](https://attach.52pojie.cn/forum/202501/23/185817thbo36h2woziwj2a.png)
Image-9.png (68.74 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc1MTQyNnw0Mzc2NjM0N3wxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyNnw0Mzc2NjM0N3wxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:58 上传
下载执行恶意程序

解密关键信息：
下载链接：
[//github.com/Fxkw45/delhi-metro/releases/download/metro/SearchFilter.7z" target="_blank"](https://github.com/Fxkw45/delhi-metro/releases/download/metro/SearchFilter.7z:rel="nofollow noopener" href="https)
该链接指向托管在`GitHub`上的恶意压缩包`SearchFilter.7z`。
解压密码：
`hR3^&b2%A9!gK*6LqP7t$NpW`
该密码用于解压下载的`SearchFilter.7z`文件，以释放其中的恶意内容。

![](https://attach.52pojie.cn/forum/202501/23/185824wffeutlw949xutkr.png)
Image-10.png (23.6 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc1MTQyN3w0NzVkZjNmN3wxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyN3w0NzVkZjNmN3wxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:58 上传
解密链接与解压密码

其下载链接指向了一个Fork的`GitHub`项目，攻击者利用`GitHub`的合法外衣来托管恶意样本。

![](https://attach.52pojie.cn/forum/202501/23/185830h2znqpz2e89sz89x.png)
Image-11.png (74.43 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc1MTQyOHw1ZDc0ZWJlYnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQyOHw1ZDc0ZWJlYnwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:58 上传
`GitHub`链接

使用解密的解压密码对其进行解压，其主体是一个`Electron`程序的病毒。

![](https://attach.52pojie.cn/forum/202501/23/185837kr690fox6p5i56z5.png)
Image-12.png (99.66 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc1MTQzMHwwYzExNDc1OHwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MTQzMHwwYzExNDc1OHwxNzkzNTc2NzUxfDE2ODA4NnwyMDAyMTA0&nothumb=yes)
2025-1-23 18:58 上传
`Electron`程序

对`Electron`程序进行解包后，发现其主文件是一个大小为1.03MB的`JavaScript`脚本文件，包含20000多行代码，且代码经过高度混淆。为了还原其逻辑，采用以下方法进行解混淆。
1.`AST`（抽象语法树）分析：
