
# [原创] 解包某电钢琴学习机的歌曲包

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1908192
- 抓取时间: 2026-11-01T22:33:52

## 0. 写在前面

这是大概两年前搞的，能搞出来纯属瞎猫碰上死耗子，看了一眼厂家的官网都不在了，发上来备份一下吧

由于个人水平不行纯粹是在瞎搞，帖子写得有点长，不想看的可以直接拉到最后面，提供了原版升级包和歌曲包下载（厂商官网已经下线，WaybackMachine 也没有备份，这里算是备份一下吧）和解包工具源代码

## 1. 背景

楼主的一个朋友无意在小黄鱼上发现了一批清仓处理的看起来像电子琴的玩意，带有可以显示歌词的屏幕和基于硬件 LED 的钢琴卷帘（类似 Synthesia 那种）显示，放张官网图：

![](https://attach.52pojie.cn/forum/202403/31/152748nwo3kcdbjxjmtt6d.gif)
`1-1FZ41HQ55E.gif` (124.48 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU1MHxkMTljNjA5M3wxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU1MHxkMTljNjA5M3wxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:27 上传

去官网看了一眼，内置快 3000 首歌曲，带有一层目录结构，官网还提供了升级包和歌曲包下载，于是果断下下来研究一下：

![](https://attach.52pojie.cn/forum/202403/31/152920zlfwb3n8vo3m1mln.png)
`屏幕截图 2024-03-31 032753.png` (994.15 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU1Mnw3OTllN2MwNHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU1Mnw3OTllN2MwNHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:29 上传

![](https://attach.52pojie.cn/forum/202403/31/152847p81cg0lgwzjo77pj.png)
`屏幕截图 2024-03-31 032855.png` (27.56 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY4NjU1MXwyYTlhY2MwMXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU1MXwyYTlhY2MwMXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:28 上传

## 2. 初步观察

其中小的是系统升级包，大点的是歌曲包，先来看升级包：

![](https://attach.52pojie.cn/forum/202403/31/153157orvizjj0ilc0n88k.png)
`屏幕截图 2024-03-31 033200.png` (167.6 KB, 下载次数: 7)
[href="forum.php?mod=attachment&aid=MjY4NjU1N3w1ZjhiODc1MXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU1N3w1ZjhiODc1MXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:31 上传

文件具有自定义的后缀名和文件头，暂时不知道里面是什么，用 7zip 打开试试：

![](https://attach.52pojie.cn/forum/202403/31/153352bokz9jyyggzxgk83.png)
`屏幕截图 2024-03-31 033337.png` (39.47 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY4NjU1OXw4NjQ0ZjQ3YnwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU1OXw4NjQ0ZjQ3YnwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:33 上传

![](https://attach.52pojie.cn/forum/202403/31/153357go7kfkczfqfpfyyg.png)
`屏幕截图 2024-03-31 033401.png` (112.87 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU2MHwzMTA1ZDhkNHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU2MHwzMTA1ZDhkNHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:33 上传

哦豁，竟然就是个普通 zip 包，而且里面就是 WinCE 平台的主程序，先记一笔，看看音乐包是个啥情况

官网提供了 3 种不同的音乐包，分别是【儿童版】【青年版】【长者版】，但下载下来之后发现大小完全相同：

![](https://attach.52pojie.cn/forum/202403/31/153610yu3lp0zj5rzhnah4.png)
`屏幕截图 2024-03-31 033545.png` (55.74 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU2MXw1NGQ1MGVhZXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU2MXw1NGQ1MGVhZXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:36 上传

推测内容应该一致，是使用了内部字段来控制应用内显示哪些歌曲，丢 WinHex 看眼先

## 3. 观察音乐包格式

头部是版本号，以及一大堆空白：

![](https://attach.52pojie.cn/forum/202403/31/154053nwrpru20trtnr0zg.png)
`屏幕截图 2024-03-31 033830.png` (91.56 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU2M3w4M2JiNzI0MXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU2M3w4M2JiNzI0MXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:40 上传

从 1KB 的位置开始，出现了很多非明文但大多重复的可疑内容

![](https://attach.52pojie.cn/forum/202403/31/154057fucr6hcurkv52z4v.png)
`屏幕截图 2024-03-31 033846.png` (383.94 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU2NHwyZWUyMWUyM3wxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU2NHwyZWUyMWUyM3wxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:40 上传

从 `0x056380` 偏移开始，出现了明文的 MIDI 数据（包括 `MThd` 头以及内嵌的 MIDI Text）：

![](https://attach.52pojie.cn/forum/202403/31/154102pqsgstt8bkjjz5lj.png)
`屏幕截图 2024-03-31 034014.png` (370.11 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU2NXw5ODBjOWRjOHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU2NXw5ODBjOWRjOHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:41 上传

到了这步，其实格式就已经很清晰了，前面很明显是经过某种加密或编码的目录信息（以某网游的数据包体进行类比，就相当于 `Trunk.dir`），后面的则是连续的明文歌曲数据。

理论上来讲，现在已经可以开始拆内容了，只需要按顺序寻找 `MThd` 文件头，将之间的内容保存为文件就可以了。但实际操作下来发现，很多 MIDI 文件里没有内嵌歌曲名称数据，也就意味着不研究一下目录数据格式的话，歌曲名称和目录名称都是搞不出来的。

那就先把主程序丢进 IDA 看眼吧，万一呢.png

## 4. 查证目录部分加密方式与密钥

已知歌曲包文件名为 `PianoCat.yym`，从字符串入手：

![](https://attach.52pojie.cn/forum/202403/31/155230syh1hiht6ikhgzzn.png)
`屏幕截图 2024-03-31 035203.png` (75.94 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY4NjU2OXxjM2E2NWVlZXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU2OXxjM2E2NWVlZXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:52 上传

![](https://attach.52pojie.cn/forum/202403/31/155231iokpup6pvu85oixl.png)
`屏幕截图 2024-03-31 035239.png` (83.33 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY4NjU3MHw3ODQzZTYwMHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU3MHw3ODQzZTYwMHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:52 上传

引用还不少，一个一个看过去，发现其中一个函数的日志中提到名称 `SongYYmDat_Find`，可能和寻找歌曲偏移有关，先改下函数名称

![](https://attach.52pojie.cn/forum/202403/31/155928qq9ac9b8db9k6dck.png)
`屏幕截图 2024-03-31 035938.png` (90.37 KB, 下载次数: 7)
[href="forum.php?mod=attachment&aid=MjY4NjU3M3xiNGMwNjgwYXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU3M3xiNGMwNjgwYXwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 15:59 上传

其中这个 `sub_14930` 的函数名根据日志可以看到叫做 `ReadFileShort`，用来读取某个文件从某偏移开始的固定长度，里面有很多 `COREDLL_XXX` 的函数，由于手头没有 WinCE 的 `CoreDLL.dll`，相关导入函数名称也没有，根据上下文猜测用途，一起重命名掉

![](https://attach.52pojie.cn/forum/202403/31/160505uiirpfievfskz1tq.png)
`屏幕截图 2024-03-31 040515.png` (35.01 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY4NjU3NHwxZWI1ODgyMnwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU3NHwxZWI1ODgyMnwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 16:05 上传

可以看到，首先从偏移 `1280` 开始读取了 `128` 字节，然后对这 `128` 字节数据进行了 `sub_394B0` 的处理。回去观察歌曲包数据：

![](https://attach.52pojie.cn/forum/202403/31/160926ney0y5etmmo2ihjt.png)
`屏幕截图 2024-03-31 040854.png` (224.63 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY4NjU3NXw3ODFmZTVjNHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU3NXw3ODFmZTVjNHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 16:09 上传

可以看到是从偏移 `1280` 开始，数据每 `32` 字节的头尾呈现明显的相似特征，而 `1024 - 1279` 这块似乎有个断层。那么 `sub_394B0` 在做什么呢？

![](https://attach.52pojie.cn/forum/202403/31/161234yz93t1numxs71ptt.png)
`屏幕截图 2024-03-31 041242.png` (28.68 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY4NjU3Nnw0NzkxZGNmZHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4NjU3Nnw0NzkxZGNmZHwxNzkzNTcyNDMwfDE2ODA4NnwxOTA4MTky&nothumb=yes)
2024-3-31 16:12 上传

刚看到还是有点迷惑的，但仔细看一下，就会发现这玩意做的事很简单：对于 `buffer` 里的每一个字节，使用 `word_1FA0F0 + 27174` 这个数组里的每一个字节（一共 `32` 字节，循环取）进行相减。C# 转写下：

[C#] 纯文本查看 复制代码
