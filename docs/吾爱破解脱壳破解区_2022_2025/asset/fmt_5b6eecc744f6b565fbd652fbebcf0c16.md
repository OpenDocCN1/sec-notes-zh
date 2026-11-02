
# [原创] 新手学习Vmp之识别handler

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1814843
- 抓取时间: 2026-11-01T22:49:00

## 新手学习Vmp之利用ghidra识别handler

最近也是在研究vmp，记录一下过程，没什么技术含量，仅提供一种思路作为参考。

这里还是以vmp2为例，为什么研究vmp2而不是vmp3呢？一方面我认为是要循序渐进，在掌握了vmp2的基础上，再去分析vmp3应该是能够容易不少的，另一方面也不排除有软件依旧会采用vmp2进行加密。

之前分析vmp用unicorn动态跟踪生成了一个动态流程图，可以参考上一篇帖子https://www.52pojie.cn/thread-1798622-1-1.html

vmp2在画出控制流程图后架构基本一目了然，通过一些简易的规则很快就能识别出handler块、handler分发块这些东东，这里就不谈了。

对于识别出handler，我认为关键点在于能对handler中的无用指令进行反混淆，在提炼出核心指令后，就好处理了。这里我并没有使用符号执行(主要是没发现好用的C++符号执行库额)，而是采用另一款开源的ghidra反编译库。

Ghidra设计了一套中间指令，能够将汇编指令转换为中间码，并对其进行SSA、死代码消除等一系列化简操作，这和反混淆本质上是一样的，我们只需对其稍加进行改造，像vmp这种乱七八糟的二进制指令，Ghidra也能处理。

例如vmp handler有一个`vAdd4`指令，原始汇编如下:

```
004D4DD9    66:F7D0         not ax 
004D4DDC    66:0FBDC1       bsr ax,cx
004D4DE0    8B45 00         mov eax,dword ptr ss:[ebp]
004D4DE3    60              pushad
004D4DE4    66:F7C4 C21F    test sp,0x1FC2
004D4DE9    83EC E0         sub esp,-0x20
004D4DF2    0145 04         add dword ptr ss:[ebp+0x4],eax
004D4DF5    60              pushad
004D4DF6    9C              pushfd
004D4DF7    885C24 04       mov byte ptr ss:[esp+0x4],bl
004D4DFB    9C              pushfd
004D4DFC    8F4424 20       pop dword ptr ss:[esp+0x20]
004D4E00    882424          mov byte ptr ss:[esp],ah
004D4E03    FF7424 20       push dword ptr ss:[esp+0x20]
004D4E07    8F45 00         pop dword ptr ss:[ebp]
004D4E0A    66:C74424 10 A5 mov word ptr ss:[esp+0x10],0x71A5
004D4E11    60              pushad
004D4E12    9C              pushfd
004D4E13    C70424 2BFA35ED mov dword ptr ss:[esp],0xED35FA2B
004D4E1A    8D6424 48       lea esp,dword ptr ss:[esp+0x48]
```

Ghidra反编译后生成如下`Pcode`:

```
0x004d4de0:1:        u0x00007a00(0x00000002:1) = *(ram,EBP(i))

0x004d4df2:162:        u0x00001d00(0x00000007:162) = EBP(i) + #0x1(*#0x4)

0x004d4df2:2f:        u0x00007a00(0x00000007:2f) = *(ram,u0x00001d00(0x00000007:162))

0x004d4df2:30:        CF(0x00000007:30) = CARRY4(u0x00007a00(0x00000007:2f),u0x00007a00(0x00000002:1))

0x004d4df2:31:        u0x00007a00(0x00000007:31) = *(ram,u0x00001d00(0x00000007:162))

0x004d4df2:32:        OF(0x00000007:32) = SCARRY4(u0x00007a00(0x00000007:31),u0x00007a00(0x00000002:1))

0x004d4df2:33:        u0x00007a00(0x00000007:33) = *(ram,u0x00001d00(0x00000007:162))

0x004d4df2:34:        u0x00007a00(0x00000007:34) = u0x00007a00(0x00000007:33) + u0x00007a00(0x00000002:1)

0x004d4df2:35:        *(ram,u0x00001d00(0x00000007:162)) = u0x00007a00(0x00000007:34)

0x004d4df2:36:        u0x00007a00(0x00000007:36) = *(ram,u0x00001d00(0x00000007:162))

0x004d4df2:163:        u0x10000080(0x00000007:163) = (cast) u0x00007a00(0x00000007:36)

0x004d4df2:37:        SF(0x00000007:37) = u0x10000080(0x00000007:163) 
```

可以说是几乎很完美地去除了无效的指令了额，看不懂没关系，我们直接提出上面`Pcode`中的出现的地址和其对应的指令:

```
004D4DE0    8B45 00         mov eax,dword ptr ss:[ebp]
004D4DF2    0145 04         add dword ptr ss:[ebp+0x4],eax
004D4DFB    9C              pushfd
004D4E07    8F45 00         pop dword ptr ss:[ebp]
```

这样会清楚一些，针对上面生成的结果，那么识别Handler就有两种识别方案，复杂一点的是编写`Pcode`规则进行识别，通过遍历`Pcode`来追踪值之间的关系再进行特征匹配；追求简单点就直接对汇编指令进行规则识别，不过缺点就是部分关键汇编指令也可能被Ghidra优化掉，会丢失一些准确度，但是应该也是能满足大部分场景了。

这里就介绍一下采取简单方案大致的识别步骤，我们将化简的指令再进行细分，例如`mov xxx,dword ptr ss:[ebp]`，这类指令可标记为`ReadVmStack0_4`，后面的两个数字，一个是代表`ebp`的寄存器偏移，一个是代表大小。

`add dword ptr ss:[ebp+0x4],xxx`，这类指令可标记为`Add_Ebp4_4`

由于我们做标记的指令都是化简之后的指令，已经是关键指令了，因此当出现这两个标记的时候，即可直接认为该vmp handler为`vAdd4`
