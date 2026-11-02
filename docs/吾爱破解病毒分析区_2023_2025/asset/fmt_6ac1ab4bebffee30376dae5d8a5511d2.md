
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY2NXw1MzQ2YjVjOXwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:42 上传

恶意程序释放恶意拓展与恶意模块部分过程，如下图所示：

![](https://attach.52pojie.cn/forum/202305/27/124313n1muamaap4umyrrv.png)

捕获9.PNG (142.87 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY2NnwxMDYyYjJjNHwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:43 上传

![](https://attach.52pojie.cn/forum/202305/27/124322gsas1s1dd3mfxzd5.png)

捕获10.PNG (151.81 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY2N3w4MWIyNDNiM3wxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:43 上传

![](https://attach.52pojie.cn/forum/202305/27/124329u84q13x00iio90s3.png)

捕获22.PNG (62.33 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY2OHxlZDEyMTFiZXwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:43 上传

该样本还存在复制`cmd.exe`执行行为的行为，尚不清楚作用是什么，存在较高的安全风险与安全隐患，如下图所示：

![](https://attach.52pojie.cn/forum/202305/27/124345jxbm7bj6nfsj48r8.jpg)

IMG_0455.jpg (262.42 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY2OXw1ZDZiYjc2M3wxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:43 上传

样本检出率仅1家，如下图所示：

![](https://attach.52pojie.cn/forum/202305/27/124402v7w0za5p775e7wsz.png)

捕获23.PNG (58.46 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3MHwxZjFjZjBjZHwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:44 上传

我们发现了多个同源样本，如下图所示：

![](https://attach.52pojie.cn/forum/202305/27/124450j82kc28rkk772qkb.jpg)

IMG_0456.jpg (251.31 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3MXxkNDk1YWUyYnwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:44 上传

该家族样本使用多个恶意拓展与恶意模块，其网站对调试器进行干扰与欺骗，强制锁定用户浏览器主页、标签页、搜索结果页，用户很难手动改回，且对用户的其他浏览器进行同样的操作，已严重越界，手法与木马病毒无异。

## 三、搜索引擎随机取样

既然该网站收录于Bing必应搜索引擎，我们在该搜索引擎当中随机取样。当搜索“谷歌浏览器”为关键词时，页面顶端出现了许多广告，我们取其中某个广告（bb[.]yiliwl[.]top）与在广告下面的第一个搜索结果对象（chrome[.]xahuapu[.]net）进行下载，如下图所示：

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3Mnw3ZGQzNDBmN3wxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:45 上传

![](https://attach.52pojie.cn/forum/202305/27/124508fnufv6efyx68vlhv.png)

捕获34.PNG (179.33 KB, 下载次数: 1)

这两个网站域名下载得到的文件 Hash 完全一致

样本文件数字签名，厦门腾瑞科技有限公司，如下图所示：

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3M3xmYTU2NjAyM3wxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:45 上传

![](https://attach.52pojie.cn/forum/202305/27/124524l2vztsg5e1u2q9gd.png)

捕获35.PNG (27.86 KB, 下载次数: 0)

其安装完成后的 Chromium 内核浏览器（非Google Chrome）同样被捆绑了主页、书签和广告图标，如下图所示：

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3NHxkZGVhOTNkZHwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:45 上传

![](https://attach.52pojie.cn/forum/202305/27/124547cj3xkz309bjyrxxk.png)

捕获37.PNG (208.46 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3NXxmZWY3NTUzZnwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:45 上传

![](https://attach.52pojie.cn/forum/202305/27/124555gyi9n6yi5uq1jt50.png)

捕获36.PNG (40.57 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzY3NnwyMjQ4ZGExNXwxNzkzNTc2OTgxfDE2ODA4NnwxNzkwNDE1&nothumb=yes)

2023-5-27 12:46 上传

![](https://attach.52pojie.cn/forum/202305/27/124602r204dzh1dye50vc5.png)

捕获38.PNG (40.24 KB, 下载次数: 1)

不过可以肯定的是，该样本首页和标签用户可以手动修改回来，且并未使用恶意拓展的方式，不会涉及用户的其他浏览器。

## 四、总结

浏览器已离不开我们的生活，许多用户喜欢去各大搜索引擎下载浏览器，下载时用户务必要擦亮眼睛，避免造成不必要的麻烦，给他人暗刷广告流量。
