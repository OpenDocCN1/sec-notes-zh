
# [原创] 记一次某 Rar 去广告和重新打包安装程序

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1663402
- 抓取时间: 2026-11-01T23:12:42

## 一、缘起

看到大佬“xiao-bai-shu”发的 RAR 去广告，自己也想实践一下，就鼓捣了一下，原贴在这：[https://www.52pojie.cn/thread-1660764-1-1.html](https://www.52pojie.cn/thread-1660764-1-1.html)

## 二、准备

1. `winrar-x64-591scp.exe`，手上正好有，就它了；
2. [win10](https://www.52pojie.cn/thread-661779-1-1.html)，这里推荐一下不忘初心的 win10 精简版 64 位，装完占用 10 个 G 这样，太棒了。虚拟机有个好处是可以避免乱七八糟的国产软件各种注入，导致程序停在意料之外的地方。
3. `X64dbg` 最新版

## 三、去广告

打开 `WinRAR.exe`，让程序停在入口，注意模块是不是 `winrar.exe`：

![](https://attach.52pojie.cn/forum/202207/19/170633fggejyg353zqe5c8.png)

右键——搜索——当前模块——跨模块调用，这个功能可以搜索出模块调用的 dll 函数：

![](https://attach.52pojie.cn/forum/202207/19/170916jkbz9epttt198epk.png)

搜索 `createwindowex` 函数，发现搜索出来的函数均为 `CreateWindowExW`，这里需要指出，windows 的 API 一般有两种，一种是 `A` 结尾的，使用的字符编码是本地 ANSI，另一种是 `W`，使用的字符编码是 Unicode，这说明 `winrar` 使用的是 Unicode 的字符串，这很有用。

![](https://attach.52pojie.cn/forum/202207/19/171239lfhyftt9hz2nhe9h.png)

F9 运行程序，停在了第一个 `CreateWindowExW`，注意调用的参数里有一个 `"WinRarWindow"`，猜测可能是主窗口，继续运行，依次出现 `"ReBarWindow32"`、`"ComboBoxEx32"`、`"ToolbarWindow32"`、`"SysListView32"`，还是比较容易猜出这是主窗口的初始化过程：

![](https://attach.52pojie.cn/forum/202207/19/171704dcsoochvvxvczyrb.png)

F9 继续运行，停在了下面这了：

![](https://attach.52pojie.cn/forum/202207/19/172313z68vg24r8sys6xq4.png)

可以看到上面的字样出现了 `ad.winrar.com.cn`，`ad` 一般是广告的意思，看来已经找到了创建广告窗口的地方。F8 单步步过，发现停在了其他 `CreateWindowExW` 调用上，F9 调过几次后从新回到开始找到的地方：

![](https://attach.52pojie.cn/forum/202207/19/173145o0ztkk030803hkz8.png)

这时候已经出现广告窗口了：

![](https://attach.52pojie.cn/forum/202207/19/173303xxatfickoo09796o.png)

将其他断点全部删除，只留下这个断点，然后重新运行，然后用 `nop` 填充 `call`，这里需要说明的是 64 位程序跟 32 位程序不一样，如果是 32 位程序你可能需要平衡堆栈：

![](https://attach.52pojie.cn/forum/202207/19/173646t9h9op93xj6ak33c.png)

F9 继续，运行，发现广告果然没有，右键——补丁——修补文件，找一个其他路径存下修改的 exe，退出 `x64dbg`，然后替换 `winrar.exe`。

跑一下，广告确实不弹了，但是标题有一个（离购买许可只剩 XX 天了）。直接调整虚拟机时间，改为 1 个月以后，发现这时候标题增加了（评估版本），并且弹出了购买提示：

![](https://attach.52pojie.cn/forum/202207/19/174617um93tzgm1rvggt9k.png)

看来只去广告是不行的，时间改回来，先攻克“离购买许可只剩 XX 天”。

设置窗口的标题可以在 `CreateWindowExW` 的时候直接设置，前面我们调试的时候发现主窗口的创建过程中没有出现 `Desktop` 关键字，说明不是创建时设置的标题。

![](https://attach.52pojie.cn/forum/202207/19/175132d4rx4rcpyc3nypcf.png)

那么还有另一个设置标题的函数，重新运行，在程序入口点，在当前模块中搜索跨模块调用，搜索 `setwindowtext` 函数，全部打上断点，F9 运行

![](https://attach.52pojie.cn/forum/202207/19/175440my5q5363n4yebx3q.png)

第一个就中了，nice~，虽然可以直接把字符串改成空字符串的，但是我不建议，还是需要寻根溯源，不然只是表象解决问题，实际还会有其他问题。

往上翻一翻，可以发现有一个

```asm
test al,al jne xxxxxx
```

![](https://attach.52pojie.cn/forum/202207/19/180538cn5xcx1nyzii7tiz.png)

高度怀疑这是关键点了，断点重新运行，改 `al` 的值为 `1`，果然成了：

![](https://attach.52pojie.cn/forum/202207/19/180656wsam9g0wwsf0assd.png)

查看 `al` 的值从哪里来，查看前一个函数，发现是直接读取内存的：

![](https://attach.52pojie.cn/forum/202207/19/192225np4jmtfek3i7zwlg.png)

进入函数，查找引用，发现这个函数还是有不少调用的：

![](https://attach.52pojie.cn/forum/202207/19/192443b4szliiyhoadeffi.png)

![](https://attach.52pojie.cn/forum/202207/19/192519foafm33ig99f3j9q.png)

高度怀疑就是这个内存地址存储的是否是注册软件。

查找引用——地址 `7FF6D98654F0`，发现只有 1 个改写，果断转到并下断点：

![](https://attach.52pojie.cn/forum/202207/19/192745kas6vtu7sp7pxuu1.png)
