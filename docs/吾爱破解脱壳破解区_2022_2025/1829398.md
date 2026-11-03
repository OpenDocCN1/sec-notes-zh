
# [Web逆向] 某交易服务平台加密内容逆向

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1829398>
- 抓取时间：2026-11-01T22:47:11

本帖作为自己逆向的记录用，不用于其他任何目的，严禁用于非法用途，网站比较敏感就不发网址了，涉及网址部分已脱敏，大家看看思路就好

## 1. 查看数据接口

先查看数据接口，查看数据是从哪个接口发送的，并查看需要提交哪些数据

![1.png](https://attach.52pojie.cn/forum/202309/05/095113km3oq1zuc8qx77qc.png)

`1.png`（33.42 KB，下载次数：4）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4Mnw5OTNkYjc2OHwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
1.1，2023-9-5 09:51 上传

![2.png](https://attach.52pojie.cn/forum/202309/05/095147km6zo22ezj2i24oo.png)

`2.png`（31.24 KB，下载次数：1）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4M3xhNWZjYzQwMnwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
1.2，2023-9-5 09:51 上传

## 2. `post` 请求与 `Portal-Sign` 参数

`post`一下，发现没有数据返回，查看是否有发送需要携带的加密参数，多次翻页以后，发现有个`Portal-Sign`是需要提交的

![3.png](https://attach.52pojie.cn/forum/202309/05/095204sfqvlhvqt49cq3yt.png)

`3.png`（21.35 KB，下载次数：4）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4NHw2YWQ4ODM5ZHwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
2023-9-5 09:52 上传

## 3. 逆向 `Portal-Sign`

下一步目标明确，先逆向`Portal-Sign`

## 4. 定位 `Portal-Sign` 并下断

通过搜索找到`Portal-Sign`位置，并下断

![4.png](https://attach.52pojie.cn/forum/202309/05/095251dcqaajaq6m0jzz1g.png)

`4.png`（43.9 KB，下载次数：4）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4NXwxM2YxYzBjMHwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
4，2023-9-5 09:52 上传

输出`f.getSign(e)`，查看`Portal-Sign`的值，通过对比发现`f.getSign(e)`就是目标

![4.1.png](https://attach.52pojie.cn/forum/202309/05/095308lwj166u8v54z8ggq.png)

`4.1.png`（34.21 KB，下载次数：1）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4Nnw1ZmI3MjMzZHwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
4.1，2023-9-5 09:53 上传

![4.2.png](https://attach.52pojie.cn/forum/202309/05/095322phhhskklgg1sf0ho.png)

`4.2.png`（25.37 KB，下载次数：1）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4N3wwZjZjY2M1NnwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
4.2，2023-9-5 09:53 上传

## 5. 扣代码生成 `Portal-Sign`

接下来就是扣代码了，都是比较简单的`js`，先扣`f`函数，然后`d`函数，巴拉巴拉，本地得到`Portal-Sign`

![5.1.png](https://attach.52pojie.cn/forum/202309/05/095336sklu92gm94zurk9z.png)

`5.1.png`（15.06 KB，下载次数：3）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4OHwxY2MxYjkxZnwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
5.1，2023-9-5 09:53 上传

![5.2.jpg](https://attach.52pojie.cn/forum/202309/05/095345ceccczoq9ddqef0c.jpg)

`5.2.jpg`（33.18 KB，下载次数：2）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc4OXxkYmEzOWNjNnwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
5.2，2023-9-5 09:53 上传

## 6. 携带 `Portal-Sign` 请求加密数据

带上`Portal-Sign`提交请求，获取加密数据，发现已经可以获取到数据了

![6.png](https://attach.52pojie.cn/forum/202309/05/095408ipn3kd5bdvk4khk3.png)

`6.png`（12.15 KB，下载次数：3）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc5MHxiMDZhMDEwYXwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
6，2023-9-5 09:54 上传

## 7. 逆向解析加密数据

下面开始逆向解析加密数据，既然数据进行了加密，那一定会有解密函数，先搜`decrypt`，看是否有对应内容

![QQ截图20230906175957.jpg](https://attach.52pojie.cn/forum/202309/06/180002mqffq3ye3qq73m2e.jpg)

`QQ截图20230906175957.jpg`（127.16 KB，下载次数：1）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc5MXxiODFlYTRkZXwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
7，2023-9-6 18:00 上传

## 8. 定位解密结果

发现有内容，直接在搜索位置和`return`处下断，多次输出`a.toString(h.a.enc.Utf8)`测试以后，发现该位置就是解密结果

![8.jpg](https://attach.52pojie.cn/forum/202309/05/095446unpi4r9jauzalxaa.jpg)

`8.jpg`（179.52 KB，下载次数：4）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc5Mnw5ZjJkMTZiN3wxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
8，2023-9-5 09:54 上传

## 9. 扣 `b` 函数解密

接下来就比较简单了，由于这个位置位置返回的就是解密结果，直接扣这个`b`函数就可以了，如果懒得扣也能看出来是`aes`加密，用`py`自己改写也可以，我比较懒，直接扣的`b`函数

![9.png](https://attach.52pojie.cn/forum/202309/05/095505dicuxd9wzfbu1dbo.png)

`9.png`（13.78 KB，下载次数：2）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc5M3w4NGU1MzNkYXwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
2023-9-5 09:55 上传

## 10. 得到真实数据

解密以后得到真实数据

![10.png](https://attach.52pojie.cn/forum/202309/05/095520qe1bfb7zbf16bchc.png)

`10.png`（13.99 KB，下载次数：1）  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0MDc5NHxiNGViZDNmYXwxNzkzNTczMjI5fDE2ODA4NnwxODI5Mzk4&nothumb=yes)  
2023-9-5 09:55 上传

## 总结

目前还是`js`逆向小白，正在边学习边练习边记录，逆向过程中一定不要着急，逆向本来就是从网站里找蛛丝马迹，一次不行就两次，多练习，找到加密位置就完成一半了，大家加油吧！
