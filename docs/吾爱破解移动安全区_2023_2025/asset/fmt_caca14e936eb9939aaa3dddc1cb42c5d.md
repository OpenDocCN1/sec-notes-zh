
# [Android 原创] 小程序游戏破解括弧新人之路，最详解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1822624
- 抓取时间: 2026-11-01T21:40:53

## 前言

这是一篇不够成熟的文章，文章内有疑点错误之处，烦请提出改正！

前几天我接触了一款游戏括弧商业都市，处处都是氪金逼氪，我便想到能不能用科技打败魔法

便开展了为期3天的学习之路，无奈网上大多为20/21年教程，相关信息略微落后

且对于我这种傻瓜堪比为天书

于是我决定写一篇让我还未出生的傻儿子都能看懂的教程

## 抓包软件介绍

首先我们先来介绍一下抓包软件 `fiddler` 括弧汉化版，对新手不要太友好

对就长这个样子，放在附件里面了

![](https://attach.52pojie.cn/forum/202308/18/093844qgaa62s8636wmmg4.png)

`6b725c2b211a79e7bf036aa0e959e0f.png` (170.75 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzNjQ1MXw3NjRmMjgxMHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ1MXw3NjRmMjgxMHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:38 上传

## 配置 fiddler

我们给它配置一下，从左往右数找到第四个工具，找到选项点进去

![](https://attach.52pojie.cn/forum/202308/18/094302ht1a30brr07mamab.png)

`2918b441c8d0f22b7fe28249788d80f.png` (67.99 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzNjQ1MnwyNmIyMDBkM3wxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ1MnwyNmIyMDBkM3wxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:43 上传

![](https://attach.52pojie.cn/forum/202308/18/094306vbbw5cxw5axj8wdx.png)

`5b86319b648f5b0a6475a17ef3a16ca.png` (88.05 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzNjQ1NHxjOGI2YWQ1NXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ1NHxjOGI2YWQ1NXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:43 上传

在第二个图 `https` 页面，依次点动作，信任根证书

## 实战抓包

接下来我们先以没加密的游戏神奇金铲子为例

![](https://attach.52pojie.cn/forum/202308/18/094647t2k5xv2awwwyr7tx.png)

`09c753b510ebc2811c7f530b9b2bac0.png` (202.86 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ1OXxhYzkzODhjMHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ1OXxhYzkzODhjMHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:46 上传

`fiddler` 页面 `f12` 开始捕获通信，运行神奇金铲子，抓包如图

![](https://attach.52pojie.cn/forum/202308/18/094948eyln06aotgg0g0v7.png)

`b028db72dcaf3dc8705a6a77f87d869.png` (769.57 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ2M3w2OGM4NTNiNXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2M3w2OGM4NTNiNXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:49 上传

我们要找哪个是游戏的服务器，多玩两把不难发现，获得金币之后小程序都会向服务器发送一个包如图

![](https://attach.52pojie.cn/forum/202308/18/095422szlx5kpd5xxzoqeu.png)

`8dd3e2ce5ae1cfda06885647dab5b91.png` (162.42 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ2NHwyZjE0NTE5YXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2NHwyZjE0NTE5YXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:54 上传

## 解析原始数据

我们点进去，点击原始，用记事本查看

![](https://attach.52pojie.cn/forum/202308/18/095531ig4gn3gavg6dga6g.png)

`97b368c4dcfc7e004ae337e9b48b3dc.png` (361.27 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ2NXwxOWIzZDgzZHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2NXwxOWIzZDgzZHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:55 上传

最上方蓝色字体便是收包的地址，而记事本里有 `token`，`appid` 等一些信息

我们再往下看，第一个便是你铲子的等级，第二个便是你的金币数

很开心没有加密，都是明文

![](https://attach.52pojie.cn/forum/202308/18/095922zxti2zb2hhhva69z.png)

`bcabe93b353de0df2ea399cfd3d23e4.png` (101.63 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ2NnwyYTZkNDMzNXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2NnwyYTZkNDMzNXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 09:59 上传

## 使用 postman 发包

接下来就到了发包的地方了，我们要用到 `postman` 这个软件

这个官方正版就挺好用的直接给链接吧 `https://www.postman.com/downloads/`

目前我们的目的是发送一个包（内含你想要的东西）给服务器

服务器地址就是蓝色字体收包的地址 `https://game-api.feigo.fun/api/user/set-game-user-value`

配置一下 `postman`，框框里面填的就是服务器的地址

![](https://attach.52pojie.cn/forum/202308/18/100709ytxblyz1clyymwj1.png)

`31987afaa033e194f174378f011c464.png` (200.53 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ2N3w2ZTk0OTVlOHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2N3w2ZTk0OTVlOHwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 10:07 上传

这个框框里面填的就是你要发的包，其他的地方按照图片里面设置一下

![](https://attach.52pojie.cn/forum/202308/18/100712m8meofiw2wj0ujxk.png)

`3eed053bfd2e371f140403fff798026.png` (201.31 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYzNjQ2OHw5Zjc2MmQ4MXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2OHw5Zjc2MmQ4MXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 10:07 上传

而我们要发的东西是啥呢，便是抓包软件里面的 `json` 那一段，也是记事本里面我画框框那一段

![](https://attach.52pojie.cn/forum/202308/18/101015u63w99301e12e0qc.png)

`c0cdc5f3cf44564f30e68f60717b4c8.png` (126.58 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzNjQ2OXxiN2YwMDRkNXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ2OXxiN2YwMDRkNXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 10:10 上传

![](https://attach.52pojie.cn/forum/202308/18/101018pmspp4hpcppkzmkl.png)

`a4b989189bdaa128f140ef630a0107e.png` (103.27 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzNjQ3MHxmODY0YWJhMXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ3MHxmODY0YWJhMXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 10:10 上传

发送之后提示你

![](https://attach.52pojie.cn/forum/202308/18/101142jzii87o8xhiru77i.png)

`981e54b5755d08bc5d51b47cb1046fb.png` (11.07 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYzNjQ3MXw2ZTQyZWQ5MXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzNjQ3MXw2ZTQyZWQ5MXwxNzkzNTY5MjUxfDE2ODA4NnwxODIyNjI0&nothumb=yes)

2023-8-18 10:11 上传

我也是查了半天才明白，这二笔小游戏有个验证登陆状态的需求，以保证只有指定用户在登录状态下才能访问对应数据，也可以用于防止表单重复提交

怎么办呢 csdn上有个博主给了相应的办法也很简单，附上链接自己看吧

`https://blog.csdn.net/qq_43233225/article/details/127429751`

（出处csdn）

如此便大功告成了，这也是最简单的小程序，没有加密

站里也有个跟我类似的帖子

`https://www.52pojie.cn/thread-828363-1-1.html`

可以两个一起参考着来

## 参考资料

对于解加密的方法和相关软件马上也会发帖出来，下面站里面相关反编译和解加密的方法

相关软件有些过时我用反正是坏掉了，可以先了解一下思想

反编译微信小程序获取小程序前端源码 `wxapkg`

`https://www.52pojie.cn/thread-1473679-1-1.html`

(出处: 吾爱[//www.52pojie.cn" target="_blank" class="relatedlink"](https://www.52pojie.cn:href="https)论坛)

Wechat小游戏逆向修改数据——以【消灭病毒】为案例

`https://www.52pojie.cn/thread-974827-1-1.html`

(出处: [//www.52pojie.cn" target="_blank" class="relatedlink"](https://www.52pojie.cn:href="https)论坛)

微信小游戏破解（一）：我要当皇上满级修改

`https://www.52pojie.cn/thread-828363-1-1.html`

(出处: 吾爱破解论坛)

微信小游戏（二）：我要猫咪满级修改

`https://www.52pojie.cn/thread-832333-1-1.html`

(出处: 吾爱破解论坛)

## 下载链接

写到这里也就结束啦，相关软件放在了附件里面，免费评分走一个走一个

附件太大了传不上去，我还是发链接吧

链接：`https://pan.quark.cn/s/64e2d7235959`

提取码：`cYng`

链接：`https://pan.baidu.com/s/14WStkdhcr4b-8d0iqugShg?pwd=qasw`

提取码：`qasw`

免费评分走一个走一个

我也是近几天才接触这些的，有诸多错误和不解之处，希望大佬能指导一二
