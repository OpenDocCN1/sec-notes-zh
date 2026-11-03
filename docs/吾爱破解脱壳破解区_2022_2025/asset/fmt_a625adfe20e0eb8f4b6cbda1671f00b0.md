
![](https://attach.52pojie.cn/forum/202501/10/121625pblq7iqi5tcqxwqz.png)
图片 13.png (28.88 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0OTAxNnwxMGZiNzgxMXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAxNnwxMGZiNzgxMXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:16 上传

![](https://attach.52pojie.cn/forum/202501/10/121627sffxxd4i9fox79o2.png)
图片 14.png (13.01 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAxN3w5ZTlhYzc4OXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAxN3w5ZTlhYzc4OXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:16 上传

### 3.4
然后我们继续单步执行，就到了 3.2 处看到了他将返回的数据进行切分又赋值给了`e.token`，最后就是我们要的参数之一

![](https://attach.52pojie.cn/forum/202501/10/121630p0q9spp4zqo2597s.png)
图片 15.png (13.98 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAxOHw0ZjFmODczMHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAxOHw0ZjFmODczMHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:16 上传

## 第四步
### 4.1
我们还是继续单步调试，虽然速度有点慢，但可以清晰的看到每一步

### 4.2
单步调试一直到这里`c.prototype.__processRequestUrl`，在这里面一直执行就会到`sign`的加密处，所以我们可以试试向下看，因为这是多次调试出来的，并且，它们都在同一个地方所以直接看到这个地方的结尾

![](https://attach.52pojie.cn/forum/202501/10/122028gvfnfff7tkfi8n88.png)
图片 16.png (42.57 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAyMHw0M2QwM2JlZnwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAyMHw0M2QwM2JlZnwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:20 上传

### 4.3
上图有些我给它折叠起来了，看着不杂乱，所以我们现在，直接在`sign`处下段，然后移动到此断点，查看内容，然后我们将这个包执行完，将此处的`sign`与发送完毕的包的`sign`对比，可以看到内容一致，那么我们就可以确定 `p` 就是我们需要的密文，那么接下来我们就开始，扣取代码，打印出`sign`

![](https://attach.52pojie.cn/forum/202501/10/122304x8ebaqa8h1bao896.png)
图片 17.png (64.59 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0OTAyMXw2YTVkMjc2ZHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAyMXw2YTVkMjc2ZHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:23 上传

![](https://attach.52pojie.cn/forum/202501/10/122306rzldzzrlzuo58lj5.png)
图片 18.png (11.89 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAyMnwwNWQxNDdjN3wxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAyMnwwNWQxNDdjN3wxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:23 上传

![](https://attach.52pojie.cn/forum/202501/10/122309sgczjbvju9ghgz1j.png)
图片 19.png (24.71 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAyM3w0MjdlYzNjN3wxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAyM3w0MjdlYzNjN3wxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:23 上传

## 代码的扣取和调用

## 第五步
### 5.1
这里我们还会继续分析代码参数组成以及代码扣取、调用

### 5.2
我们找到了`sign` ，它是由`p` 生成的，那么就可以知道，`p`是在某处定义了，所以我们向上找，因为肯定是先定义后使用，步骤2就是我们要找的`p`,这得多亏作用域，因为`p`在步骤3里只有一个，超出步骤3，就无法使用了，那么可以知道`sign`使用的就是这个`p`

![](https://attach.52pojie.cn/forum/202501/10/122727ece6hkddhze0v6ac.png)
图片 20.png (37.54 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAyOHwxZGIyOGQwNXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAyOHwxZGIyOGQwNXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:27 上传

### 5.3
我们扣取出`p`放到Code里进行对`p`的分析，扣取完毕后我们返回浏览器，对这个函数`p`尾部下断点，查看值的生成

![](https://attach.52pojie.cn/forum/202501/10/122730e8v49arqnnm8kl6a.png)
图片 21.png (39.26 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAyOXwyMGNiNzY1OHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAyOXwyMGNiNzY1OHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:27 上传

![](https://attach.52pojie.cn/forum/202501/10/122732pra5l5gkdttpp66o.png)
图片 22.png (31.47 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAzMHw4YTU0MzhiNXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAzMHw4YTU0MzhiNXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:27 上传

### 5.4
下完断点后我们重新分页发包，让断点到此处，我们从内往外看，先看函数的返回值，然后与 `sign:p` 的值进行对比，之后再看传入`p`的参数，可以看到函数的返回值，与`sign:p`的值是一致的，而返回值，所加密的参数已经用红色箭头标记出来

![](https://attach.52pojie.cn/forum/202501/10/123103beeevjmjztnvtvr4.png)
图片 23.png (27.05 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAzMnw0MjEwNDE5MHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAzMnw0MjEwNDE5MHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:31 上传

### 5.5
我们在来看传入的参数，`token`我们在单步调试第三步已经说明了它的生成，这里的`token`和 第三步的`token`是一样的，大家有兴趣的可以自己对比对比，`a`是时间戳，`s`多次调试时一个固定的字符串，`n.data`就是我们的表单数据里面的`page`就是我们当前所在的页数，他们用“&”连接之后传入到5.3的头部

![](https://attach.52pojie.cn/forum/202501/10/123106ydudxnvjqvndugnd.png)
图片 24.png (45.77 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAzM3w3MDUxMzUzOXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAzM3w3MDUxMzUzOXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:31 上传

### 5.6
扣取的代码会报错是因为参数的缺少，而我们在5.5已经分析完毕，所以我们可以自己创建，也可以回到浏览器扣取，这里我扣取的是浏览器器的小小改动一下就可以用

![](https://attach.52pojie.cn/forum/202501/10/123535e4ym3phvxs5xxxrm.png)
图片 25.png (86.59 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAzNXw2YmJlNGU1NHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAzNXw2YmJlNGU1NHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:35 上传

![](https://attach.52pojie.cn/forum/202501/10/123538wyd8cjgvdrj1gvl7.png)
图片 26.png (17.82 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc0OTAzNnxjMzg3NGYzOXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAzNnxjMzg3NGYzOXwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:35 上传

### 5.7
我们来看一下最终的效果，显然结果是对的，那么到此`sign`的加密就结束了

![](https://attach.52pojie.cn/forum/202501/10/123642jzlvnvgdhic3nn1c.png)
图片 27.png (111.38 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0OTAzN3w0ZmUxNGUyNHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0OTAzN3w0ZmUxNGUyNHwxNzkzNTcxNDExfDE2ODA4NnwxOTk4Njg5&nothumb=yes)
2025-1-10 12:36 上传

## 数据包Cookie分析

## 第六步
### 6.1
对于这个站虽然扣取出了代码和加密参数，但如果不是多次调试，和猜想实践，就会造成少扣取，打击信心，命名代码扣了很多，但最后组合，结果不是很理想
