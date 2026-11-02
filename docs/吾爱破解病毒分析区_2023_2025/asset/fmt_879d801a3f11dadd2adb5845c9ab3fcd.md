
![](https://attach.52pojie.cn/forum/202403/14/084114osy2oqusksqbby2m.png)
20240311131454.png (659.29 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY4MTQ1MHwzZTE5MmY1OHwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MTQ1MHwzZTE5MmY1OHwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes)
2024-3-14 08:41 上传

接下里还用说吗？赶紧把这些函数打上断点呀。

看到这里大头基本上已经分析的差不多，下面展示一些细节；

## 反沙箱
继续分析会看到一个明显的反沙箱操作；之前再历史文章中也总结过过这种方法；
![](https://attach.52pojie.cn/forum/202403/14/084117uzp6r0vvz7ezppv8.png)
20240311131915.png (134.75 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY4MTQ1MXwzYThmZjI0Y3wxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MTQ1MXwzYThmZjI0Y3wxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes)
2024-3-14 08:41 上传
创建一个线程 `sleep`  83s，然后使用`waitForSingleObject`等待这个线程返回，等待时间是80s，如果没有超时说明`sleep`函数被加速了，那就是沙箱。

## 反调试
反调试还是利用`waitForSingleObject` , 等待线程的返回；不过这次是等待0s，如果没返回 `v38`就加一；
![](https://attach.52pojie.cn/forum/202403/14/084120c6pdc3p2qr6nr6d6.png)
20240311132445.png (454.76 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4MTQ1Mnw1MWIwMjUzZXwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MTQ1Mnw1MWIwMjUzZXwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes)
2024-3-14 08:41 上传
只有的等`v38 > 0x26000` 的时候才会执行恶意代码；
如果是单步调试的话，`v38`可能不会大于 `0x26000`; 同时这里也是一个对系统性能的判断，如果性能比较差也不会> `0x26000`.

## c2地址
继续更跟踪会发现IP地址解码函数；
![](https://attach.52pojie.cn/forum/202403/14/084122ozqft02fet5phf7p.png)
20240311133242.png (258.91 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY4MTQ1M3xiMjcxNTRmMXwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MTQ1M3xiMjcxNTRmMXwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes)
2024-3-14 08:41 上传
继续跟踪发现使用的User-Agent;
![](https://attach.52pojie.cn/forum/202403/14/084125awx3xcfw9abfwy7w.png)
20240311133414.png (418.02 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY4MTQ1NHwzZDE4ZTkxN3wxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MTQ1NHwzZDE4ZTkxN3wxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes)
2024-3-14 08:41 上传
最后找到请求的url地址：
![](https://attach.52pojie.cn/forum/202403/14/084128i9dugjtridd282uq.png)
20240311134326.png (246.96 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY4MTQ1NXw4Yjc5YTFhOXwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MTQ1NXw4Yjc5YTFhOXwxNzkzNTc2OTAyfDE2ODA4NnwxOTAwODUy&nothumb=yes)
2024-3-14 08:41 上传
[443/kumoko/wakaba.bmp" target="_blank" rel="noopener noreferrer nofollow"](https://39.xx.xx.xx:443/kumoko/wakaba.bmp:rel="nofollow noopener" href="https://39.xx.xx.xx)

## 总结
此二进制用的技术方法；
- 直接patch原始二进制，保留原文件的字符串、统计特征、资源等，防止被杀软识别
- 时间流速反沙箱
- 反调试
- 垃圾代码、虚假控制流混淆视听
- 独立编写的shellcode，防止被识别
- 混淆比较小儿科，分析起来比较简单

## 参考
- 被修改文件：e20d0311a9e436c08ecb72c349903b22
