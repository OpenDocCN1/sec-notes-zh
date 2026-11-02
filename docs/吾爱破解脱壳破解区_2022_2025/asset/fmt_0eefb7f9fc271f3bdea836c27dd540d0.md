
# [原创] 某CAD论坛里的一个CM分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1630970
- 抓取时间: 2026-11-01T23:27:19

第一次做这种分析，可能难免有些不对的地方，也请大家多多指教。另外先说明一下，我已经先在那个论坛里把答案贴了出来，下图是我在那个论坛的会员截图（为避免广告，论坛logo已经打码）

![](https://attach.52pojie.cn/forum/202205/03/133120w6a0g0g5p57a2b7g.png)

Snipaste_2022-05-03_13-01-25.png (213.76 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUxOTE4Nnw3NTBlMzlkY3wxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTE4Nnw3NTBlMzlkY3wxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 13:31 上传

根据作者的说法，在CAD里加载了`syz-int.fas`后，通过`(syz-write-int "abcd.txt" 1415927)`就可以把1415927写到`c:\programdata\abcd.txt`中，并且`abcd.txt`的文件大小为零。但当你用`(princ (syz-read-int "abcd.txt" ))`去读取`abcd.txt`时，你会发现CAD可以读出1415927。作者给出的要求是找出这个数字藏在了什么地方。

## 分析过程

（我这边用7355608作为写入的整数）

1、在AutoCAD里完成对`syz-int.fas`的加载后，打开火绒剑，监控了一下系统的操作，发现确实只有`acad.exe`在对文件进行写操作。

![](https://attach.52pojie.cn/forum/202205/03/134528fokxodm0ssezdoki.png)

Snipaste_2022-05-03_13-45-05.png (932.37 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUxOTE5MnwyN2JhNTQ5ZXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTE5MnwyN2JhNTQ5ZXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 13:45 上传

此时在`C:\ProgramData`下确实只有一个0字节的`52pojie.txt`文件。

![](https://attach.52pojie.cn/forum/202205/03/134630v0qvcaq3svccznyh.png)

Snipaste_2022-05-03_13-46-05.png (35.65 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUxOTE5NHw0MzQxZWZkNXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTE5NHw0MzQxZWZkNXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 13:46 上传

接着在AutoCAD里读取这个文件，确认可以正常读出数值。

![](https://attach.52pojie.cn/forum/202205/03/135107wxmbxb919fe99akf.png)

Snipaste_2022-05-03_13-50-58.png (19.45 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUxOTIwMXwwOTZjYzg0MnwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTIwMXwwOTZjYzg0MnwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 13:51 上传

既然这个数不在文本里，但又和这个文件有关联，那99%会藏在“不可视境界线”里。

于是祭出x-ways forensics，打开C盘，直奔这个文件

![](https://attach.52pojie.cn/forum/202205/03/135734p3sb44jihw3wh1i8.png)

Snipaste_2022-05-03_13-56-59.png (254.88 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUxOTIwNnw5Y2I2NmZlNXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTIwNnw5Y2I2NmZlNXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 13:57 上传

不出所料，在这个文件里看到了一组“D83C7000”的十六进制数。

7355608的十六进制是703CD8，恰好就是D83C70倒序输出。所以，可以确定那位作者是把整数藏在了文件的开头部分。

为了方便对比，我也建了一个`empty.txt`的空白文件，大家应该可以发现这两个文件的差异了。

![](https://attach.52pojie.cn/forum/202205/03/140259mlslgksqysllgc8y.png)

Snipaste_2022-05-03_14-02-36.png (61.85 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUxOTIwOHxjZDE2NWY1NHwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTIwOHxjZDE2NWY1NHwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 14:02 上传

![](https://attach.52pojie.cn/forum/202205/03/140302y3nmltom6plyh8uk.png)

Snipaste_2022-05-03_14-03-04.png (61.42 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUxOTIwOXxhODhkM2FmOXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTIwOXxhODhkM2FmOXwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw&nothumb=yes)

2022-5-3 14:03 上传

最后把作者的源文件放在附件里，供大家尝试。

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[href="forum.php?mod=attachment&aid=MjUxOTIxNHw1MWE4YjE4NHwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTIxNHw1MWE4YjE4NHwxNzkzNTc1NjM3fDE2ODA4NnwxNjMwOTcw)

(1 KB, 下载次数: 1)

2022-5-3 14:06 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB

啥？不知道怎么加载？

把`syz-int.fas`复制到`c:/APPS/`下，然后在CAD里通过`APPLOAD`命令加载那个`lsp`文件即可。当然也可以用记事本打开那个`lsp`文件，把`fas`文件的路径改成自己当前的。
