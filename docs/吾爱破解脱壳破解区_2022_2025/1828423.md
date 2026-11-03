
# [Web逆向] 某论坛视频加密M3U8下载

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1828423`
- 抓取时间: `2026-11-01T22:47:19`

## 原因
由于公司需要，目前开始学习`WebGIS`全栈开发，网上资料较少，在论坛求助也没大佬帮助一下：`https://www.52pojie.cn/thread-1828351-1-1.html`(要是有人有`CSDN`的这个“开源架构WebGis解决方案开发指南”资源最好可以分享一下)；有一个是`CSDN`需要付费购买，还一个是超级会员可看，论坛大佬指点了一下就到咸鱼上买了个五天的`CSDN`超级会员，可以解决第二个教程的问题了(虽然第一个会好点)。下面就是下载`CSDN`课程视频的具体方法

## 方法(`fiddle`+`js`抓包+逍遥大佬的`m3u8`下载器)
- `CSDN`课程视频采取`m3u8`加密的方式进行播放，打开`fiddler`，并进入视频播放页面，`fiddler`搜索`m3u8`(下图1、图2)
![](https://attach.52pojie.cn/forum/202309/01/215406tmqu4bb9ztn4sgb4.png)
屏幕截图 2023-09-01 214517.png (114.61 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY0MDE4NnxlZWM2Yzg3OXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDE4NnxlZWM2Yzg3OXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes)
视频播放页
2023-9-1 21:54 上传
![](https://attach.52pojie.cn/forum/202309/01/215720q3qkxudbaxuxik33.png)
屏幕截图 2023-09-01 215657.png (227.85 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY0MDE4N3xjOTczZGYzNHwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDE4N3xjOTczZGYzNHwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes)
fidder抓包`m3u8`显示
2023-9-1 21:57 上传
- 本来以为还需要看看`js`代码，结果发现在`m3u8`下面有个请求是
> `/mp/hls-service/video/api/ali/decrypt`
![](https://attach.52pojie.cn/forum/202309/01/220259r26jji36b7w96882.png)
屏幕截图 2023-09-01 220134.png (566.73 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY0MDE5OXxjZDkyYWU1NXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDE5OXxjZDkyYWU1NXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes)
十六进制
2023-9-1 22:02 上传
- 猜测估计这个就是动态秘钥的请求，`fiddler`切换为十六进制结果查看，如上图圈出的部分即是返回的十六进制数据，选择目标数据后点击保存选中的字节
![](https://attach.52pojie.cn/forum/202309/01/220601gw51hl45p8tpxl0w.png)
屏幕截图 2023-09-01 220456.png (314.44 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY0MDIwMHxmMTUxY2MzOXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDIwMHxmMTUxY2MzOXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes)
2023-9-1 22:06 上传
- 下面就拿出论坛逍遥大佬的`m3u8`下载器(论坛有，自行查找)，先拖动刚刚保存的字节文件到下载器文件位置设置`key`(下图1),然后将第一步的`m3u8`地址复制到链接添加文件，即可成功开始下载了(下图2)。
![](https://attach.52pojie.cn/forum/202309/01/220817ud4d2odzvb1dbdvs.png)
image.png (155.02 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY0MDIwMXwxOTJhMjA4NXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDIwMXwxOTJhMjA4NXwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes)
2023-9-1 22:08 上传
![](https://attach.52pojie.cn/forum/202309/01/220935p407qdvhzevbhddc.png)
image.png (80.97 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY0MDIwMnw1MTdkZTBkMHwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDIwMnw1MTdkZTBkMHwxNzkzNTczMjM3fDE2ODA4NnwxODI4NDIz&nothumb=yes)
2023-9-1 22:09 上传

## 总结
目前来说，在课程视频没加密的`m3u8`视频相对来说比较少的，加密方式也就是固定秘钥或者是动态秘钥，需要观察请求和`js`脚本，求得解密方式即可
