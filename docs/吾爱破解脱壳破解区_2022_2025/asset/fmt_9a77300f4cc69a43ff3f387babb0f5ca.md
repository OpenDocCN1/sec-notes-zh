
# [CTF] flare-on ctf 2023 第三题mypassion分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1919883
- 抓取时间: 2026-11-01T22:29:52

## 前言
Flare-on每年的ctf都比较有趣，大家可以去试一下，已经办了10届比赛了，官网显示这届最难，因为庆祝比赛10周年。本文就2023的第三题mypassion学习一波逆向，很考验逆向基本功。主要防止忘了，写下来回顾一波。

## 题目
题目：aHR0cHM6Ly9mbGFyZS1vbi5jb20v

## 分析
查壳，无壳。

![](https://attach.52pojie.cn/forum/202405/02/010222sjgdi7wouw2zbdfq.png)
QQ图片20240429182856.png (172.64 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY5MzQwNHwwMWY4ZmNjM3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzQwNHwwMWY4ZmNjM3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 01:02 上传

```
strings.exe .\mypassion.exe 查看一下字符串，发现了有趣的字符串
C:\send\bitcoin\for\symbols\send\bitcoin\for\symbols\send\bitcoin\for\sym.pdb
```

挖矿的？可能根据某恶意软件修改的。运行了一下程序，后无反应，ok。

### IDA分析
`_start`函数没有反调试，直接来到`main`函数，可以看到程序运行需要带参数，并且参数要满足一些条件，大致用ida看了一下程序执行流程，没有看到特别有用的信息，不能一步登天。函数会在运行时会从数据段导`shellcode`执行，所以我打算动静结合，慢慢分析。首先，参数第7个字母影响`VirtualAlloc`，第一个字母为'0', 第2,3字母得满足`x+4*y == 0x127`，用`py`找一下, 我选择9，C

```
for x in range(0x20,0x7f):

        for y in range(0x20,0x7f):

                if(x+4*y==0x127):

                        print(chr(x),chr(y))
```

![](https://attach.52pojie.cn/forum/202405/02/010147ne08qsu8pzp118zl.png)
QQ图片20240429201311.png (48.58 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY5MzQwM3xiZjgxYzFmMXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzQwM3xiZjgxYzFmMXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 01:01 上传

所以得到初步参数09CxxRxxxxxxxxx, 然后用`windbg`动态调试，`windbgx.exe .\passion.exe 09CxxRxxxxxxxxx`，最好不要用`powershell`，后面会出现奇怪的问题？导致程序运行失败。`cmd`没问题

然后我们在`virtualAlloc`下断点，程序在这儿退出了，看下原因

![](https://attach.52pojie.cn/forum/202405/02/010101lszz8bd8hzm8zfwf.png)
QQ图片20240429200948.png (39.48 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY5MzQwMXw4MjFhZDU3NnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzQwMXw4MjFhZDU3NnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 01:01 上传

用`err.exe`查看错误值原因，查看相关文档，得到，`virtualAlloc`最后一个参数得是`PAGE_EXECUTE_READWRITE(0x40)`。即参数第7个字母得为@。

![](https://attach.52pojie.cn/forum/202405/02/005838zw4hfuerwpwuu1hr.png)
QQ图片20240429202126.png (40.38 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjY5MzM5OXwwYjMwMDVjNHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5OXwwYjMwMDVjNHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 00:58 上传

修改参数为09CxxR@xxxxxxxxx，继续往下分析

![](https://attach.52pojie.cn/forum/202405/02/010015amb3n03k3y4ck8cz.png)
QQ图片20240429211809.png (62.84 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY5MzQwMHxkNjlmODM4OXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzQwMHxkNjlmODM4OXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 01:00 上传

因为参数13字母会改代码，所以在`func1`函数入口下个断点，看看这个函数输入是什么，具体代码是干什么的

在偏移1299下断点，可以看到输入参数`v39`的值是5个'x', 为什么5个，因为`v40`和`v39`相邻，`v40`赋值为'x',高位被清零（汇编代码看会更明白），如图：

![](https://attach.52pojie.cn/forum/202405/02/005700lacgexxs183ddsx4.png)
QQ图片20240429223211.png (101.83 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5MzM5N3xlOTIyOTg0ZXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5N3xlOTIyOTg0ZXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 00:57 上传

然后我们可以明显看到`xabort 56h`这行汇编有问题，c678f456 （16进制） ，而78正好是'x'的16进制ASCII码，所以这个是受第13字母影响，我们可以猜测c6后面可能是45（试试），因为上面几行都是，即字母'E'。所以新参数：09CxxR@xxxxxExxx。也可以用`CyberChef`进行调试汇编（慢慢猜。。），后面也会用到，因为好几个地方会通过参数影响`shellcode`。

![](https://attach.52pojie.cn/forum/202405/02/005712ag9w9vg9xvv7uyov.png)
QQ图片20240429224901.png (122.63 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5MzM5OHw5NzY5ZGRhZXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5OHw5NzY5ZGRhZXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 00:57 上传

那么这段汇编代码在干嘛呢？它好像对后续没什么影响？直接分析很痛苦，动态分析就很舒服，我们知道输入是5个'x',即参数8-12位置的值，我们直接在`ret`前`f9`下断点，看看输出，相关寄存器啥情况,可以发现`rsp`有东西'brUc3',正好5位，单步分析的话会发现`rsp`的字符是临时算出来的。猜测可能和这个字符串比较，暂时不知道有什么用，后面相邻的代码也没用到它，但是我们可以把`x`改为这个，反正没影响。

![](https://attach.52pojie.cn/forum/202405/02/005400se4qiggefzefsclc.png)
QQ图片20240430105630.png (278.41 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY5MzM5Nnw4MTY0Yzk1OXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5Nnw4MTY0Yzk1OXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)
2024-5-2 00:54 上传
