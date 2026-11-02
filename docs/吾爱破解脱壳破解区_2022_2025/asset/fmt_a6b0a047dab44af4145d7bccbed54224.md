
# [原创] 破解“某MP3分割器”

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1777542
- 抓取时间: 2026-11-01T22:53:38

## 背景

我朋友在使用一个MP3分割器，每次导出MP3的时候都跳十秒钟广告。但广告居然有脸写自己是有良好口碑的免费软件，着实是很矛盾。但是我还是劝我朋友使用`ffmpeg`，不要用这种软件了。不过还是想尝试[//www.52pojie.cn" target="_blank" class="relatedlink"](https://www.52pojie.cn:href="https)一下。

![](https://attach.52pojie.cn/forum/202304/23/143450ld9635cs3s59wp9p.png)

`1.png` (33.84 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwOTc0Mnw0MjU0ZWVhNHwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTc0Mnw0MjU0ZWVhNHwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes)

2023-4-23 14:34 上传

## 查壳与反编译

先用`DIE`查壳，显示：

```
Compiler: Borland Delphi(7)[-]
Linker: Turbo Linker(2.25*,Delphi)[GUI32]
```

我可从来没调试过Delphi的软件，这下有点害怕。

我下载了`Delphi Decompiler`，本以为能直接反编译出Delphi语言，结果反编译出来是窗口和控件信息，还有控件对应方法的汇编代码。

![](https://attach.52pojie.cn/forum/202304/23/143853ibbpbz7xba7x6ob5.png)

`2.png` (23.72 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYwOTc0M3w3YjliZDRlZXwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTc0M3w3YjliZDRlZXwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes)

2023-4-23 14:38 上传

但是元数据都没有混淆，`TFrmSponsor`一看就是捐助窗口。

## 分析按钮逻辑

打开`BtnCloseClick`的过程，显示：

```
0048E610 C7804C02000007000000 mov dword ptr [eax+$024C], $00000007
0048E61A C3 ret
```

看来这个按钮确实是关闭窗口的。

打开打开`BtnDoClick`的过程，显示：

![](https://attach.52pojie.cn/forum/202304/23/144208pz7bjczcow680617.png)

`3.png` (30.88 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYwOTc1MHw3OWRhMWI1ZXwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTc1MHw3OWRhMWI1ZXwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes)

2023-4-23 14:42 上传

显然从`push ebx`到`ret`是这个按钮的过程。

## 修改与验证

在`ollyice`中无脑把两个`call`指令`nop`掉，成功显示：

![](https://attach.52pojie.cn/forum/202304/23/144436qjudv99bl7l8fvjj.png)

`4.png` (12.27 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwOTc1MnxhMzRmYThjYnwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTc1MnxhMzRmYThjYnwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy&nothumb=yes)

2023-4-23 14:44 上传

一些朋友说看不懂，其实我也不明白机理，我是根据经验把两个`call` `nop`掉的，先`nop`的第一个看成功否，再`nop`了第二个。

## 附件

原文件和`cracked`

![](https://static.52pojie.cn/static/image/filetype/rar.gif)

[href="forum.php?mod=attachment&aid=MjYwOTc1NHw3ODkyMmY0NXwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTc1NHw3ODkyMmY0NXwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy)

(1.29 MB, 下载次数: 598)

2023-4-23 14:47 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB

sjc140400制作的MP3分割软件：

![](https://static.52pojie.cn/static/image/filetype/rar.gif)

[href="forum.php?mod=attachment&aid=MjYwOTgxMXxjYWUzZDZjMHwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwOTgxMXxjYWUzZDZjMHwxNzkzNTczNjE2fDE2ODA4NnwxNzc3NTQy)

(489.47 KB, 下载次数: 411)

2023-4-23 20:38 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB
