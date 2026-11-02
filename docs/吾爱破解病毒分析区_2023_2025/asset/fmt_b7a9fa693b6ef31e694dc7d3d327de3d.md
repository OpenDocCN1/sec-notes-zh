
# [PC样本分析] 警惕，黑客通过伪装游戏作弊器 dll注入器等的GitHub开源项目传播多种恶意软件

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1913167`
- 抓取时间: `2026-11-01T23:48:17`

## 一、背景

近期发现有人称在GitHub上下载了一个工程文件打开，不幸中招恶意软件，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132528viq992jkf0zg3ni2.png)
捕获1.PNG (40.23 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4OTIwOXwzN2ZhZTRhZXwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIwOXwzN2ZhZTRhZXwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:25 上传

### （一）文件信息

下载后可发现，该项目的`sln`工程文件，实为一个`PE`可执行文件，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132541ndrdt6oc8gj57cdp.png)
捕获2.PNG (131.56 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIxMnw4N2ExZTA4NnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIxMnw4N2ExZTA4NnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:25 上传
`VirusTotal`中的历史文件名称，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132552l00np74epncyu0pz.png)
a0.PNG (58.34 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIxNHxkYTFlODYwZnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIxNHxkYTFlODYwZnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:25 上传
该`PE`文件使用了无效数字签名，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132606scbrakmwcfzm6rra.png)
捕获3.PNG (32.16 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIxNnxhMmJmOWNmOHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIxNnxhMmJmOWNmOHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:26 上传
该文件使用`.NET`编写，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132618jodqz2vgg2wgpwf7.png)
捕获4.PNG (39.79 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIxN3w1MzYzYzQwN3wxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIxN3w1MzYzYzQwN3wxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:26 上传

### （二）恶意GitHub organization信息

该仓库隶属于一个名为`Hayrowi`的GitHub organization，截至发帖前仍然存在并有效，该organization下部分仓库，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132631dc0kr0cngk0rgwee.png)
捕获5.PNG (115.39 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4OTIxOHwyZGQxNmMzMnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIxOHwyZGQxNmMzMnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:26 上传
我们随意点开几个仓库，均发现了相关恶意软件，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132644gj6blleojoe8yv97.png)
捕获6.PNG (36.13 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY4OTIxOXw2MzUxMDBjZnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIxOXw2MzUxMDBjZnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:26 上传
![](https://attach.52pojie.cn/forum/202404/13/132700xci25a0c0390ui9p.png)
捕获7.PNG (56.12 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4OTIyMHwyZWEyNWMyNHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyMHwyZWEyNWMyNHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:27 上传
![](https://attach.52pojie.cn/forum/202404/13/132717ezwamajgwopde49d.png)
捕获8.PNG (65.14 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIyMXw4NTU1MzlmN3wxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyMXw4NTU1MzlmN3wxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:27 上传
其中，`Releases`中也包含相关的恶意软件，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132733rm200c05ml72380r.png)
捕获9.PNG (94.07 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4OTIyMnxmNWU2OTdhYnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyMnxmNWU2OTdhYnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:27 上传
此外，这些仓库都包含一个名为`Star`的`workflows`，Github actions执行后会修改`LOG`文件中的最后更新时间，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132746q9eph2a4q00e110q.png)
捕获10.PNG (55.55 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4OTIyM3w0MDc4ZGQzYnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyM3w0MDc4ZGQzYnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:27 上传
![](https://attach.52pojie.cn/forum/202404/13/132759nemxwixze0wpk2jk.png)
捕获11.PNG (56.67 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIyNHxhOWY4ZWExNHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyNHxhOWY4ZWExNHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:27 上传
（比较有趣的是，黑客填写的本地git邮箱是ischhfd83#rambler.ru）
![](https://attach.52pojie.cn/forum/202404/13/132815yl1kkatd3kkmska8.png)
捕获12.PNG (22.59 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4OTIyNXxiNDg3MGNkMnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyNXxiNDg3MGNkMnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:28 上传
其`README.md`均较为逼真，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132836l3n132iuna55pm1j.png)
捕获13.PNG (265.44 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIyNnxjZDI0N2Y2ZHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyNnxjZDI0N2Y2ZHwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:28 上传

## 二、样本分析

### （一）第一层.NET文件

将样本使用`dnspy`反编译后，第一眼映入眼帘的是样本伪装成`sln`文件有多么“良苦用心”，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132855ae7d3uhzdyq77r87.png)
捕获14.PNG (22.86 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIyN3w0NzVjMzNhMXwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyN3w0NzVjMzNhMXwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:28 上传
随后映入眼帘的便是`UHJvZ3JhbUFB`下的 "`DownloadAll()`" "`DownloadRU()`" "`GetCountry()`" 等函数名，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132909aq3xxzkkco352gqc.png)
捕获15.PNG (21.54 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4OTIyOHwxNDhmZTkyMnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyOHwxNDhmZTkyMnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:29 上传
我们还可以看到，其有一个`namespace`，名为`Costura`，进入后可以看到其下`QXNzZW1ibHlMb2FkZXJB`类和`QXNzZW1ibHlMb2FkZXJB`类之下的静态构造函数`.cctor()`中有大量调用`0xb11a1()`模块进行疑似加解密的操作，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/132933c44k4khzuur4hn84.png)
捕获16.PNG (120.66 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4OTIyOXxkM2Y4MDA3YXwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIyOXxkM2Y4MDA3YXwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:29 上传
![](https://attach.52pojie.cn/forum/202404/13/132946ec797pqgw4h3hyb9.png)
捕获17.PNG (115.99 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4OTIzMHw2ZTgyNjhlZnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4OTIzMHw2ZTgyNjhlZnwxNzkzNTc2ODk0fDE2ODA4NnwxOTEzMTY3&nothumb=yes)
2024-4-13 13:29 上传
于是我们进入`0xb11a1()`中查看，可以看到`0xb11a1()`是一个解密函数，其首先将转化为`Base64`编码的二进制数据解码得到一个字节数组，随后使用`MD5CryptoServiceProvider()`计算指定密钥，随后使用`TripleDES`算法进行解密操作，最后返回的结果为`Encoding.UTF8.GetString(bytes)`，如下图所示：
![](https://attach.52pojie.cn/forum/202404/13/133005bgt011iy4ipi1vp1.png)
捕获18.PNG (111.37 KB, 下载次数: 0)
