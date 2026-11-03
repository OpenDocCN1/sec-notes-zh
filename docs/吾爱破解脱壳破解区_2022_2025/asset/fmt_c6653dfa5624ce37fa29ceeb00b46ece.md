
# [Web逆向] 某酒店登录系统滑块逆向分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2049410
- 抓取时间: 2026-11-01T22:06:19

本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关.本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责

某酒店登录系统滑块，难度适中，食用地址：

aHR0cHM6Ly9ob3RlbC5vY3l1YW4uY29tL2xvZ2lu

密码学参考文章：

RSA密钥格式解析&转换------aHR0cHM6Ly9tcC53ZWl4aW4ucXEuY29tL3MvcHcycEFEYU1QaWJpTlJvQ3dhOUlzZz9zY2VuZT0x

加密的诗篇：RSA与Padding技术的艺术融合------aHR0cHM6Ly9tcC53ZWl4aW4ucXEuY29tL3MvaU0xS1lldy13R1VORXBwWk1YTG5vdw==

【密码学】分组密码模式------aHR0cHM6Ly9tcC53ZWl4aW4ucXEuY29tL3MvNzZjNXNyWVZrakdIS203MmN4TU1jUQ==

分析`crypto-js`当中`AES`的实现----aHR0cHM6Ly9tcC53ZWl4aW4ucXEuY29tL3MvRlBjUzlvRUQ5UHFDOV9nLXpmTEhqUQ==

本文主要是对滑块的思路进行分析讲解，分析`ast`还原以及滑块加密逻辑

![](https://attach.52pojie.cn/forum/202507/29/085043wbwkcvzqx7cvvlqv.png)

image.png (173.29 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5Mjc4MHwwZjkwMDg4YnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc4MHwwZjkwMDg4YnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

滑块图片

2025-7-29 08:50 上传

## 一、流程分析

完成一次完整的滑块滑动过程，确定需要解决的加密参数

![](https://attach.52pojie.cn/forum/202507/29/085012a6pj6f6d1jrjq6gn.png)

image.png (116.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5Mjc3OXxjZWRmZDQzNHwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc3OXxjZWRmZDQzNHwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

流程接口

2025-7-29 08:50 上传

按照整体流程，我们先从两个前置接口开始分析

![](https://attach.52pojie.cn/forum/202507/29/085205fip027auogr4019r.png)

image.png (49.25 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5Mjc4Mnw5NWRlMGI3OXwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc4Mnw5NWRlMGI3OXwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

前置载荷

2025-7-29 08:52 上传

![](https://attach.52pojie.cn/forum/202507/29/085257fz4uulokooorgl0z.png)

image.png (78.78 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5Mjc4M3w0NzEwZmM5M3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc4M3w0NzEwZmM5M3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

第一前置返回值

2025-7-29 08:52 上传

目前看这里主要就是返回一些请求信息，对当前状态的信息存储，载荷中`user_id`参数疑是`uuid`，可以借助python的`uuid`库调用实现。接下来看看第二个请求和第一个请求的关系

![](https://attach.52pojie.cn/forum/202507/29/085548f2s8nwqzoob2zq2p.png)

image.png (53.92 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5Mjc4NHwyYzZhNTQyZnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc4NHwyYzZhNTQyZnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

第二次请求载荷

2025-7-29 08:55 上传

在这里我们发现`challenge`参数和第一个请求返回的一致的，说明第一次请求返回值会继续在第二次接口中使用。

![](https://attach.52pojie.cn/forum/202507/29/085827gd79af8iyyxfi0wx.png)

image.png (110.06 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5Mjc4OHw3ZDg1MDlhZXwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc4OHw3ZDg1MDlhZXwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

第二前置接口返回值

2025-7-29 08:58 上传

这里的返回值就有很多参数信息，有背景图，缺口图，还有一个大数组，还有一个很重要的关键词`publicKey`熟悉的人一看就会猜加密会用到`RSA`（不了解的可以参考密码学文章）。这样发现前置接口没有过多的需要加密的点。

我们继续看看大数组的用处

![](https://attach.52pojie.cn/forum/202507/29/090126yd7o977u9uurr6t3.png)

image.png (229.04 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5Mjc4OXwyNjkxZTk3M3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc4OXwyNjkxZTk3M3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

图片切块

2025-7-29 09:01 上传

查看图片内容，发现和之前滑块一样被切片，但是这个直接返回了切片的顺序，只需要找到前端还原的算法就可以复现。

![](https://attach.52pojie.cn/forum/202507/29/090514yxfbbjej1zos0oe9.png)

image.png (97.11 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5Mjc5MXxiZDJlNmMwN3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc5MXxiZDJlNmMwN3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

轨迹加密逻辑

2025-7-29 09:05 上传

这里我们发现了三个点，`challenge`参数值与前置接口的值保持一致，需要处理的值是`collectData`和`key`，在我写文章的时候发现，如果滑块长期静置不去动会有新接口更新`challenge`参数。这样就是我们整个要处理的部分

![](https://attach.52pojie.cn/forum/202507/29/091920wq7i79mc9qb997no.png)

image.png (49.65 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5Mjc5NHw5MzIwZjk2M3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc5NHw5MzIwZjk2M3wxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

重置接口

2025-7-29 09:19 上传

![](https://attach.52pojie.cn/forum/202507/29/091955g6bh66zc8bsiq1e8.png)

image.png (80.02 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5Mjc5NXxjYWQ1ZGQxYnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc5NXxjYWQ1ZGQxYnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

重置接口返回值

2025-7-29 09:19 上传

重置接口的载荷也是用过期的`challenge`参数去换新的`challenge`参数，这也可能是一个风控点，可以注意一下

## 二、`ast`还原混淆代码逻辑分析

我`ast`写的很丑陋，能用就行，大佬轻喷。

我们其实在接口返回的过程中已经看到一个动态加载的`js`代码，猜测应该是加密逻辑的部分，经过测试没有问题，确实是，但是在核心代码部分在虚拟机中执行（非常恶心），不能进行替换（可能是我太菜了，大佬们教教我）

所以`ast`解混淆后更多用于本地对照，方便分析流程。

### 1、分析混淆逻辑

`ast`解混淆是90%的思路+10%的`api`，思路分析是最重要的部分（蔡老板讲的）。先把全部代码扒拉下来，

![](https://attach.52pojie.cn/forum/202507/29/092242x5zi8t2u2adsgg8t.png)

image.png (752.97 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5Mjc5NnwwNTZhNWNjMHwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mjc5NnwwNTZhNWNjMHwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

编码混淆

2025-7-29 09:22 上传

首先可以看到的就是编码的混淆，这个用`ast`处理就很简单了.。然后发现他有很多赋值操作，都是同一个`a0_0x4000`，跟过去看看。

![](https://attach.52pojie.cn/forum/202507/29/093010vd4inffd4tmnrd2f.png)

image.png (109.25 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5MjgwMXw0OWU5NGFhYnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MjgwMXw0OWU5NGFhYnwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

解密函数

2025-7-29 09:30 上传

![](https://attach.52pojie.cn/forum/202507/29/093031ewzukxw0mjw0m0a9.png)

image.png (29.06 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5MjgwMnwyYzA2ZmExMHwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MjgwMnwyYzA2ZmExMHwxNzkzNTcwNzc3fDE2ODA4NnwyMDQ5NDEw&nothumb=yes)

大数组

2025-7-29 09:30 上传

发现是一个很明显的`ob`逻辑，大数组+解密函数，我们要调用这个需要做的就是把他存进内存里面，去替换调用
