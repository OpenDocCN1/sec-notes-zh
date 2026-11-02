
[href="forum.php?mod=attachment&aid=MjU4NDQ5N3w2MzYyMzBhZXwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NDQ5N3w2MzYyMzBhZXwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes)

17

2023-1-11 21:30 上传

花指令已经清除的差不多了，只需要找剩下的值替换进去就可以拿到`sign`的值了。

在控制台输出`_0x4fe853`、`_0x38facc['ts']`和 `_0xd5726e`的值

![](https://attach.52pojie.cn/forum/202301/11/213344b66h1ljg8x8vvjv8.png)

18.png (8.02 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4NDQ5OHw2MjFhYWY2NHwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NDQ5OHw2MjFhYWY2NHwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes)

18

2023-1-11 21:33 上传

## 第五次代码改写：

![](https://attach.52pojie.cn/forum/202301/11/213530o3vv69424s4xnu9o.png)

19.png (20.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjU4NDQ5OXw0ZjNhZDFlNXwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NDQ5OXw0ZjNhZDFlNXwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes)

19

2023-1-11 21:35 上传

解完混淆发现函数是一个`md5`里面传入一个未定义和两个下划线跟两段数字

将值进行`md5`加密跟参数进行对比，未定义是`token`，因为我们没有登录所以没有`token`，登录之后就不是未定义了，中间那段数字是`ts`是段时间戳，第三段数字是参数`nonce`，而`sign`就是几个参数拼接起码`md5`加密的。

![](https://attach.52pojie.cn/forum/202301/11/214126jagxsgccb9slg9xx.png)

20.png (49.17 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4NDUwMnxkZWZlOGEwMXwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NDUwMnxkZWZlOGEwMXwxNzkzNTc0MTQ5fDE2ODA4NnwxNzM1NDU1&nothumb=yes)

20

2023-1-11 21:41 上传

## 六.结尾

此教程仅供学习交流使用，如有侵犯权益，联系作者删除。
