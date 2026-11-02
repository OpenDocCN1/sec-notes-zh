
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwMHwwZjBjNjAwN3wxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

## 正则表达式检测

正则表达式的内容如下：

```
/\w+ *\(\) *{\w+ *['|"].+['|"];? *}/
```

就是用这个正则表达式去匹配上面定义的 `XxpyjG` 的函数代码。匹配成功则返回 `true`，否则返回 `false`。

其实如果毕竟懂正则表达式的话，应该可以看出这个表达式就是验证代码是否被格式化过。我这里在控制台验证一下。

![](https://attach.52pojie.cn/forum/202507/31/102441c9svrvl2zkhvlli9.png)

`36.png` (27.53 KB, 下载次数: 249)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwMXw0NTk3ZjRiMXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

调试的时候默认情况下是 `false` 的，但是把格式化代码生成的空格和换行符删掉后再次匹配就变成了 `true`。

## 修改函数定义并清除调试代码

修改好函数定义以后，清除掉所有的 `debugger` 和断点。重新尝试运行脚本，一定要注意这里的函数定义必须把所有的换行和多余的空格（`return` 后面那一个不算）全部清理干净，可以参考我的图片。

![](https://attach.52pojie.cn/forum/202507/31/102444ua6m895zzztrt6gx.png)

`37.png` (145.13 KB, 下载次数: 253)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwMnwzMjNkOTA1MnwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

系统提示有东西未定义，而不是直接卡死，说明刚才那个格式化检测已经过掉了。

此时将 Pycharm 中的 `js` 文件对应修改好后，进行调试。

![](https://attach.52pojie.cn/forum/202507/31/102446s2ejeexmehvsmoce.png)

`38.png` (162.35 KB, 下载次数: 249)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwM3w4MTJkMzQyY3wxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

也是提示未定义，没有问题。继续重复之前的步骤，准备扣代码补环境。

但是此时你再仔细观察一下，会发现报错的这行代码的上面不是有一个已经定义的 `_0x313b78` 吗？怎么会提示还提示没有定义呢。其实在这里是因为前面补的一大长串代码的最后面是一个逗号，我们将其改成语句结束的分号再调试一下。

![](https://attach.52pojie.cn/forum/202507/31/102448lkzfjehhmneyv00h.png)

`39.png` (170.96 KB, 下载次数: 248)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwNHxkNGI0ZDI4OXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

成功通过，但是又报了一个新的无定义。但是这个报错的内容怎么好像找不到？其实是因为上面的字符串被转码后导致的，你可以根据报错的行号和错误提示上面的原代码来找到具体的内容。

找到以后在控制台查看这是个什么东西。

![](https://attach.52pojie.cn/forum/202507/31/102450hfpulz4qg4uqg4o8.png)

`40.png` (19.61 KB, 下载次数: 243)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwNXwyZGU1YzczZHwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

可以看到这是调用的 `_0x5500bb` 里面的 `KNqZI` 函数。

利用之前过格式化检测的方法，将代码放到浏览器中进行调试。

![](https://attach.52pojie.cn/forum/202507/31/102452lply0exp0zdjdm0y.png)

`41.png` (128.62 KB, 下载次数: 251)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwNnwyNzk5NWRiNXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

可以看到函数显示未定义，而且里面调用的函数也和原版中的不一样。

![](https://attach.52pojie.cn/forum/202507/31/102455icd3i39p43scsz33.png)

`42.png` (110.72 KB, 下载次数: 250)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwN3xhYjQ4MjBlOXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

通过比较可以看出，所有的参数都是没问题一致的，所以问题只能出在 `$dbsm_0x42c3` 上面。

![](https://attach.52pojie.cn/forum/202507/31/102457qsjviopvsko3sxs7.png)

`43.png` (134.32 KB, 下载次数: 245)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwOHxmZWUxYTZlNXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

这里可以看到这个函数虽然是已经补过的，但是上面的自执行函数的参数却是这个函数。

其实这里是一个比较常见的混淆套路，一个大数组保存 16 进制的函数名，然后有一个在代码里出现频率极高的解密函数（此处就是 `$dbsm_0x42c3`），除了解密函数以外，还有一个用于将 16 进制字符串还原的移位函数（也就是上面那个参数是解密函数的自执行函数）。

在上次补解密函数的时候，没有补上这个移位函数，所以解密函数解密出来的内容出了错（乱码），自然也就找不到正确的函数，也就会报函数未定义了。

解决办法很简单，只要把移位函数补上去即可。

![](https://attach.52pojie.cn/forum/202507/31/102459hhylikqqilimzpq2.png)

`44.png` (155.54 KB, 下载次数: 252)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUwOXw1YzIzYjIyNHwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:24 上传

成功通过，然后继续开始扣代码补环境。

## 补移位函数与补环境

在补移位函数的时候需要注意下里面还存在格式化检测，有兴趣了解具体原因的可以参考上次过格式化检测的方法来搞定。如果懒得调试可以直接将移位函数压缩，然后再补上即可。

补完移位函数后会有非常多的函数要一个一个补，一定要有耐心，自行参考前面补环境的方法，此处不再详解。

当补到 `_0x1316f4` 的时候你可能会发现调试 `js` 文件时，`node` 会一直执行，其实这里是因为你扣代码的时候把最后的自执行括号给加上去了，实质上这里应该是一个函数。

![](https://attach.52pojie.cn/forum/202507/31/102501r3bsslipnvziisuu.png)

`45.png` (33.88 KB, 下载次数: 251)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUxMHw3ZjJiMzZkOXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:25 上传

在补环境的时候要注意与原版的要保持一致，不清楚时可在 `debugger` 的控制台中查看。

补环境的途中你可能会碰到如下报错：

![](https://attach.52pojie.cn/forum/202507/31/102504fuua8hkmbhho0khz.png)

`46.png` (171.49 KB, 下载次数: 248)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUxMXxlOGI0OTk3Y3wxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:25 上传

提示这个是因为要用到浏览器的属性，而我们用 NodeJs 没有这个环境导致的，只需要在文件开头补上即可，环境代码如下：

```javascript
window = global;window.navigator = {};
```

补到最后，你会发现，程序直接运行结束了。

![](https://attach.52pojie.cn/forum/202507/31/102506ydzq4eqqi8jm44ei.png)

`47.png` (115.76 KB, 下载次数: 251)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUxMnw1Y2RkZjI4NHwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:25 上传

这说明环境已经补得差不多了。在底下 `console.log` 一下看看。

![](https://attach.52pojie.cn/forum/202507/31/102508g7hudcyoah58g0qo.png)

`48.png` (175.11 KB, 下载次数: 253)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUxM3w2MzUxMzVhMXwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:25 上传

又报了一个未定义，但是这个报错的确实 `console.log`，看来是上面的环境代码为了防止调试把 `console.log` 给改掉了。我们在文件的最开头把 `console.log` 保存一下，然后再调用咱们自己保存的来调试。

![](https://attach.52pojie.cn/forum/202507/31/102510alm8mg1449mkl92g.png)

`49.png` (46.94 KB, 下载次数: 254)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzUxNHxlNjgzNDBkMnwxNzkzNTc1NDg5fDE2ODA4NnwxNjM5ODg1&nothumb=yes)

2025-7-31 10:25 上传

成功打印出 `m` 的值。

至此，整个补环境已经完成，接下来只需要调用这个 `js` 就可以获取到 `m` 的值。
