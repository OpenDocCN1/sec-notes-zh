
# [CTF] flare-on 2023 第四题aimbot分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1921051
- 抓取时间: 2026-11-01T22:29:48

## 前言

我手贱看了一眼官网的荣誉榜，发现前几名神仙打架，第一名只用了2天的时间解决了所有的题，神的世界真离谱。。。想了解一下神的思路，在推上找到了第一名(jimo123)，大概率韩国人，可惜推上显示没时间写writeup。不能了解神的思想，很痛苦。。。

回归正题，这题主要考验反反调试能力和对shellcode的理解力。这题也有点俄罗斯套娃，与第三题不相上下。。第三题更像是恶意软件在隐藏自身，而这题就是干会坏事了，

这个题的程序表面是个外挂程序，但它在后台默默偷取数据。

## 文件分析

exeinfo PE 显示文件用 x64 - MinGW-w64 GCC编译的，resources显示99%（有东西），用ResourceHacker得到三个文件，加密了识别不了。

运行程序后，有个按钮，点击没反应。

## `aimbot.exe`的ida分析

跟着窗口函数，来到主要逻辑地方`sub_402150()`函数，分析`sub_402150()`函数，它主要做了以下几件事：

1. 打开`"%PROGRAMFILES(X86)%\Sauerbraten\bin64\sauerbraten.exe"`文件，我将`aimbot.exe`和三个资源文件都试了不对，后面才知道是个游戏得下载（http://sauerbraten.org/）。

