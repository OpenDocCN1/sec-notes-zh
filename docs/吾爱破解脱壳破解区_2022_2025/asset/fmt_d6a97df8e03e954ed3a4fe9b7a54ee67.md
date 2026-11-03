
# [Web逆向] 某某网站载荷逆向------小白讲解05

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1999830
- 抓取时间: 2026-11-01T22:16:45

本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关.本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责。

![](https://static.52pojie.cn/static/image/smiley/default/17.gif)

![](https://static.52pojie.cn/static/image/smiley/default/17.gif)

![](https://static.52pojie.cn/static/image/smiley/default/17.gif)

---

目标网址: aHR0cHM6Ly93d3cubXl0b2tlbmNhcC5jb20vbmV3LXRva2VuLWxpc3RpbmdzLzEv

---

## 文章思路

1. 找出目标数据包
2. 分析参数跟栈
3. 找到加密位置
4. 扣取代码与修改代码

---

文章哪里讲的不对或不清楚 还请各位大佬给出宝贵意见

![](https://static.52pojie.cn/static/image/smiley/default/8.gif)

![](https://static.52pojie.cn/static/image/smiley/default/8.gif)

![](https://static.52pojie.cn/static/image/smiley/default/8.gif)

---

## 第一步：

### 1.1 打开开发者工具找出目标数据包

![](https://attach.52pojie.cn/forum/202501/14/182400u7egigpqcd1odt1e.png)

图片 1.png (119.85 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc1MnxmMDk2Y2U4OXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1MnxmMDk2Y2U4OXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:24 上传

### 1.2 对载荷进行分析 确定要逆向的参数

![](https://attach.52pojie.cn/forum/202501/14/182448bat8htzuayygzogu.png)

图片 2.png (52.64 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc1M3wxNDZkODI4NHwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1M3wxNDZkODI4NHwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:24 上传

### 1.1 分析完之后就开始点击启动器，进入源文件对参数进行跟踪

### 1.2 然后下断点，追目标参数

![](https://attach.52pojie.cn/forum/202501/14/182556worlpzjz30s7dp0l.png)

图片 3.png (27.85 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0OTc1NHwwMjJmOTU0NnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1NHwwMjJmOTU0NnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:25 上传

![](https://attach.52pojie.cn/forum/202501/14/182600gch4dswqh1tw6d6d.png)

图片 4.png (129.92 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc1NXwwNGM0M2M0NnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1NXwwNGM0M2M0NnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:26 上传

## 第二步

### 2.1 接着依次向后跟栈 到指定位置

### 2.2 让断点断在加载器处，我们进行分析

![](https://attach.52pojie.cn/forum/202501/14/182712gfgee21gie2a21tv.png)

图片 5.png (94.16 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc1N3xhMjA4OTFlYXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1N3xhMjA4OTFlYXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:27 上传

### 2.3 我们看这段代码

每次执行，它里面的函数，结束后，就会将执行过的函数删除掉，也就是 `L` 变量，我们看到里面有很多函数。

### 2.4 在这里进行分析

首先各位可以在堆栈往后上追，参数就丢了，那么参数在这之前是有的，在之后就没了，所以推断出，参数的生成，是在这个拦截器里面生成的，也就是 `L` 里面某个函数返回的，所以我们依次进入，来寻找。

![](https://attach.52pojie.cn/forum/202501/14/182821xb9ymrx1bzr1xkhd.png)

图片 6.png (54.82 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc1OHwwOGJkMThhYXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1OHwwOGJkMThhYXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:28 上传

## 第三步：

根据上述我们开始进入拦截器在每一个函数的 `return` 处下断，然后再分页发包看最后会断在那个地方，如果没有断住说明数据包没有经过这里，随后我们另外分析。

### 3.1 因为 `L` 里面的下标 3 没有，所以不展示，只进入有的

![](https://attach.52pojie.cn/forum/202501/14/182928b5hzxx5yy2995m22.png)

图片 7.png (30.29 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc1OXw3OGJlZDk0MnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc1OXw3OGJlZDk0MnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:29 上传

![](https://attach.52pojie.cn/forum/202501/14/182931a5lm5vapwdpdad3m.png)

图片 8.png (58.7 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc2MHxlNzgxMmNkZnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2MHxlNzgxMmNkZnwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:29 上传

![](https://attach.52pojie.cn/forum/202501/14/182933slx95nrgp2v292nb.png)

图片 9.png (67.6 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc2MXxjZjgyM2RhZXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2MXxjZjgyM2RhZXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:29 上传

![](https://attach.52pojie.cn/forum/202501/14/182936kw72zzjvc7j1fn2v.png)

图片 10.png (79.6 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc2MnxhYzJhM2Q4MXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2MnxhYzJhM2Q4MXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:29 上传

![](https://attach.52pojie.cn/forum/202501/14/182938w5kamb1yr2sbr558.png)

图片 11.png (70.86 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc2M3w0ZTA4MmMwMXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2M3w0ZTA4MmMwMXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:29 上传

### 3.2 按下图，去掉其余断点和拦截器断点，然后分页发包看断点最后断在哪里

![](https://attach.52pojie.cn/forum/202501/14/182943pcle3vfv5zf3c3xf.png)

图片 12.png (123.43 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0OTc2NHw3ZjY4MzgxZHwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2NHw3ZjY4MzgxZHwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:29 上传

### 3.3 可以看到断点断下来了

并且证实了，我们的目标数据包，经过这里，还有最后返回的 `n.data` 里面就存放了我们想要的参数，这样看还不清晰，我们将全部断点去掉，只保留这一个 ，并且在他的头部也下一个。

![](https://attach.52pojie.cn/forum/202501/14/183209yb7o6vg659bdzaea.png)

图片 13.png (100.72 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0OTc2Nnw1NDM3OGY3ZXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2Nnw1NDM3OGY3ZXwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:32 上传

### 3.4 操作完之后 我们先看一看 这个函数内有没有，线索，或者参数就是在这里面生成的

![](https://attach.52pojie.cn/forum/202501/14/183249g8s42nyiuno4f26f.png)

图片 14.png (35.24 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0OTc2N3w4NGU5OGJmMHwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTc2N3w4NGU5OGJmMHwxNzkzNTcxNDAzfDE2ODA4NnwxOTk5ODMw&nothumb=yes)

2025-1-14 18:32 上传

### 3.5 不过为了准确，我们分页发包让断点，到头部，看看有没有新情况

### 3.6 断点到头部之后

我们看到形参 `n` 里面的 `data` 我们要的参数没了，那我们就又可以猜测上面没有参数的生成，那么就有可能是在往下时进行的参数生成与赋值,因为我们需要参数在最后 `return n` 里面，可以在 3.3 清楚看到,那我们就单步调试一步一步走，看看在哪一行出现的。

![](https://attach.52pojie.cn/forum/202501/14/183434wf3cs9mfnhxnt5xt.png)

图片 15.png (44.24 KB, 下载次数: 1)
