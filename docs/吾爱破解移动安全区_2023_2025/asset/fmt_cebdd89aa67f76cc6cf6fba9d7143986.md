
# V 标志与溢出
`V` 标志表示结果的符号位是否跟两个寄存器不同，我举一个例子：

```
# 假设寄存器的大小只有 1 字节 
# r0 r1 的二进制数都是 b1000 0000 = -128
ADD r0, r0 , r1
# r0 结果按道理是 1 0000 0000  可是我说假设是由 1个字节
# r0 被截取后 变成了 0
# 此时代表了溢出 V = 1
```

## 与 LE 的关系
那这跟我们 `LE` 有什么关系呢？
首先 `N = 1` 一般来说肯定是 `x0` 小于 `x1` 的
但由于符号位问题可能会溢出成正数。
如下：

```
# 假设 寄存器还是只有 1个 字节

# r0 是 b1000 0000 = -128 && r1 = b0000 0001 = 1

subs r0, r0 , r1 

# 由于寄存器最大是 1字节 负数最大只能是 -128

# b1000 0000 -  b0000 0001 = b0111 1111 = 127

# 此时变成了负数且溢出 此时 N = 0 V = 1

# 但 r0 实际 比 r1 小
```

所以 `N != V` 用于有符号位的大小判断。

### BGT 大于跳转
`3.BGT` (`greater than`) 大于跳转 判断标志位 `N = V`
正常不溢出且 `r0 - r1` 不会成负数，那么就代表 `r0 > r1`

## 条件跳转
下面是最后剩余片段，我们只看它是如何进行条件跳转的

```
# 1. 从栈中取出 sum
LDR             W8, [SP,#0xC]
# 2. 跟 10 做比较 并影响 psr 状态寄存器
SUBS            W8, W8, #0xA
# 3. 如果 N!=V 也就是说 sum =0 直接跳到 loc_7770038748 上
B               loc_7770038748

; ---------------------------------------------------------------------------
# 5. 跳到 loc_7770038748 标记
loc_7770038748                         
LDRSW           X8, [SP,#0xC]
LDR             X9, [SP]
ADD             X8, X8, X9
ADRP            X9, #0x777003A000
STR             X8, [X9,#0xAC0]
B               loc_7770038760

; ---------------------------------------------------------------------------
# 6. 跳到 loc_7770038760 标记
loc_7770038760                        

ADRP            X8, #0x777003A000
LDR             X8, [X8,#0xAC0]
MOV             W0, W8
ADD             SP, SP, #0x40 ; '@'
RET
```

对应着 C++ 的代码：

```
if (sum > 10)

{

    //6. 测试全局变量

    _global_sum = sum + sum2;

}

    // 7. 测试返回值

    return _global_sum;
```

## 全局变量
在回顾一下目前栈储存的值：

```
-40 |   sum2  
```

结果值肯定是 `sum > 10` 的，我们只看摘取的一部分：

```
# 反之  sum >=0 直接跳到 loc_7770038748 上

B               loc_7770038748

; ---------------------------------------------------------------------------

# 跳到 loc_7770038748 标记

loc_7770038748                         

LDRSW           X8, [SP,#0xC]

LDR             X9, [SP]

ADD             X8, X8, X9

ADRP            X9, #0x777003A000

STR             X8, [X9,#0xAC0]

B               loc_7770038760
```

### LDRSW
`LDRSW` 是 `LDR` (`Load register`) 的扩充指令，也是从内存中取出数据到寄存器中。
后面的 `SW` 代表着 `signed word` (with optional Extend, 意思是带符号扩充到 64 位。也就是说 `32`位的数据扩充到 `64`位最高位符号不变。

```
# 获取 sum 并转换 64 位 放入 x8 中
LDRSW           X8, [SP,#0xC]
```

### ADD

```
# 获取 sum2 放入 x9 中
LDR             X9, [SP]

# sum + sum2
ADD             X8, X8, X9
```

### ADRP / PC / STR
`adrp` 获取基于 `pc` 和目标偏移的 `4kb` 对齐地址
`pc` 是当前指令执行的地址
`str` 是储存寄存器到内存地址
整合起来的意思是：

```
# 将当前 pc 内存对齐的地址 到 x9 中
ADRP            X9, #0x777003A000
# [x9 + 0xAC0] = sum + sum2
# [x9 + 0xAC0] 就是全局变量的指针
STR             X8, [X9,#0xAC0]
```

我们看看 `0x777003A000 + 0xac0` 在内存中是什么样子的

![](https://attach.52pojie.cn/forum/202304/28/163222cew7ae6nez3i7cp6.png)

01.png (31.97 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMTA4N3xlNTBiMmUxYXwxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTA4N3xlNTBiMmUxYXwxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes)

2023-4-28 16:32 上传

由于是小端排序，且是 `64` 位的 `uint64`：

```
     42 00 00 00 00 00 00 00

转换  00 00 00 00 00 00 00 42     
```

`0x42` 的结果是 `66` 就是我们 `sum + sum2` 的结果。
它们换算成代码就是：

```
_global_sum = sum + sum2;
```

## RET
最后就讲解下返回值了
`RET` 实际类似与我们使用 `return` 命令

```
return _global_sum;
```

来看看最后的汇编做了什么：

```
# 1. 获取全局变量 _global_sum
ADRP            X8, #0x777003A000
LDR             X8, [X8,#0xAC0]
# 2. 将全局变量放在 x0 中用于返回
MOV             W0, W8
# 3. 释放局部变量空间
ADD             SP, SP, #0x40 ; '@'
# 4. 返回到 x30 寄存器指向的地址
RET
```

此时的栈空间就又变成原始的：

```
.....(这些被释放掉了)  
0   |   a9      
```

`RET` 指令的话依赖于 `X30` 寄存器。也就是 `linker` 地址。
一般在使用 `BL` 会计算下一条指令的位置，用图展示方便大伙分析：

![](https://attach.52pojie.cn/forum/202304/28/163224x7ohn7876l6n6j77.png)

02.png (63.57 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxMTA4OHw5ZjI3MTA2ZHwxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTA4OHw5ZjI3MTA2ZHwxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes)

2023-4-28 16:32 上传

用鼠标点击下 `x30` 寄存器跳过去

![](https://attach.52pojie.cn/forum/202304/28/163226ng7wfbb8wyiybb58.png)

03.png (66.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMTA4OXw0M2EzMTY4YXwxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTA4OXw0M2EzMTY4YXwxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes)

2023-4-28 16:32 上传

你会发现 `ida pro` 给你翻译成数据了
此时你按 `C` 翻译成代码

![](https://attach.52pojie.cn/forum/202304/28/163228c96ukf53xtvf4j4h.png)

04.png (68.66 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxMTA5MHxmMDYyMTNmY3wxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMTA5MHxmMDYyMTNmY3wxNzkzNTY5NDkwfDE2ODA4NnwxNzc5NzU0&nothumb=yes)

2023-4-28 16:32 上传

到此你就会发现它就是我们 `BL sum` 指令下的位置
下一章我们就讲解 `dobby` `inline Hook`
