
# TVM 逆向分析

## 初始判断与 `handle` 跳转

`R8b - 1` 如果大于 `0xC8`，就跳转到 `1400D9954`，说明这是未知的 `tvmopcode`，出错。`1400D9954` 是 `int3` 指令。

然后 `handleTable` 放入 `R9`，取表项 `[R9 + R8 *8]`，即为这个 `handle` 的偏移，加上基址 `0x140000000`，即为这个 `handle` 的实际处理地址：`1400d60f2`，通过 `jmp r8` 跳转过去。

（可以看到取得第一个 `tvmopcode` 是 `0xe8`，和上面图中的是一样的）

## 6. 小总结

### 1. 从 `tvmopcode` --> `handle` 的方式

```
handleTableBase = 0x1400db5aa
Dllbase = 0x140000000
handle = Dllbase + qword[(tvmopcode^0x5d - 1)*8 + handleTableBase]
```

注意，`tvm` 有多张 `handleTable`，但是里面的内容都是一样的。所以拿到一张表就行了。

### 2. `handle` 的个数

从 `tvmopcode^0x5d - 1` 可以推测一共有 `0xc8`（`0~0xc7`）个 `tvmopcode`，例：

```
handleTable的第 0x00 项: tvmopcode = (0x00 + 1) ^ 0x5d = 0x5c

......

handleTable的第 0x3c 项: tvmopcode = (0x3c + 1) ^ 0x5d = 0x60

......

handleTable的第 0xc7 项: tvmopcode = (0xc7 + 1) ^ 0x5d = 0x95
```

### 3. 导出所有 `handle`

现在我们知道，`tvm` 的 `handleTable` 有 `0xc8` 个有效项，我们就可以遍历 `handleTable`，并且静态跟踪出 `handle`。看它是如何处理的：

导出 `handle` 代码（`idapython/TVMHandleOut.py`）

补充：虽然 `handle` 有 `0xc8` 个有效项，但是很多是重复的，是留作拓展用的，真正有效的 `handle` 就 80 个：

左边是 `handle` 处理地址，右边是 `tvmopcode`

```
1400d9954 : 9a  9b  98  99  9c  9d  e2  e3  e0  e1  e6  e7  e4  eb  ec  ed  f2  f3  f0  f1  f6  f7  f5  fa  fb  f9  fe  ff  c2  c0  c1  c6  c7  c4  c5  c8  ce  cf  cc  cd  db  d8  de  df  dc  dd  23  26  27  24  2a  28  2e  2c  2d  32  33  30  31  36  37  34  35  3a  3b  38  39  3e  3f  3d  02  03  00  07  04  05  0a  0b  08  0e  0f  0d  12  13  10  17  14  15  1a  1b  18  1e  1f  1c  63  61  66  67  64  6b  69  6f  6c  71  76  77  74  7b  79  7e  7f  7c  53  50  55  5b  58  59  5e  5f  5c
1400d8eb1 : 5a
1400d5b5a : 54
1400d5cd8 : 57
1400db3d3 : 56
1400d761b : 51
1400da3e5 : 52
1400dabb0 : 4d
1400d8196 : 4c
1400d6c8c : 4f
1400d82fa : 4e
1400d6e7d : 49
1400d7172 : 48
1400d9374 : 4b
1400daa8e : 4a
1400db0f9 : 45
1400dae51 : 44
1400d8be2 : 47
1400d946b : 46
1400d995a : 41
1400daf90 : 40
1400d7a8a : 43
1400d5e10 : 42
1400d7258 : 7d
1400daecb : 78
1400d5d02 : 7a
1400d974c : 75
1400d7f26 : 70
1400d8e1b : 73
1400d8dbf : 72
1400d8362 : 6d
1400d6fc9 : 6e
1400d7c1f : 68
1400d63cd : 6a
1400d9abb : 65
1400d8431 : 60
1400d7797 : 62
1400da161 : 1d
1400d7313 : 19
1400d92b0 : 16
1400d9c5c : 11
1400dab1e : 0c
1400d86e2 : 09
1400d7352 : 06
1400dad5e : 01
1400d9524 : 3c
1400dad2c : 2f
1400d8b8d : 29
1400d7d39 : 2b
1400d8f7b : 25
1400d696f : 21
1400d71f5 : 20
1400d71b3 : 22
1400d770e : d9
1400da48c : da
1400d74d2 : d5
1400d70ee : d4
1400d95ad : d7
1400d6f6c : d6
1400d84d0 : d1
1400da357 : d0
1400d78f9 : d3
1400d7998 : d2
1400d8561 : c9
1400d624c : cb
1400d9169 : ca
1400d64ac : c3
1400d5c7b : fd
1400d9dd5 : fc
1400d6913 : f8
1400dafcf : f4
1400d7bc7 : ef
1400d7fc4 : ee
1400d9446 : e9
1400d60f2 : e8
1400d69e8 : ea
1400d9584 : e5
1400da10c : 9f
1400d8e89 : 9e
1400d8cdd : 95
```

