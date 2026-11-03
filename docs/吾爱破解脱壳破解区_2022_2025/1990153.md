
# [CTF] 【BUUCTF】wdb2018_GUESS题解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990153
- 抓取时间: 2026-11-01T22:18:16

## 题目地址

[//" target="_blank" rel="noopener noreferrer nofollow"](http://:rel="nofollow noopener" href="http)

## 解题

- 查看可执行文件的安全属性，发现其是64位，有`canary`、`NX`，无`PIE`以及`GOT`可写。

![](https://attach.52pojie.cn/forum/202412/11/084012sxioo2xyn09oz0i2.png)

check.png (43.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0MjgyMXw2OTRkN2ZkZnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyMXw2OTRkN2ZkZnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 运行程序感受其逻辑结构，发现是个有三次机会的猜标志。放入`IDA`
- 在`main()`函数中，可以发现在37行存在一个`gets()`函数，可以利用栈溢出，需要注意程序有`canary`。分析结构发现，是把创建的`flag`文件读到了`buf`中，并存在了栈上。`strcmp()`中的`s2`也同样在栈上。

![](https://attach.52pojie.cn/forum/202412/11/084021ed9dm9cdyyw9mbdv.png)

main.png (101.18 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgyNXw2YzM1MzI2YnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyNXw2YzM1MzI2YnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 在`ubuntu16`环境下，由于存在金丝雀值，发生栈破坏时，对金丝雀产生了破坏会调用`__stack_chk_fail()`函数，此函数源码如下：

![](https://attach.52pojie.cn/forum/202412/11/084010g45z6lo5k3uy60l2.png)

__stack_chk_fail.png (29.41 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MjgyMHw1Y2QzM2E5MnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyMHw1Y2QzM2E5MnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 手动输入一个不合法的字符串，可以看到，发生栈溢出是会打印出`__libc_argv[0]`，其含义是程序的全路径。试验发现打印出的是文件的相对路径，如图
- `argv[0]`在`main()`函数的栈上，由此可以得到一个整体的思路：通过将`libc_argv[0]`覆盖为`read_got`表中内容，此时破坏了`canary`，将会调用`stack_chk_fail`，打印出`read_got`表内容，然后得到`libc`基址。下面动态调试，找到`libc_argv[0]`相对于`gets()`函数的偏移

![](https://attach.52pojie.cn/forum/202412/11/084016i6x1rfp1oprxxcx1.png)

gets地址.png (31.03 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgyM3xlMzBlM2UyZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyM3xlMzBlM2UyZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 动态调试找到`gets()`函数入口为图示，此为用户输入的字符串，要想构造垃圾字符覆盖栈空间到`__libc_argv[0]`位置，需要计算偏移，`__libc_argv[0]`的地址如下

![](https://attach.52pojie.cn/forum/202412/11/084019fttkts6yfky99bef.png)

libc_argv[0]地址.png (16.63 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgyNHwyNTEzYzdkZnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyNHwyNTEzYzdkZnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 计算偏移如下

![](https://attach.52pojie.cn/forum/202412/11/084037zfnpkld6mk1rbu6l.png)

偏移1.png (8.1 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MjgzMnxkYzlkZjQ0NnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzMnxkYzlkZjQ0NnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 所以可以编写`exp`泄露`read_got`表地址如下

![](https://attach.52pojie.cn/forum/202412/11/084027fia0l7cjnznpn5ta.png)

read_got.png (101.76 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc0MjgyOHxjNmVlMzZkZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyOHxjNmVlMzZkZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 进而计算`libc`基址，再由此计算出`environ`表地址。通过相关偏移就可得到所有栈上值的地址

![](https://attach.52pojie.cn/forum/202412/11/084014g1lv4mcfckz3chhl.png)

env地址.png (35.48 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgyMnw4OTJkZDg0YnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyMnw4OTJkZDg0YnwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

![](https://attach.52pojie.cn/forum/202412/11/084048pj9kptktp6u9m5pm.png)

值1.png (3.8 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgzN3xkNGUwN2FmM3wxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzN3xkNGUwN2FmM3wxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 可以得到环境变量地址开头1个字节为`\x7f`，编写下列代码泄露出栈地址

![](https://attach.52pojie.cn/forum/202412/11/084034lfnd6rljifqlnavo.png)

得到栈地址.png (38.37 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgzMXxjMWFjZmIwOHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzMXxjMWFjZmIwOHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

![](https://attach.52pojie.cn/forum/202412/11/084050l2je2zhzjhyjovjh.png)

值2.png (4.12 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc0MjgzOHwwMWZjMmVhZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzOHwwMWZjMmVhZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 接下来计算，`flag`位置与`stack`地址的偏移，经过动态调试可以得到

![](https://attach.52pojie.cn/forum/202412/11/084046tzzgk9j1eet9b44q.png)

运行env.png (11.26 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc0MjgzNnw0ODg2NGMzZXwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzNnw0ODg2NGMzZXwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

![](https://attach.52pojie.cn/forum/202412/11/084039pax2c0w42n2abviv.png)

偏移2.png (9.6 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgzM3xiNzJmZWUxOHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzM3xiNzJmZWUxOHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 偏移为360，编写`payload`如下

![](https://attach.52pojie.cn/forum/202412/11/084025lj9hjooa9207o0a0.png)

payload.png (22.24 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc0MjgyN3w0YTIzNzIxZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyN3w0YTIzNzIxZHwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

- 最后运行`exp`，得到`flag`

![](https://attach.52pojie.cn/forum/202412/11/084821mwxnnzex70qenhnm.png)

flag.png (36.97 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc0MjgzOXw5MDlkNjc3N3wxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgzOXw5MDlkNjc3N3wxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:48 上传

通过题目

![](https://attach.52pojie.cn/forum/202412/11/084023v1g5ghc5x9i2rrds.png)

pass.png (62.59 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc0MjgyNnwyN2FmZDY4OXwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjgyNnwyN2FmZDY4OXwxNzkzNTcxNDk0fDE2ODA4NnwxOTkwMTUz&nothumb=yes)

2024-12-11 08:40 上传

## 体会

- 需要掌握一定的系统知识，例如本题中的`ubuntu16`系统栈溢出调用`__stack_chk_fail()`函数，以及该函数的特性，会打印相关内容。
- 如果充分理解以上内容，可以帮助更快反映解题方向：利用程序会`fork()`3次，加上父进程，总共可以打印3次内容（`fork()`部分没有详细提及，具体可以参考其它blog），相当于可以泄露3次相关地址。
- 还需要更加熟练地掌握`gdb`动态调试，下合适的断点，找到想要得到的地址。
