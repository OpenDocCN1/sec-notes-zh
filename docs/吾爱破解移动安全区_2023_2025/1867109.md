
# [Android 原创] 有谁不喜欢剑仙呢 存档解密 (Unity IL2Cpp 新手向)

## 原帖信息

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1867109](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1867109)
- 抓取时间：2026-11-01T21:36:14

## 游戏与存档位置

最近沉迷修仙挂机小游戏，尤其是那种本地保存，带有云存档的。

游戏本身可以用 GG 修改器无脑改值(死刑！)，作者也停更了很久，自带的云存档也停用了，刚好作为新手练习作品。

游戏地址：`aHR0cHM6Ly93d3cudGFwdGFwLmNuL2FwcC8zNTMwOTQ=`

打开游戏，释放资源，确定存档位置为 `/data/data/[app]/files/sdvsgeah2`。

## 存档文件内容

文件内容：

![图片](https://attach.52pojie.cn/forum/202312/09/141123r3exu8m5dhrdjhor.png)

`ANJOK{S5U@RLB$U)~@V%X_Y.png` (20.14 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY2MnxjMzcxZjIwY3wxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:11 上传

引号中是双等号结尾，判断为 `base64+`，尝试一遍后没有解出明文，此时有两个想法，其中还嵌套着一层加密，或者使用的是作者自定义的数据结构。

## 初步判断

只能进 `so` 文件分析了。

`app` 没有加密，`dump` 的很顺利，使用 `il2cppdumper` 输入 `apk` 包中的 `assets\bin\Data\Managed\Metadata\global-metadata.dat` 和 `lib\arm64-v8a\libil2cpp.so` 后提取。

![图片](https://attach.52pojie.cn/forum/202312/09/144627owtdzdq3a8tjrhf7.png)

``)J`IE5O58PN4VC_~7WGS9OP.png`` (46.02 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3Mnw0MGVmZWMxOXwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:46 上传

`IDA` 打开 `libil2cpp.so`，加载完后 `File -> Script File`，选择提取的 `ida_py3.py` -> `stringliteral.json`。

加载完后选择 `ida_with_struct_py3.py` -> `script.json`、`il2cpp.h`。到此准备工作完成。

## 代码定位

开始代码定位，在 `stringliteral.json` 中搜索 `sdvsgeah2`。记录地址 `0x14ABAF8`。

![图片](https://attach.52pojie.cn/forum/202312/09/145044mhm4gftfmjtehryh.png)

`B~%8NK}AS7UKJFJD8VUNSU.png` (45.15 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3M3w0ZDNlOGVlNHwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:50 上传

在 `IDA View` 按 `G` 或跳转->跳转道指定位置，输入记录的地址。

![图片](https://attach.52pojie.cn/forum/202312/09/145303md8jj8gdnn7n0gd7.png)

`ISG~CUK~9${_C_SA5~JUY$B.png` (139.46 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3NHxlODMwMDZiN3wxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:53 上传

## 交叉引用与关键函数

对应的为 `StringLiteral_7528`，在 `DataManager.load` 函数中被调用。

右键字符串 `StringLiteral_7528`，选择交叉引用列表。

![图片](https://attach.52pojie.cn/forum/202312/09/145514qfqjr7igr7ra8r7y.png)

`6$P$$E)7VB(Z9N20]5EL~2B.png` (171.78 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3NXwzNzYxYWIzM3wxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:55 上传

其中 `DataManager.load` 和 `DataManager.save` 中使用到了此字符串，为了解密存档是如何生成的，双金 `save` 跳转到对应函数。

![图片](https://attach.52pojie.cn/forum/202312/09/145716jud7gz3p7fwb3qdd.png)

`~W)[7@5U9L3VG_8O]_$Y_AI.png` (20.96 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3Nnw3NjYzZWI3MXwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:57 上传

![图片](https://attach.52pojie.cn/forum/202312/09/145756jt3raw3mzcc1q916.png)

``}2JL844E3MS[DO413`B17LG.png`` (133.01 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3N3w0ZTBhYTU3ZnwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:57 上传

按 `F5` 查看伪代码。

![图片](https://attach.52pojie.cn/forum/202312/09/145848ny8n41ixidrr3ryb.png)

`M)RLU9ZFHUQ(D17PG}X9%GG.png` (98.72 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3OHw1NmMxZTA3NXwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 14:58 上传

## 伪代码分析

关键代码在 `DataManager.getSaveContent`，其中返回的 `SaveContent` 就是加密后的密文。双击跟踪。

![图片](https://attach.52pojie.cn/forum/202312/09/150020uaz6t1p616hxgshd.png)

`7PC19(4G0W46ABP53Y1Z1%A.png` (122.4 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY3OXwzMTA1ZWIyZnwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:00 上传

前面的都是如何序列化角色信息，关键在于最后一行代码，很明显使用的是 aes 加密字符串 `v10`。

因为我过于小白，不会下断点啥的拦截到 `v10` 变量的信息进行动态调试，继续静态分析。

![图片](https://attach.52pojie.cn/forum/202312/09/150332ps27xt2bpoyb7ed6.png)

`DWX)QWV$MV43WVW8CZVMGT6.png` (42.38 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY4MHw3Yzg2ZWM2ZnwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:03 上传

## AES 参数分析

`v10` 是明文，`StringLiteral_5936` 为 `key`，还有个我不清楚的 `v11` 为 `methodinfo`。

双击 `StringLiteral_5936` 根据获取 `key` 为 `cdd******527191`。

![图片](https://attach.52pojie.cn/forum/202312/09/150617xphpmukgtghikpsp.png)

`屏幕截图 2023-12-09 150537.png` (151.6 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY4MXw3YzU3YjkyMXwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:06 上传

尝试脚本解密，别问我为什么是 CBC，我也不知道 C# 里面的 aes 默认是啥，当时全部都试了一遍。

![图片](https://attach.52pojie.cn/forum/202312/09/151008ph78khzxhvkkvqjw.png)

`D8B_EQPR9]0[3RS%}KK8EIQ.png` (86.52 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY4Mnw1YTM0MDE5NXwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:10 上传

嗯。。。能看到明文了，但是头部存在乱码，后续查找资料了解到，AES 为 CBC 模式时，除了 `key` 外还存在 `iv`，只有 `key` 和 `iv` 全部相同时才能成功解密。双击 `CRYPTO_Crypto_aesEncode` 跟进。

![图片](https://attach.52pojie.cn/forum/202312/09/151620op4n8vbpkch4877z.png)

`W8Z%FYTIVSOLLZ$XBDSY}HY.png` (125.84 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY4M3xkNjE1OGRlOXwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:16 上传

函数体头部存在一个类似初始化的操作，只会执行一次，其中在 `v9` 对应的 `StringLiteral_5714` 中找到 `iv`。

![图片](https://attach.52pojie.cn/forum/202312/09/151749ggobrrotb0ctto3c.png)

`P{2I@~S48ETQD@6(WPK3QSH.png` (148.35 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY4NHw5Nzc4MTU5MHwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:17 上传

## 解密脚本与总结

最后脚本如下，成功解密存档，加密方式为先 AES 后套 base64，都是对称加密，解密后修改数据后可重新加密使用，到此这个游戏彻底被玩坏了。

![图片](https://attach.52pojie.cn/forum/202312/09/152042tmf3zailimqcmdeq.png)

`屏幕截图 2023-12-09 152036.png` (92.43 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MjY4NXwwZDY0ZTNhMHwxNzkzNTY4OTcyfDE2ODA4NnwxODY3MTA5&nothumb=yes)

2023-12-9 15:20 上传