2. 计算`sauerbraten.exe`的md5 与硬编码比较

   ![QQ图片20240505184233.png (20.75 KB, 下载次数: 3)](https://attach.52pojie.cn/forum/202405/06/204006hf2tf6fflkllml2l.png)

   [2024-5-6 20:40 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2MHw5NmQyNjI0Y3wxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

3. 通过AES使用密钥`"yummyvitamincjoy"`解密三个资源，并将3个文件`miner.exe`、`config.json`、`aimbot.dll`提取到文件夹`%APPDATA%\BananaBot`中。

4. 从文件`sauerbraten.exe`（游戏）创建一个进程，然后调用函数`sub_401E80()`，`sub_401E80()`函数使用先前解密的`aimbot.dll`，将 DLL 注入到文件刚刚创建的游戏进程中。

   ![QQ图片20240505184818.png (42.86 KB, 下载次数: 3)](https://attach.52pojie.cn/forum/202405/06/204048q2ag0s55rrrsvvg5.png)

   [2024-5-6 20:40 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2M3w0ZDI4OTkzNnwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

所以我们在`aimbot.exe`按下按钮后，会执行几个操作，1. 在`%APPDATA%\BananaBot`文件夹中创建三个文件。2.运行文件`miner.exe`。 3.运行游戏，注入`aimbot.dll`。

`miner`是个开源、跨平台的挖矿程序，不用分析，接下来分析`aimbot.dll`。

## `aimbot.dll` 分析

来到`dllmain()`，`aimbot.dll`创建了三个线程，分别查看每个线程：

![QQ图片20240505192720.png (30.37 KB, 下载次数: 4)](https://attach.52pojie.cn/forum/202405/06/204113d2hpfydt2dhhyt6h.png)

[2024-5-6 20:41 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2NHw5ZjU4M2IzOXwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

第一个线程包含对`GetAsyncKeyState()`的调用，并在用户按下 5 小键盘数字键和 6 小键盘数字键时执行某些操作（作弊），当在游戏过程中按下小键盘数字键5时，玩家角色锁定最近的敌人并执行射击。当按下 6 小键盘的数字键时，每当敌人经过玩家的目标时，玩家角色就会开枪。游戏里试了下，5的功能勉强能玩玩。。6有点废物。所以第一个线程不重要pass。

第三个线程执行调试反操作。它运行一个无限循环来检查进程是否处于调试状态，如果是，则调用`ExitProcess()`函数并结束进程。

`aimbot.dll`用了许多反调试技术，明面的Windows api: `IsDebuggerPresent()`、`CheckRemoteDebuggerPresent()`、`DbgBreakPoint()`全与反调试相关。

第二个线程是主要的逻辑，`sub_62F43070()`函数最后的shellcode 被解码后运行。为了获得解码后的代码需要动态调试对其进行分析。但由于第三个线程中的反调试，不可能直接附加到游戏进程。为了解决这个问题，我们将在游戏进程创建之后，在第三个线程开始运行之前附加到游戏进程。

我对反反调试技术一知半解，所以我打算使用ScyllaHide插件，然后结合`NOP`大法和改`rip`来实现动态反调试，后面的方法要注意堆栈平衡，不然容易崩。。。为了防止不知道什么情况导致程序崩了，从头再来，使用虚拟机快照保存每个值得快照的阶段。

为了能附加到游戏进程，在`aimbot.exe`进程中的`sub_401E80()`函数上设置断点。运行程序并按下 BananaAimBot 窗口中的按钮。游戏打开后，`aimbot.exe`进程会停在断点处。我们将附加到游戏进程，并在`CreateThread()`函数上设置断点。然后继续运行游戏进程，直到在`CreateThread()`函数上的断点处停止。我们将继续运行直到创建第二个线程之前。在创建它之前，我们将在它的线程函数的开头设置一个断点，即`sub_62F43070()`。我们将继续运行，直到创建第三个线程之前。在创建第三个线程之前，我们将跳过对`CreateThread()`的调用（通过编辑`rip`寄存器或将调用`CreateThread()`的代码更改为`NOP`）。

接下来分析第二个进程的主要逻辑`sub_62F43070()`函数。

该函数开头是检查当前时间，休眠 5 分钟，并检查已经过去的时间确实至少为 5 分钟。如果测试通过，则执行该函数的逻辑（`rip`大法）。

然后它创建文件夹`C:\depot`。接着调用函数`sub_62F43A20()`，分析后改为`requestData_fromUrl()`。

![QQ图片20240505212422.png (54.15 KB, 下载次数: 3)](https://attach.52pojie.cn/forum/202405/06/204135mrw11kn4pp9ddryr.png)

[2024-5-6 20:41 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2NXw3MjgwNWJkMnwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

`get_key1_62F42300()`函数被多次用到，需重点分析一下。`requestData_fromUrl()`函数里也用到它，通过它解密url相关的字符串。在`get_key1_62F40230()`里，先是反调试，然后调用了`sub_62F42020()`，主要逻辑就在这个函数。在这个函数中，使用了更多的反调试技术：

1. `NtQueryInformationProcess()` 反调试。

2. 使用 `PEB` 结构中的 `BeingDebugged` 标志并检查其值是否为零。

3. 该函数对计算机上运行的所有进程执行枚举过程，计算每个进程名称的哈希值，并在预定义的哈希值数组中查找哈希值。当数组中的哈希值与计算机上运行的进程名称匹配时，该函数会增加计数器并重置数组中与该进程相对应的值。可以调试得到相关进程的名字（如果你有运行的话），例如`explorer.exe`、`x64dbg.exe`、`ida.exe`，在`0x62F42246`上设置断点，查看相关的进程名称。

![QQ图片20240505221311.png (34.86 KB, 下载次数: 2)](https://attach.52pojie.cn/forum/202405/06/204148qn0444wrbviwchyz.png)

[2024-5-6 20:41 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2N3w4NmQyZDg0MXwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

![QQ图片20240505221603.png (45.62 KB, 下载次数: 3)](https://attach.52pojie.cn/forum/202405/06/204156vkqb0x000xv9u9az.png)

[2024-5-6 20:41 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2OHwyNjRiZWQwZnwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

![QQ图片20240505221645.png (30.59 KB, 下载次数: 1)](https://attach.52pojie.cn/forum/202405/06/204209lzkpf58qyeps88rz.png)

[2024-5-6 20:42 上传](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA2OXwxYWMzYTA5M3wxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)

函数的最后会通过相加以下几个值，作为返回值。

1. 从父进程（`aimbot.exe`进程）内存中偏移量`0x406220`处读取4个字节。
2. 常数 `0x1337 + 程序计数`。
3. 从当前进程（游戏）内存中偏移量`0x138E`处读取4个字节。

动态变化的只有程序计数。通过动态调试，当程序计数为1时（即只检测到`explorer.exe`，没有其他的调试进程），函数的返回值为`0x6499F8A9`，这个值可以解密字符串。

为了防止反调试干扰，直接在`get_key1_62F42300()`函数开头修改：
