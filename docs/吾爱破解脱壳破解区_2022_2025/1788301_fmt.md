
# [Web逆向] 某视频解析及真实视频地址获取

- 原帖：`https://www.52pojie.cn/forum.php?mod=viewthread&tid=1788301`
- 抓取时间：`2026-11-01T22:52:24`

抽点时间更新一下前面的视频解析获取真实链接的帖子。

这次不再发布成品软件了（因为写的有点小问题没时间处理了哈哈~说白了就是菜的一bi）。

网站地址：`aHR0cHM6Ly9qeC54bWZsdi5jb20v`

这个网站具体怎么用的不再过多赘述，他更新的频率还是蛮快的。

我们上来就是`Fiddler`伺候，这样包看起来更好分析一些。包含地址的地方我就直接打码了。

我们有前车之鉴，每次请求他的响应都会伪装成一个`Image`。我们还是挨个分析。

![image.png](https://attach.52pojie.cn/forum/202305/22/102310eyjnlj1awnzam8jy.png)

`image.png`（48.05 KB，下载次数：4）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjU0NnwzMzVjMjY0ZXwxNzkzNTczNTQyfDE2ODA4NnwxNzg4MzAx&nothumb=yes)  
2023-5-22 10:23 上传

在分析到这个地址的时候，我们可以看到，对这个`JS`文件有一个`POST`请求。他的请求参数有视频地址、时间戳、`Origin`和一个`key`（多次请求刷新都没有变化）。他的响应参数有`url`、`next`、`html`这几个加密参数。

![1684722497014.png](https://attach.52pojie.cn/forum/202305/22/102816cmq9f5g9fa5bjaba.png)

`1684722497014.png`（72.49 KB，下载次数：3）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjU0OXw4YTY1NDhkNnwxNzkzNTczNTQyfDE2ODA4NnwxNzg4MzAx&nothumb=yes)  
2023-5-22 10:28 上传

知道了响应参数，我们看看先对这几个加密的参数解密一下看一看。大家可以下一个`XHR`断点挨着跟踪一下，看看他的解密后的参数，这样更直观一些。

既然是加密，那我们必然先全局搜`function decrypt`关键词啊哈哈。找到一个我们在这里下一个断点，刷新，可以看到第一个过来的是一个`next`参数。

![image.png](https://attach.52pojie.cn/forum/202305/22/103555iuuu7dompbm9u5hh.png)

`image.png`（87.57 KB，下载次数：4）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjU1MHxlNGExMmMyOHwxNzkzNTczNTQyfDE2ODA4NnwxNzg4MzAx&nothumb=yes)  
2023-5-22 10:35 上传

让断点卡在这里，然后我们使用控制台尝试对三个参数解密一下试一试。

![image.png](https://attach.52pojie.cn/forum/202305/22/103914btiz3rqzsxwz77it.png)

`image.png`（30.38 KB，下载次数：2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjU1MnxhZTY1N2FkMHwxNzkzNTczNTQyfDE2ODA4NnwxNzg4MzAx&nothumb=yes)  
2023-5-22 10:39 上传

这里我只对`next`和`url`解密了，`next`是我们想看的，他这个`url`返回的也是一个地址，我们回到`Fiddler`中查看这个解密后的`url`地址。

![image.png](https://attach.52pojie.cn/forum/202305/22/104202gotidbctboctnnc4.png)

`image.png`（68.19 KB，下载次数：2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjU1M3xkNGVkZDdjOXwxNzkzNTczNTQyfDE2ODA4NnwxNzg4MzAx&nothumb=yes)  
2023-5-22 10:42 上传

可以看到他返回了一个`M3U8`的集合。多次观察后发现，视频为免费时，他`url`参数返回的是完整视频地址；视频是收费的时候，他返回的就是这个`m3u8`的集合。

（在这里好像是有一个小坑，因为代码是写了好几天了，我有些记不清了。再看代码没有找到，只是提一嘴：如果请求参数没问题的时候，看一看是不是需要校验证书。）

他这个返回的`m3u8`集合在请求的时候有一个`302`重定向，跳转到的才是真正的视频地址。后面的思路就是扣`JS`代码，然后将所有`ts`下载后再组成一个`MP4`就可以了。

希望各位食客给个一键三联。

想要成品的看看评论区会不会有大佬放出来，或者来跟我要我的屎山代码（`C#`的屎山哈哈）。

这是我扣的`JS`。

![zip.gif](https://static.52pojie.cn/static/image/filetype/zip.gif)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNjU1OHwwN2YwMWE5ZXwxNzkzNTczNTQyfDE2ODA4NnwxNzg4MzAx)（39.8 KB，下载次数：132）  
2023-5-22 11:08 上传  
点击文件名下载附件  
下载积分：吾爱币 -1 CB
