
# [原创] 一个 vb 写的行业软件逆向破解过程

## 原帖信息

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1696987](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1696987)
- 抓取时间：`2026-11-01T23:08:13`

## 问题背景

今天收到一朋友的朋友发过来一个印花机程序，由于找不到原来的作者，问我能不能 pj。说实在我对这个没经验。我是先浏览一下论坛里对于 vb 程序的内容：[https://www.52pojie.cn](https://www.52pojie.cn)，搜索到“我是用户”的帖子。

因为这个 vb 写的程序也是输入注册码没反应。

![](https://attach.52pojie.cn/forum/202210/08/235001vaupd5hhczmhmb1q.png)

`image.png` (63.7 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE4OHwyMDk0ZTk0MXwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-8 23:50 上传

## 初步定位

按确定 3 次，就会自动退出，没点提示。搜索字符串也没有搜索到什么有价值的东西。

程序肯定是对比输入字符串的长度。我们看了“我是用户”的关于 vb 的帖子，输入 API 断点：`bp __vbaLenBstr`

![](https://attach.52pojie.cn/forum/202210/08/235232p8pilif8v8pllfp8.png)

`image.png` (307.13 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE4OXxmMDAzOGQ3ZXwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-8 23:52 上传

按 F9 运行程序，一路反复追踪。

![](https://attach.52pojie.cn/forum/202210/09/000141sa8hc6acgn8ccy2y.png)

`image.png` (342.99 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE5Mnw5NDIxZDVlNHwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-9 00:01 上传

发现运行这个 `call`，注册窗口就会出现！

## 寻找关键调用

我这里目的只能爆破。研究注册机不是我的能力范围。

要想不出现注册界面，程序运行时，肯定要跟注册过的数据对比。我要让它对比跳过验证就行了！

上翻到段头，发现有 3 个位置调用它。

![](https://attach.52pojie.cn/forum/202210/09/000457nttzlmewntkqlmnn.png)

`image.png` (271.22 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE5M3w1YTNlMjE5YXwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-9 00:04 上传

我就在这 3 个调用地方下断点。

重新运行程序，程序断在这里，我们又去上翻找段头，看下还有哪里调用它。

![](https://attach.52pojie.cn/forum/202210/09/000631leiddil0u1w7t71e.png)

`image.png` (270.08 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE5NHw2NWVhOGRiZnwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-9 00:06 上传

![](https://attach.52pojie.cn/forum/202210/09/000705i96wtd5wabbr3w5n.png)

`image.png` (240.48 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE5NXxmMzkwYzdhM3wxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-9 00:07 上传

## 关键跳转与爆破

好了，这次我们来到关键点。

![](https://attach.52pojie.cn/forum/202210/09/000807vw77zzvtacg7zwwt.png)

`image.png` (283.34 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE5Nnw5YTEwZWU3ZXwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-9 00:08 上传

我把这个 `call` `nod` 掉，运行后还是会出现注册窗口。不过下面这 2 个跳转很可疑。分别尝试把 `jnz` 改为 `je`，每个尝试下。

## 爆破结果

好了，注册后的界面终于出来了。

![](https://attach.52pojie.cn/forum/202210/09/001159q92pv2n61s226wyb.png)

`image.png` (15.27 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTE5N3w4OWM4OWY2ZXwxNzkzNTc0NDkxfDE2ODA4NnwxNjk2OTg3&nothumb=yes)

2022-10-9 00:11 上传