### 4. 全部 `handle` 跟踪

并在当前文件夹 `/handleout` 文件夹内，输出全部 `handle` 静态跟踪（运行脚本的同时会对 `handle` 去简单的混淆）：

（刚好 80 个不相同的 `handle`，文件名用 `tvmopcode`）

![](https://attach.52pojie.cn/forum/202305/26/004816qiuppdpmuwzwapl8.png)

`4全部handle跟踪.png`（53.12 KB，下载次数：1）

[https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM0OXw0NmI2YjU4N3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM0OXw0NmI2YjU4N3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 00:48 上传

一共有 80 个不同功能的 `tvmopcode`，为节省篇幅，我这里挑一个常见的讲解，全部的 `handle` 分析，其余的全部放在（`handle_out/`）

（那个特别多的是 `int3`，应该是预留以后更新用的）

我静态跟踪 `handle` 是以 `jnb` 为结尾的（就是判断是否大于 `0xc8` 后的 `jnb`），所以跟踪文件后半段有一些不用看。

```
0xe8 p_a b_ULONG64

0x3F26

*(PULONG64)p_a = b_ULONG64;

v_mov_iregll_ll

----------------------------------------//虚线以上是分析出来的

0x1400d60f5 : mov     r9, [rbp+8]

0x1400d6102 : mov     r8w, [r9]                       这里取2字节 (取虚拟寄存器都是取2字节)

0x1400d6106 : xor     r8w, 3F26h                      异或 3F26(不同的handl异或的值不同) 得到 a,

0x1400d610c : mov     rdx, 0F84A86395161A270h         所以就是取虚拟机寄存器  V_REG_P + a

0x1400d6116 : not     rdx

0x1400d6119 : jmp     loc_1400D9077

0x1400d9078 : lea     rdx, [r10+rdx]

0x1400d907c : movzx   r8, r8w

0x1400d9080 : mov     rcx, 7B579C6AE9E5D8Eh

0x1400d908a : not     rcx

0x1400d908d : add     r8, rcx

0x1400d9090 : lea     r8, [rdx+r8]                    p_a = r8 = V_REG_P + a

0x1400d9094 : lea     r9, [r9+2]                      V_RIP+=2

0x1400d909b : mov     rdx, [r9]                       b_ULONG64  从tvm指令表中取8字节

0x1400d909e : jmp     loc_1400DA2DD

0x1400da2ed : mov     [r8], rdx                       放入 [r8],即放入 p_a

0x1400da2f0 : jmp     loc_1400DA9AC

0x1400da9b4 : lea     r9, [r9+8]                      V_RIP+=8

0x1400da9be : mov     [rbp+8], r9                     V_RIP放回[rbp+8]

0x1400da9c8 : jmp     loc_1400DAC79

0x1400dac7c : mov     r9, [rbp+8]                     这里以下属于下一个handle分发，不用看

0x1400dac89 : mov     r8b, [r9]

0x1400dac8c : xor     r8b, 5Dh

0x1400dac90 : mov     rdx, 0D3676A56DAFF3C65h

0x1400dac9a : jmp     loc_1400D7763

0x1400d7764 : not     rdx

0x1400d7767 : lea     rdx, [r9+rdx]

0x1400d776b : mov     r9, 2C9895A92500C398h

0x1400d7775 : not     r9

0x1400d7778 : lea     r9, [rdx+r9]

0x1400d777f : jmp     loc_1400D5B91

0x1400d5b95 : mov     [rbp+8], r9

0x1400d5b9e : movzx   r8, r8b

0x1400d5ba2 : sub     r8, 1; switch 200 cases

0x1400d5ba6 : jmp     loc_1400D98A8

0x1400d98aa : cmp     r8, 0C8h

0x1400d98b1 : jnb     def_1400D655C;
```

我对 `tvmAsm` 的命名规则：

```
v_opcode_op0_op1_op2_....._opn

例如:
v_mov_iregw_iregw :
从虚拟寄存器op1 取 2字节 放入虚拟寄存器op0
w 表示 USHORT

v_mov_ipreg_iregl :
从虚拟机寄存器op1 取4字节，放入以虚拟机寄存器op0作为地址的空间，有点类似与:
mov [reg],reg

v_mov_iregb_b :
从tvm指令表中取1字节放入虚拟机寄存器op0中,类似于Asm中的立即数赋值:
mov reg,0xff

v_and_oregl_iregl_iregl_oregl:(i 就是 in ，o 就是 out 的意思)
取虚拟寄存器op1 和 虚拟寄存器op2进行与运算，并将结果放入虚拟寄存器op0，与运算后的 rflag 放入 虚拟寄存器op3
l 表示 ULONG32

v_cmp_iregll_iregll_oregl:
即为 cmp 虚拟寄存器op0,虚拟寄存器op1 ，cmp后的rflag 放入 虚拟寄存器op2
ll 表示 ULONG64
```

我这里直接将全部 `tvmAsm` 展示出来：（参考 `idapython/deTvm.py`，`tvmHandleTableInit()`）

```
TVMTABEL.append("v_sar_oregll_iregll_iregb_oregl",0x01,0xF8BE)

    TVMTABEL.append("v_or_oregll_iregll_iregll_oregl",0x1D,0x4AA7)

    TVMTABEL.append("v_mov_iregll_iregl",0x2B,0xBF3E)

    TVMTABEL.append("v_movzx_iregl_iregb",0x2F,0x7EE9)

    TVMTABEL.append("v_ror_oregb_iregb_iregb_oregl",0x3C,0xF8E1)

    TVMTABEL.append("v_mov_iregl_iregl",0x4A,0x564B)

    TVMTABEL.append("v_mov_iregw_iregw",0x4B,0xD916)

    TVMTABEL.append("v_add_oregb_iregb_iregb_oregl",0x4C,0xDD9D)

    TVMTABEL.append("v_add_oregll_iregll_iregll",0x4D,0x477D)

    TVMTABEL.append("v_add_oregl_iregl_iregl_oregl",0x4E,0xA9C7)

    TVMTABEL.append("v_add_oregw_iregw_iregw_oregl",0x4F,0x82BC)

    TVMTABEL.append("v_sub_oregl_iregl_iregl_oregl",0x5A,0xC198)

    TVMTABEL.append("v_sar_oregl_iregl_iregb_oregl",0x06,0x8374)

    TVMTABEL.append("v_and_oregl_iregl_iregl_oregl",0x6A,0x5CF0)

    TVMTABEL.append("v_and_oregll_iregll_iregll_oregl",0x6D,0xD9B1)

    TVMTABEL.append("v_and_oregl_iregl_iregl",0x6E,0xA1CE)

    TVMTABEL.append("v_xor_oregl_iregl_iregl_oregl",0x7A,0xD8ED)

    TVMTABEL.append("v_mov_ipreg_iregll",0x7D,0xD878)

    TVMTABEL.append("v_shr_oregll_iregll_iregb_oregl",0x09,0x9D87)
```
