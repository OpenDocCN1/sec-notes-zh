
# [Web逆向] 搭建属于自己的某多多视频解析接口

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1762470>
- 抓取时间：2026-11-01T22:58:20

最近要用到一些pdd商品页的视频，找了一些手机上能用解析的基本都是收费的，于是自己动手搭了一个解析接口，跟大家分享一下搭建过程。

## 1. 抓包分析

浏览器访问任意一个有视频的多多商品链接，跳转到了登录页面，`F12` 开始分析。

![](https://pic.imgdb.cn/item/6419389fa682492fccdfcc23.png)

登录账号后，可以正常查看商品页面，点击商品页面中的播放按钮，播放视频，网络选项卡里面直接就能找到明文的视频地址。

![](https://pic.imgdb.cn/item/641938e6a682492fcce03b76.png)

搜索一下来源

![](https://pic.imgdb.cn/item/6419398aa682492fcce14b47.png)

![](https://pic.imgdb.cn/item/641939aca682492fcce189d9.png)

发现商品返回文本里面的有一个专门存放商品信息的 `json`，`sku`/图片/价格等信息都在里面，将找到的地址复制出来，直接访问一下，发现视频能正常下载。

流程总结：登录账号获取 `cookie` ---- 带 `cookie` 访问目标商品 ---- 返回文本中找到视频地址

## 2. 接口实现

上一步已经分析完整个过程，接下来就是用自己熟悉的语言实现一下就好，关于第一步【登录取`cookie`】，我使用最简单的模拟操作，模拟人工在浏览器里面登录。比如可以用 `python` 里面的各个浏览器 `dirver` 来模拟，我使用的是易语言里面的 `Web浏览器`，道理都是一样的。

这里我用了 `cchttp` 模块搭建接口请求

![](https://pic.imgdb.cn/item/64193eeca682492fccea33e1.png)

搭建好后测试一下接口，因为用的是 `get` 请求，`?` 后面接商品的分享链接就可以了。

`http://127.0.0.1:8080/getVideo?https://mobile.yangkeduo.com/xxxxxxxxxxxxxx`

![](https://pic.imgdb.cn/item/64193eeca682492fccea33f4.png)

可以成功的取出视频地址了，收工。

最后附上源码，用到的模块就不放了，自己找一下。

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[`forum.php?mod=attachment&aid=MjYwMDE5NnxlMDIxOWNiNHwxNzkzNTczODk3fDE2ODA4NnwxNzYyNDcw`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMDE5NnxlMDIxOWNiNHwxNzkzNTczODk3fDE2ODA4NnwxNzYyNDcw)

(320.66 KB, 下载次数: 402)

2023-3-21 13:29 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB
