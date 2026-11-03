
```c
install_hook_name(sum,int, int a1, int a2,int a3,int a4,int a5,int a6,int a7,int a8,uint64_t a9,uint64_t a10){
    printf("进入到 hook 方法,参数:%d,%d,%d,%d,%d,%d,%d,%d,%llu,%llu \n",a1,a2,a3,a4,a5,a6,a7,a8,a9,a10);
    int ret =   orig_sum(a1,a2,a3,a4,a5,a6,a7,a8,a9,a10);
    printf("调用 原方法获取的结果:%d\n",ret);
    return ret;
}
```

## 栈

首先第一行指令是 `SUB` `SP`, `SP`, `#0x60` 表示 局部变量使用的空间

```
|-60
```

`STP` (Store Pair) 用于储存一对寄存器到 寄存器中

所以 下面这条指令执行后

```
STP             X29, X30, [SP,#0x50]
```

栈应该是这样的:

```
|-60
```

`x29` 和 `x30` 是两个 关键的寄存器。

它们分别代表着 帧指针 `FP`  和 链接指针 `LR`

`LR` 各位肯定是知道的。 `FP` 按照网上的意思是说 它代表着栈低  `SP` 代表着栈顶

## 参数

下面就是传过来的参数 保存在 栈中，只不过要注意的是,它并不是直接使用 `SP` 而是从  栈的 -10 的地方开始计算。 这样可以很方便获取栈中 `a9-a10` 的参数

```
# 从 sp -10 的地方开算
FD 43 01 91                   ADD             X29, SP, #0x50 ; 'P'
# 根据调用约定 x0-x7 额外的参数放入栈中 
# 所以 x9 = a10
A9 0B 40 F9                   LDR             X9, [X29,#0x10]
# x8 = a9
A8 0F 40 F9                   LDR             X8, [X29,#0x18]
# stur 貌似是有符号的 str 指令 
# 所以下面储存  a1-a8 的参数
A0 C3 1F B8                   STUR            W0, [X29,#-4]
A1 83 1F B8                   STUR            W1, [X29,#-8]
A2 43 1F B8                   STUR            W2, [X29,#-0xC]
A3 03 1F B8                   STUR            W3, [X29,#-0x10]
A4 C3 1E B8                   STUR            W4, [X29,#-0x14]
A5 83 1E B8                   STUR            W5, [X29,#-0x18]
A6 43 1E B8                   STUR            W6, [X29,#-0x1C]
A7 03 1E B8                   STUR            W7, [X29,#-0x20]
# 储存 a10 和 a9
E9 17 00 F9                   STR             X9, [SP,#0x28]
E8 13 00 F9                   STR             X8, [SP,#0x20]
```

那么现在换算栈中的样子:

```
|-60
```

## 调用方法

之前的指令应该都只是对标着传参

```c
int fake_sum(sum,int, int a1, int a2,int a3,int a4,int a5,int a6,int a7,int a8,uint64_t a9,uint64_t a10)
```

后面的代码是:

```c
printf("进入到 hook 方法,参数:%d,%d,%d,%d,%d,%d,%d,%d,%llu,%llu \n",a1,a2,a3,a4,a5,a6,a7,a8,a9,a10);
```

来看看对应的 arm 段

```
# 首先准备  x1 ~ x7 的参数 也就是 a1,a2,a3,a4,a5,a6,a7
LDUR            W1, [X29,#-4]
LDUR            W2, [X29,#-8]
LDUR            W3, [X29,#-0xC]
LDUR            W4, [X29,#-0x10]
LDUR            W5, [X29,#-0x14]
LDUR            W6, [X29,#-0x18]
LDUR            W7, [X29,#-0x1C]
# x11 = a8
LDUR            W11, [X29,#-0x20]
# 获取 a9 和 a10
LDR             X10, [SP,#0x28]
LDR             X8, [SP,#0x20]
# 获取栈顶 x9 = sp
MOV             X9, SP
# 多余的参数储存在栈中 
# [sp] = a8
STR             W11, [X9]
# [sp + 0x8] = a9
STR             X10, [X9,#8]
# [sp + 0x10] = a10
STR             X8, [X9,#0x10]
# 获取 字符串的地址 (进入到 hook 方法,参数...)
ADRL            X0, aHookDDDDDDDDLl     ; "进入到 hook 方法,参数:%d,%d,%d,%d,%d,%d,%d,%d,"...
# 调用 printf 并 记录 LR 
BL              printf
```

此时调用后栈应该变成了:

```
|-60                a8
```

## orig_sum

接下来是重点。我们分析下 `orig_sum` 是如何调用原有的方法。

首先我们看下传参调用的过程:

