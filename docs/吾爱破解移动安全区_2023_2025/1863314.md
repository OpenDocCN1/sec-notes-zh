
# [Android 原创] 某鹅痛APP课程逆向m3u8与key解密下载，因被忽悠才研究！

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1863314`
- 抓取时间: `2026-11-01T21:36:36`

## 前言

昨天买了个C语言课程，买的时候说是可以PC观看，买了又不让PC看.....于是开启了对抗之路！仅提供思路，限学习观看，禁止任何非法行为！

## 教程开始

1. 打开某鹅痛app，播放视频前开始抓包，这里采用某鸟抓包(需要`root`安装证书，建议苹果手机抓包)，然后播放视频，视频播放出画面后停止抓包！

2. 开始分析抓包数据，抓包软件搜索`m3u8`，找到响应带`ts`分片地址的`m3u8`数据包，将`m3u8`链接复制下来！

![](https://attach.52pojie.cn/forum/202311/29/161326mzecelehhzvk8tch.png)

1.png (112.22 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY2MDQ0M3xjMTMzMWNjOXwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDQ0M3xjMTMzMWNjOXwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes)

2023-11-29 16:13 上传

3. 将链接丢到`m3u8`下载器发现下载`ts`一直重试，然后下载`m3u8`文件与抓包播放时的`ts`对比分析，发现`m3u8`文件里面的`ts`地址缺少三个参数(3个参数在接口里面也有，适合写解析使用接口方式)！

![](https://attach.52pojie.cn/forum/202311/29/161856wpzeawua0htczexe.png)

2.png (19.35 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY2MDQ0NHxjYWJmYzllMnwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDQ0NHxjYWJmYzllMnwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes)

2023-11-29 16:18 上传

这时就只能重写`m3u8`文件，将`ts`链接都完全还原！

```
抓包的ts地址：https://cdn域名脱敏/9764a7a5vodtransgzp1252524126/2d27600b3270835009582907648/drm/v.f421220_0.ts?start=0&end=120143&type=mpegts&sign=63d9ccff615c61cd9c13cab46b4c579d&t=65679a03&us=WUqZnQoqta
原m3u8的ts地址：v.f421220_0.ts?start=0&end=120143&type=mpegts
```

参考示例：

![](https://attach.52pojie.cn/forum/202311/29/162733a6w9wa0p0l9vtzm5.png)

3.png (227.91 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY2MDQ1MnxkYTIwMjc1OHwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDQ1MnxkYTIwMjc1OHwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes)

2023-11-29 16:27 上传

这时将`m3u8`文件拖到`m3u8`下载器下载，发现`ts`解密失败，尝试把`m3u8`文件内`key`地址放到浏览器打开，发现提示

```
Key: 'VodPrivateKey.Uid' Error:Field validation for 'Uid' failed on the 'required' tag
```

4. 这时继续分析抓包的数据，将`key`链接上的`mid`去搜索，发现`https://域名脱敏/app/xe.vod.privatekey.get/1.0.0`接口返回了一个密文，于是直接`hook` app解密，这里复杂，省略步骤！发现是`AES`，解密接口密文后的数据前32位即`m3u8`的16进制`key`！文末提供获取`key`工具，仅限学习使用！

5. 将`key`与`m3u8`文件放到下载器开启愉快的下载，对抗结束，欢迎评论区讨论更优质的解决方案！解决了某课程宣传`PC`可看，实际不让我`PC`观看的套路！！！

## 软件工具

KEY工具的截图：

![](https://attach.52pojie.cn/forum/202311/29/173114hnq9pz8hn74k8aa9.png)

5.png (45.36 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY2MDUxMnxjZGE3YmY4OXwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDUxMnxjZGE3YmY4OXwxNzkzNTY4OTk0fDE2ODA4NnwxODYzMzE0&nothumb=yes)

2023-11-29 17:31 上传

由于我`python`库安装太多，所以打包的包较大，不想折腾了，有能力的自行逆向解决，勿喷！

`m3u8`下载器自行百度或者参考我以前文章！

`key`工具链接：`https://wwyh.lanzouu.com/ipR0X1gbcj2h`
