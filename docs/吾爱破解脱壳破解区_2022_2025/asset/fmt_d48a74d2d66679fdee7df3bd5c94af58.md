
# 分析记录

## 初始参数破解

```
chr((0x74^0x16)) ='b'
chr((0x65^0x17)) ='r'
chr((0x6E^0x3B)) ='U'
chr((0x74^0x17)) ='c'
chr((0x65^0x56)) ='3'
```

所以新参数：`09CxxR@brUc3Exxx`

继续往下分析，结合chatgpt和分析，将`main`函数中后面的函数重命名

![](https://attach.52pojie.cn/forum/202405/02/005330njw4nnn4wgmnezkh.png)

QQ图片20240430012408.png (82.41 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM5NXw5MzEzZTVjNHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5NXw5MzEzZTVjNHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:53 上传

接着进入`sub_1400013E0`继续分析，`ida`静态分析一波，可以看到，函数会通过`/`来截取参数，所以修改新参数为：`09CxxR@brUc3Exxx/test1/test2/` , 大体分析一下，`sub_1400013E0`会从数据段写入文件从而创建文件，并且会解密数据段的数据，作为代码执行。

![](https://attach.52pojie.cn/forum/202405/02/005312gxmgnm3woworz3nn.png)

QQ图片20240430013040.png (94.81 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM5NHwzMjE4MzZjN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5NHwzMjE4MzZjN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:53 上传

![](https://attach.52pojie.cn/forum/202405/02/005244zuh4npngvugpvhsv.png)

QQ图片20240430013301.png (71.95 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzM5M3xhYTIyOGI3M3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5M3xhYTIyOGI3M3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:52 上传

## `sub_1400013E0` 分析

`sub_1400013E0`中间还有个有趣的部分，现在用不到，但后面有影响，即`v25`变量，`v25 = 0x1f + day(系统今天的日期)`, `v25`变量会和数据段进行运算（而这块正属于文件的尾部）。

![](https://attach.52pojie.cn/forum/202405/02/005145t9111g3tjrw10s4j.png)

QQ图片20240430014332.png (65.73 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzM5Mnw5NWYwNjAzOHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5Mnw5NWYwNjAzOHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:51 上传

继续后面，我们通过动态分析，进入`sub_1400013E0`动态解密的code，看看这块具体干了什么，在1892哪儿下断点，运行调试器后会发现生成了`test1`的文件，所以参数会影响文件名。

到达断点后，我们直接单步进入动态分析。在函数前下断点，我们只关心输入和输出。

![](https://attach.52pojie.cn/forum/202405/02/005122dm8yo6expyps8jor.png)

QQ图片20240430021553.png (260.32 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM5MXxkN2ZhNTI2YXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5MXxkN2ZhNTI2YXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:51 上传

根据`main`函数，可以得到390是`retrieve_140002F00`，380是`strtol_1400071A0`，358是`ExitProcess`，以此类推，最后的3B0是`sub_1400018B0`函数，所以后面需要分析`sub_1400018B0`

动态分析后，可以调试得到这段代码在截取`test2`，并且`test2`前面得含有数字，并且该数字会转为4为基数的数字，然后与后面数字后面的字符串长度比较，比如这里我选择`0x20`，转为4为基数为8，所以后面需要8个字符，不然`ExitProcess`，so修改参数为`09CxxR@brUc3Exxx/test1/20testtest/`

## `sub_1400018B0` 分析

接着分析`sub_1400018B0`函数，大体过一遍，该函数会读取之前创建的`test1`文件，并通过输入参数对该文件进行一些判断。该函数开头和`retrieve_140002F00`作用一样，在这里它截取第三段，修改参数`09CxxR@brUc3Exxx/test1/20testtest/test3/`，然后在`exitprocess`函数下断点，查看一下条件满足

![](https://attach.52pojie.cn/forum/202405/02/004923yc84dd6jcct0cdtx.png)

QQ图片20240430112957.png (50.21 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY5MzM4OXxkZGYwYmM1NXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4OXxkZGYwYmM1NXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:49 上传

![](https://attach.52pojie.cn/forum/202405/02/004934cct3bkpbeuk90e4p.png)

QQ图片20240430115757.png (218.69 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY5MzM5MHwzZTdiMTUzNXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM5MHwzZTdiMTUzNXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:49 上传

所以第三段得改为`'?pizza'`，加‘？’是因为`v8[0]`没有被比较，跟踪一下`v12`，`v12`的逻辑与之前`sub_1400013E0`函数中`v25` 变量一样，所以`v8[0] = 0x1f + day(今天30）`即字符`'='`

修改参数为`09CxxR@brUc3Exxx/test1/20testtest/=pizza/`

![](https://attach.52pojie.cn/forum/202405/02/004732fy1nglg6fnefe6t6.png)

QQ图片20240430121047.png (39.54 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM4OHw5NTZiOWJmOXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4OHw5NTZiOWJmOXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:47 上传

接下来我们在下一个`exitprocess`下断点，可以看到判断逻辑是文件头，所以我们在`sub_1400013E0`创建文件时忽略了一些东西，我们再去看一下哪里对文件头进行了操作。

![](https://attach.52pojie.cn/forum/202405/02/004607dvx8poqbipoo0phb.png)

QQ图片20240430133209.png (52.09 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzM4N3w2OGM5MDE2YnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4N3w2OGM5MDE2YnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:46 上传

回到`sub_1400013E0`，文件开头第一段进行了操作，从中提取了数字，所以第一段得包含数字，赋值给了`v5`，而`v5`会影响文件头，所以修改参数为：`09CxxR@brUc3Exxx/1337test1/20testtest/=pizza/`

![](https://attach.52pojie.cn/forum/202405/02/004527egt1lfsiwiivigvx.png)

QQ图片20240430152005.png (42.86 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM4NnwxZjEwMWRiMnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4NnwxZjEwMWRiMnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:45 上传

用`010editor`，可以看见`test1`开头就是`0x11333377`。继续`sub_1400018B0`函数，调试定位下一个`exitprocess`前面，看看需要什么条件。

![](https://attach.52pojie.cn/forum/202405/02/004311dondj0dk6rjid7gp.png)

QQ图片20240430152930.png (45.54 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY5MzM4NXw3MWVkNDRjZXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4NXw3MWVkNDRjZXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:43 上传

![](https://attach.52pojie.cn/forum/202405/02/004124qrtl0n2cihl3wkhk.png)

QQ图片20240430154109.png (121.96 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzM4NHw3YjRlOThlMnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4NHw3YjRlOThlMnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:41 上传

调试可以得到，比较的是文件名，需要为`pr.ost`, 所以修改参数为`09CxxR@brUc3Exxx/1337pr.ost/20testtest/=pizza/`，成功通过判断。后面函数进入`sub_140002170`。

## `sub_140002170` 与 Shellcode 分析

`sub_140002170`里面的逻辑是比较难以理解的，开头的`sub_140001DA0`，`sub_140001F80`返回的都是函数指针，然后里面的逻辑也是动态生成的，不能理解。`sub_140001F80`还受输入参数影响，会改变代码

试试用`windbg`从内存打印`sub_140001DA0`函数指针的汇编代码，进行分析，或者动态分析函数输入输出从而了解功能。`sub_140001F80`后面再分析。

![](https://attach.52pojie.cn/forum/202405/02/003509cw3s8wyqssmu1szy.png)

QQ图片20240501171655.png (49.49 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM3N3xmZDIyMjgwZnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM3N3xmZDIyMjgwZnwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:35 上传

![](https://attach.52pojie.cn/forum/202405/02/003541ofmrwcbm6yxbdtwn.png)

QQ图片20240501171239.png (60.98 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzM3OHw0YWQ5NGRkN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM3OHw0YWQ5NGRkN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:35 上传

`sub_140001DA0`的`f1.bin`丢进`ninja`可以反汇编为`c`语言，该函数功能与`retrieve_140002F00`一样，用`/`截取字符串，也可以通过动态分析根据输入输出知晓功能。

![](https://attach.52pojie.cn/forum/202405/02/004001cwwmtpwmwpwmpeom.png)

QQ图片20240430195325.png (78.34 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM4M3xiYWIxN2I3MXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4M3xiYWIxN2I3MXwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:40 上传

`shellcode2`暂时理解不了，但他与`shellcode3`相关。但是我们可以猜想一下，输入的参数是个指针和字符串，输出是指针。那么就有这么几个情况，1.对字符串的处理，字符串对比或者添加什么的。2.看返回
值，`shellcode2`最后赋值给了`*(a1+3C8)`,在后面的`sub_140002910`函数中，可以找到它被用做函数，所以返回的是函数指针，可以猜想`shellcode2`的作用可能是根据字符串查找函数地址，类似`GetProcAddress`
函数，`shellcode2`的输入指针还是`shellcode3`的返回值。理解`shellcode2`有助于理解`shellcode3`。先在`shellcode3`下断点，因为输入参数会影响`shellcode3`的代码，我们把汇编代码扣出来，慢慢分析。修改参数
为：`09CxxR@brUc3Exxx/1337pr.ost/20testtest/=pizza/XXXXXXXXXXXXX/`

![](https://attach.52pojie.cn/forum/202405/02/003902vzczrssjjul43qe4.png)

QQ图片20240501125735.png (52.56 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MzM4MXw1M2VhMWIwN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4MXw1M2VhMWIwN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:39 上传

![](https://attach.52pojie.cn/forum/202405/02/003914ar0phtdp0huvdkd3.png)

QQ图片20240501130330.png (43.85 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM4Mnw0MDljYzVmN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4Mnw0MDljYzVmN3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:39 上传

将`shellcode3`抠出来的16进制，放到`cyberchef`，`'X'`的ASCII码是`0x58`，找到无效的指令，影响`shellcode3`代码的输入字符（第四段字符串），依次位置是“12,5,8,9,7,6”，然后仔细分析一下应该用什么代替

![](https://attach.52pojie.cn/forum/202405/02/003629i0obothiovzpozle.png)

QQ图片20240501150354.png (98.25 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MzM4MHxhMjkyMzk0MHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM4MHxhMjkyMzk0MHwxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:36 上传

![](https://attach.52pojie.cn/forum/202405/02/003558pjp85rjm4b788qq4.png)

QQ图片20240501153613.png (87.57 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MzM3OXxhZmU3ZGE4Y3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzM3OXxhZmU3ZGE4Y3wxNzkzNTcyMTkwfDE2ODA4NnwxOTE5ODgz&nothumb=yes)

2024-5-2 00:35 上传

在上图，我们慢慢分析，首先第一个`58` 后面接着的是 “`48 8b`”，明显像`mov`指令，所以大概率是`mov`，那么什么指令后面还能接`mov`，而且"`MOV RAX,QWORD PTR [0000000000000060]`",有个`60`，显而易见`[Asm]` 纯文本查看 复制代码
