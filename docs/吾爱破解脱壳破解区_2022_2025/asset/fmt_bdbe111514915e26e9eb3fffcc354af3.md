
# [CTF] 【2024春节】解题领红包之web题writeup

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1893062`
- 抓取时间: `2026-11-01T22:37:48`

## 【2024春节】解题领红包之web题writeup

一年一度的吾爱春节ctf开赛了

原视频：`https://www.bilibili.com/video/BV1ap421R7VS`

### flag1

来源于视频00:02-00:03几帧，下图圈上的位置白点中间的字就是flag1。

推荐把视频下载下来，使用播放器/剪辑软件逐帧看会更明显。

![](https://attach.52pojie.cn/forum/202402/24/233706tzam7xvxp57d95jd.png)

20240218_2105539657.png (61.01 KB, 下载次数: 4)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0MXxiOTNiNTk4YnwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

flag1{52pj2024}

### flag2

视频00:04-00:07里有四部分被切开的二维码

![](https://attach.52pojie.cn/forum/202402/24/233710pvj0kvh4xjk1vy88.png)

20240218_2105557945.png (48.71 KB, 下载次数: 5)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0Mnw2MDE2MTZjYXwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

全部截图出来，放进PS，使用多边形套索工具圈出图片里的二维码，然后拼到一起。

![](https://attach.52pojie.cn/forum/202402/24/233712qhx2pqxmxpfrxxrs.png)

20240218_2105577218.png (82.13 KB, 下载次数: 4)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0M3w2MDEyZWYyM3wxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

这里发现，二维码定位点是白色，因此需要做一次反色处理

把4个残缺的二维码合并到同一图层，然后点击上方图像-调整-反相

![](https://attach.52pojie.cn/forum/202402/24/233714nom9819rmjiwsw8h.png)

20240218_2105594814.png (115.93 KB, 下载次数: 3)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0NHw1ZWY2NzQ1NHwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

这样就能拿到正常的二维码

![](https://attach.52pojie.cn/forum/202402/24/233717wuq6uadju9u8yy5b.png)

20240218_2106004460.png (81.52 KB, 下载次数: 3)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0NXwzNDNhMDFkZXwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

直接扫码，得到一个网址：`https://2024challenge.52pojie.cn/`

直接尝试打开，发现无法打开。

想到去年的思路，同属.52pojie.cn，应该和主站在同一台服务器。

直接`nslookup` `52pojie.cn`得到主站ip，然后去系统`hosts`内添加一条

```
124.232.185.97 2024challenge.52pojie.cn
```

就可以打开了。

先打开浏览器`f12`，来到网络面板，再输入网址尝试进入。

flag2就在第一个请求的响应头内的`X-Flag2`。

![](https://attach.52pojie.cn/forum/202402/24/233719z49z1ncu1jxu5cxx.png)

20240218_2106024351.png (109.93 KB, 下载次数: 5)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0NnwyNTk4YzM1MnwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

flag2{xHOpRP}

### flag3

视频00:00-00:01是横向移动的雪花，中间有串暂停就看不见的文字，就是flag3.

flag3{GRsgk2}

这段文字自身横向移动方向和背景相反，从而能被人眼识别到。

类似实现的还有这类视频：

`https://www.bilibili.com/video/BV1qF411u7sc/`

`https://www.bilibili.com/video/BV1GM4y1b7JB/`

### flag4

继续看上一步的网络面板，可以看到请求了一个名为`flag4_flag10.png`，直接预览就能看到flag4

![](https://attach.52pojie.cn/forum/202402/24/233722vp1q9kla9tpzitda.png)

20240218_2106047383.png (84.53 KB, 下载次数: 2)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0N3w1MjY3MThjZHwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

flag4{YvJZNS}

可以在这里先把图片保存到本地，为寻找flag10做准备。

这里记录个大坑：最好使用`curl`等工具下载该图片到本地，最新版Edge自带了个“Edge Image Viewer”，其中的“另存为”功能会导致图片丢失信息，无法找到flag10

### flag5

浏览器`f12`切到元素面板，发现了个元素，上方注释了`flag5`和`flag9`

鼠标选中该元素，取消勾选这三个就能在网页上方看到文字（这三个主要作用为将其内部文字颜色设为白色并且使用户无法选中该段文字）

![](https://attach.52pojie.cn/forum/202402/24/233724r34i33bubzvge794.png)

20240218_2106063566.png (17.17 KB, 下载次数: 2)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0OHw3NTZmNGY5OXwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

![](https://attach.52pojie.cn/forum/202402/24/233726dlrl48wre4iv6viq.png)

20240218_2106072667.png (17.11 KB, 下载次数: 3)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE0OXxhYTliMGJkNXwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

粗略一看，有flag四个字母零散放置。

打开文本编辑器，找到l的位置，换行，随后在a的位置换行

可以发现刚好字符数量能对齐，那就按照等字符数量在相同的位置换行（这里一定要使用等宽字体）

![](https://attach.52pojie.cn/forum/202402/24/233728wo1i731o7pczygxc.png)

20240218_2106093691.png (42.79 KB, 下载次数: 3)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE1MHxkMjkwNWU3NHwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

可以发现第一列竖着看刚好是flag5，右侧是flag9的字符画。

flag5{P3prqF}

flag9{KHTALK}

### flag6

直接点击网页内flag6链接打开，检查元素，发现下方有js代码

![](https://attach.52pojie.cn/forum/202402/24/233730pl3ldb4c3q37dbll.png)

20240218_2106115485.png (56.36 KB, 下载次数: 2)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE1MXxmNzhkZDE0MXwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

看下方js逻辑，大致为寻找`md5`是`1c450bbafad15ad87c32831fa1a616fc`的字符串`$str`，然后拼接为形如`flag6{$str}`的字符输出。

打开`cmd5.com`，搜一下这串`md5`就能推出结果。

![](https://attach.52pojie.cn/forum/202402/24/233732pq5rkq35t7uhoqor.png)

20240218_2106139405.png (33.83 KB, 下载次数: 2)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE1Mnw4MWY2YzZiMHwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

flag6{20240217}

（或者，直接点击上方按钮，然后像论坛每次开放注册一样，等几分钟？）

（实测2.17号18:20跑87s可以出结果，不到2分钟）.

### flag7

视频00:21下方出现github仓库网址

`https://github.com/ganlvtech/52pojie-2024-challenge`

![](https://attach.52pojie.cn/forum/202402/24/233735eilf0fdxr0pildqm.png)

20240218_2106158710.png (109.79 KB, 下载次数: 2)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE1M3wxZTcwNTc0NHwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

打开提交历史，查看diff就能找到flag7

![](https://attach.52pojie.cn/forum/202402/24/233737z9y93ok8njwyzzjn.png)

20240218_2106163898.png (34.5 KB, 下载次数: 3)

`https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzE1NHxjMmMyYTZhMnwxNzkzNTcyNjY2fDE2ODA4NnwxODkzMDYy&nothumb=yes`

2024-2-24 23:37 上传

flag7{Djl9NQ}

### flag8

回到`https://2024challenge.52pojie.cn/`

直接进入flag8&flagB页面，可以看到是个2048游戏，flag8需要总计积分（金币）达到10000，不算太多，可以直接玩几局拿到（我大概玩了7分钟）
