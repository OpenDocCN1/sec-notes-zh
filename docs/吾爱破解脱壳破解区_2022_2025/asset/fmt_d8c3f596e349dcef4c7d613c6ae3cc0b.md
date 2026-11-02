
# `Name` 字符串存储与长度判断

`Name` 字符串（以字符串的对象的形式）存储的地址为：`[ebp-0x88]`。

## 长度判断

接受到字符串后，下一步是判断 `Name` 的长度是否小于 4（这个部分的代码一直到 `4030F9`，其实有一部分是小于 4 时的错误输出），判断依据为：

![](https://attach.52pojie.cn/forum/202203/26/004305chah006dn0asknc0.png)

``6{56)GN7B7S{`@YZ9)J4(6G.png`` (31.74 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgxOHwyNjE2ODE5YnwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgxOHwyNjE2ODE5YnwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:43 上传

`__vbaLenVar` 函数判断的是字符串长度。

`__vbavartstLt` 这里是上一个函数返回的长度，是否大于等于于 4，这里注意是大于四时才跳转到 `4030F9` 处的地址继续执行，否则执行下方代码（也就是报错）。

![](https://attach.52pojie.cn/forum/202203/26/004505u3k93c2rjr151qbj.png)

``284{G1H@}FCZ}R(%}5O`[0G.png`` (15.06 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwNzgxOXwyMzNjOGU1ZXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgxOXwyMzNjOGU1ZXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:45 上传

### 长度小于 4 时的测试

这里尝试了一下长度小于四时：

![](https://attach.52pojie.cn/forum/202203/26/004719mfsfaffb44jvmubb.png)

`QF4[5BZHU4J$ET)RE$P5BWN.png` (9.91 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgyMHwxYWUwMGU2Y3wxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyMHwxYWUwMGU2Y3wxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:47 上传

## 加密循环

当输入的数值不小于 4 跳转到 `4030F9` 处后，下面代码进行的就是加密循环了，大概流程如下：

```
00403102   .  BB 04000000   mov ebx,0x4                                ; EBX = 4(loop count)
···
0040318B   .  FF15 30104000 call dword ptr ds:[  ; \__vbaVarForInit
00403191   .  8B1D 4C104000 mov ebx,dword ptr ds:[  ;     msvbvm60.rtcMidCharVar
00403197   >  85C0          test eax,eax                               ; loop start
00403199   .  0F84 06010000 je abexcm2-.004032A5
···
0040329A   .  FF15 C0104000 call dword ptr ds:[  ; \__vbaVarForNext
004032A0   .^ E9 F2FEFFFF   jmp abexcm2-.00403197                      ; loop end
004032A5   >  8B45 08       mov eax,dword ptr ss:[ebp+0x8]
```

先总体分析一下这个循环的框架，`EBX` 作为循环的次数，而 `vbaVarForInit()` 函数和 `vbaVarForNext()` 函数是对字符串对象进行逐个字符引用。

### 加密方法分析

下面我们主要对循环的加密方法进行分析：

在循环开始也就是 `403102` 处下个断点，`Ctrl+F2` 重新调试，弹出对话框随便输入一些东西：

开始一路 `F8` 分析，直到注意到下图

![](https://attach.52pojie.cn/forum/202203/26/005215ip59vjiptdvqi2pd.png)

``5)2WC6@4{JV5EW`[TFSJRIE.png`` (23.67 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgyMnxkYmJhYWY4ZnwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyMnxkYmJhYWY4ZnwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:52 上传

`__vbaStrVarVal` 函数获取该字符的 `ASCII` 码的内存地址（直接指向该字符而不是字符变量）。

`__rtcAnsiValueBstr` 获取该字符的 `ASCII` 码的十六进制值并保存在 `eax` 中（`eax` 直接存储着这个码）。

当指令执行到 `4031FD` 处时，就可以看到 `eax` 的值为 68。

![](https://attach.52pojie.cn/forum/202203/26/005259fpcdppucolgupgod.png)

``%M7YBQK{QYSVY@ZBW`RE.png`` (34.75 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgyM3wzMmViMTdjNXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyM3wzMmViMTdjNXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:52 上传

#### 汇编代码解释

```
004031F0   .  FF15 80104000 call dword ptr ds:[]    ; \__vbaStrVarVal

004031F6   .  50            push eax                                          ; 从Name字符串中获取的一个字符(UNICODE)

                                                                              ; 这里获得的值为“h”

004031F7   .  FF15 1C104000 call dword ptr ds:[; \rtcAnsiValueBstr()获取字符的十六进制ASCII码值

00403221   .  8D55 AC       lea edx,dword ptr ss:[ebp-0x54]

00403224   .  8D85 24FFFFFF lea eax,dword ptr ss:[ebp-0xDC]

0040322A   .  52            push edx                                          ; EDX = 68

0040322B   .  8D8D 64FFFFFF lea ecx,dword ptr ss:[ebp-0x9C]                   

00403231   .  50            push eax                               

00403232   .  51            push ecx                                          ; dest

00403233   .  C785 2CFFFFFF>mov dword ptr ss:[ebp-0xD4],0x64                  

0040323D   .  89BD 24FFFFFF mov dword ptr ss:[ebp-0xDC],edi                   ; [EBP-DC] = EAX = 64
```

接着往下走 `call` 了一个 `__vbaVarAdd`，这个函数的作用是两个变量值相加。

![](https://attach.52pojie.cn/forum/202203/26/005432yjcqdrcr58cyz7dq.png)

``%XLB_3K5Y{F8H@8I{O`~GVW.png`` (8.43 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgyNHwzZjY4ZGI1M3wxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyNHwzZjY4ZGI1M3wxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:54 上传

看上面的两条指令，第一个变量的值已经确定为 `0x64`，另一个的值跟踪数据地址可以发现就是用户名的第一个地址

![](https://attach.52pojie.cn/forum/202203/26/005820b2mq0mq0b1qqqqdj.png)

`QF67)6E@9ZA_UP}(MMQO99Q.png` (6.35 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgyNXxjMDIyYjA2OHwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyNXxjMDIyYjA2OHwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 00:58 上传

也就是说这条指令的作用就是 `0x68+0x64=0xcc`，这里的 `0x64` 可以理解成加密过程中的一个运算。

再看下一个函数：

`__rtcHexVarFromVar` 将计算结果转换为 `UNICODE` 形式：

指令 `F8` 到 `403261` 处，就可以看到此时值已经 `add` 后的结果已经变成了 `UNICODE` 形式：

![](https://attach.52pojie.cn/forum/202203/26/010234anf7dbmaay7hofp5.png)

`C(9_%U8BG@63}[Q[N3HOM.png` (1.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwNzgyOHw1MGI3ZWUwMXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyOHw1MGI3ZWUwMXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 01:02 上传

### 拼接生成字符串

循环的最后一部分是把生成的字符串连接出来：

```
0040326C   .  8D4D BC       lea ecx,dword ptr ss:[ebp-0x44]
0040326F   .  8D55 AC       lea edx,dword ptr ss:[ebp-0x54]
00403272   .  51            push ecx           ;Old
00403273   .  8D85 64FFFFFF lea eax,dword ptr ss:[ebp-0x9C]
00403279   .  52            push edx           ;NEW
0040327A   .  50            push eax           ;Serial = strcat(Old, New)
0040327B   .  FF15 84104000 call dword ptr ds:[]       ;  msvbvm60.__vbaVarCat
```

那具体是连接什么呢？

分别跟踪这三个值就会发现一处其实是 0，这让我有点想不明白，直到我做第二次循环才发现这个加密的逻辑：

- 接收用户名字符串，序列号从前四个字符生成
- 将每个字符的 `ASCII` 码值加上 `0x64` 然后把结果转换成 `UNICODE` 形式
- 将所有 `UNICODE` 值拼接在一起

## 写注册机

```
Old = input("请输入您的用户名：")

i = 0

Serial = ""

while i
```

写完试一下：

![](https://attach.52pojie.cn/forum/202203/26/010419k90993lpcszrs5cp.png)

`~L1LYBABYOJSPDNSHPA$W1D.png` (29.76 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgyOXxjYWVjZjY0NXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgyOXxjYWVjZjY0NXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 01:04 上传

![](https://attach.52pojie.cn/forum/202203/26/010510tncvgiimaj219sjp.png)

`@J8JVDYF]GV6IRFAUL)6ORB.png` (10.65 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzgzMHwyNjMyMzYwYXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzgzMHwyNjMyMzYwYXwxNzkzNTc1ODk1fDE2ODA4NnwxNjEwMTA1&nothumb=yes)

2022-3-26 01:05 上传