```
# 获取 orig_sum 方法地址放入到 x8 中
ADRP            X8, #_ZL8orig_sum@PAGE  ; orig_sum
LDR             X8, [X8,#_ZL8orig_sum@PAGEOFF] ; orig_sum
# 准备 x0-x7 的参数
LDUR            W0, [X29,#-4]
LDUR            W1, [X29,#-8]
LDUR            W2, [X29,#-0xC]
LDUR            W3, [X29,#-0x10]
LDUR            W4, [X29,#-0x14]
LDUR            W5, [X29,#-0x18]
LDUR            W6, [X29,#-0x1C]
LDUR            W7, [X29,#-0x20]
# 其余的参数放入到 栈中
# x11 = a9
LDR             X11, [SP,#0x28]
# x9 = a10
LDR             X9, [SP,#0x20]
# x10 = sp
MOV             X10, SP
# [sp] = a9
STR             X11, [X10]
# [sp + 8] = a10
STR             X9, [X10,#8]
# 调用 orig_sum
BLR             X8
```

接下来我们进入 `orig_sum` 看看它做了什么:

![](https://attach.52pojie.cn/forum/202305/01/122835v8ffzx6c4o3r4r4z.png)

03.png (40.39 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMTU4M3w5YTUxMTA3ZnwxNzkzNTY5NDg1fDE2ODA4NnwxNzgwNDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTU4M3w5YTUxMTA3ZnwxNzkzNTY5NDg1fDE2ODA4NnwxNzgwNDU4&nothumb=yes)

2023-5-1 12:28 上传

不知道各位熟悉这个代码么？

如果大伙不记得看下面的图, （只看前12个字节）

这个图就是之前 没有hook前 `sum`方法的图。

由于之前的忘记 按 Q转换 并且没有显示字节码。 但我可以肯定告诉你,它们是一样的

我把 `orig_sum` 截取出来大伙看看

```
FF 03 01 D1                   SUB             SP, SP, #0x40 ; '@'

E9 23 40 F9                   LDR             X9, [SP,#0x40]

E8 27 40 F9                   LDR             X8, [SP,#0x48]

                              _753EDCE894

51 00 00 58                   LDR             X17, =0x753EDCE894

20 02 1F D6                   BR              X17
```

前面12 个字节都好说,就是之前 `sum` 被替换成 跳转指令之前的样子

下面是目前被替换成的代码:

```
# 获取 fake_sum 地址 放入 x17
11 00 00 90 31 A2 28 91       ADRL            X17, _ZL8fake_sumiiiiiiiimm
# 绝对地址跳转 到 fake_sum                                        
20 02 1F D6                   BR              X17
```

那后面的两条指令是跳到哪里呢？

```
51 00 00 58                   LDR             X17, =0x753EDCE894

20 02 1F D6                   BR              X17
```

我们在 `ida pro` 按 `G` 跳转下:

![](https://attach.52pojie.cn/forum/202305/01/122837bjjfb2mcfpw2hpb2.png)

04.png (75.15 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMTU4NHxjYWUwOTRiMHwxNzkzNTY5NDg1fDE2ODA4NnwxNzgwNDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTU4NHxjYWUwOTRiMHwxNzkzNTY5NDg1fDE2ODA4NnwxNzgwNDU4&nothumb=yes)

2023-5-1 12:28 上传

可以很明显的发现  `=0x753EDCE894` 地址就是 原来 `sum`  的 12字节 后面的地方。

那么当执行完 `sum` 方法后,又如何跳到 我们的 `fake_sum` 接着往下执行呢？

之前说过 , `RET` 指令会根据 `x30`(`LR`) 链接寄存器的值来跳转。

所以这也是为什么 `fake_sum` 在调用 `orig_sum` 的时候 选择了 `BLR` 指令。

而`orig_sum` 则只使用 `BR` 指令

下面是摘取的 `fake_sum` 和 `orig_sum` 的片段:

```
# fake_sum 调用 orig_sum 的片段
ADRP            X8, #_ZL8orig_sum@PAGE  ; orig_sum
LDR             X8, [X8,#_ZL8orig_sum@PAGEOFF] ; orig_sum
.....
BLR             X8
```

```
# orig_sum 跳到 sum 的片段
LDR             X17, =0x753EDCE894

BR              X17
```

## RET

剩余 `fake_sum` 的返回也没啥好说的。 就是正常返回。

```
# 将返回值 放入 x0 
E0 1F 00 B9                   STR             W0, [SP,#0x1C]
# 通过 printf 打印下返回值
E1 1F 40 B9                   LDR             W1, [SP,#0x1C]
C0 FF FF 90 00 EC 27 91       ADRL            X0, unk_753EDC69FB
76 79 00 94                   BL              printf

E0 1F 40 B9                   LDR             W0, [SP,#0x1C]
# 还原 x29 x30 寄存器
FD 7B 45 A9                   LDP             X29, X30, [SP,#0x50]
# 还原堆栈
FF 83 01 91                   ADD             SP, SP, #0x60 ; '`'
# 返回 x0
C0 03 5F D6                   RET
```

# 总结

基于 `dobby` 对于 `function inline hook` 所做的事情简单来说就 2个 步骤

- 
创建 `orig_sum` 的内存 储存 `sum` 前12个 字节 并携带 `BR` 跳回到后12个字节的地方

- 
修改 `sum` 的前 12 个字节 跳到 `fake_sum` 中
