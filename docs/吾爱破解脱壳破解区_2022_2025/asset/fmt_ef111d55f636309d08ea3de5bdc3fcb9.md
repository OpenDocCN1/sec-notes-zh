
```
00000214C437135E   | 48:83F9 FE                  | cmp rcx,FFFFFFFFFFFFFFFE                    |
00000214C4371362   | 74 15                       | je 214C4371379                              |
00000214C4371364   | 6548:8B0425 30000000        | mov rax,qword ptr gs:[30]                   |
00000214C437136D   | 8B58 40                     | mov ebx,dword ptr ds:[rax+40]               |
00000214C4371370   | E8 FB1D0000                 | call 214C4373170                            |
00000214C4371375   | 3BD8                        | cmp ebx,eax                                 |
00000214C4371377   | 75 04                       | jne 214C437137D                             |
00000214C4371379   | 33C0                        | xor eax,eax                                 |
00000214C437137B   | EB 11                       | jmp 214C437138E                             |
00000214C437137D   | 44:8BCE                     | mov r9d,esi                                 |
00000214C4371380   | 4D:8BC6                     | mov r8,r14                                  |
00000214C4371383   | 8BD5                        | mov edx,ebp                                 |
00000214C4371385   | 48:8BCF                     | mov rcx,rdi                                 |
00000214C4371388   | FF15 62630000               | call qword ptr ds:[214C43776F0]             |
00000214C437138E   | 48:8B5C24 30                | mov rbx,qword ptr ss:[rsp+30]               |
00000214C4371393   | 48:8B6C24 38                | mov rbp,qword ptr ss:[rsp+38]               |
00000214C4371398   | 48:8B7424 40                | mov rsi,qword ptr ss:[rsp+40]               |
00000214C437139D   | 48:8B7C24 48                | mov rdi,qword ptr ss:[rsp+48]               |
00000214C43713A2   | 48:83C4 20                  | add rsp,20                                  |
00000214C43713A6   | 41:5E                       | pop r14                                     |
00000214C43713A8   | C3                          | ret                                         |
```

直接 `patch` 调用 `ZwSetInformationThread` 的函数，组织好返回值跳到末尾即可绕过这处反调。

回到脱壳后的 `main` 函数，程序先调用 `sub_140007940` 创建了几个线程，然后调用 `DialogBoxParamW` 创建了一个窗口。跟入 `DialogBox` 的 `handle` 函数（`sub_140009EC0`），它尾调用到 `sub_140009240`，这里基本是响应用户请求，构造消息发给其它线程处理，并通过 `ConditionVariable` 唤醒线程。

接下来看线程入口函数 `sub_140007B50`，基本一个大循环：等待指定的 `ConditionVariable` 被唤醒、调用某结构体特点偏移的处理函数、继续 `sleep` 直到下一次被唤醒。

在 `140007C4D` 处下断点，跟入找到每个消息的处理函数。发现 `sub_140008A80` 对用户输入作 `check`，是一个核心函数。期内又构造了四个消息请求由其它线程处理核心的加解密逻辑，依次实现如下功能：

- `unhex` 密码；
- 解密并 `unpad` 密码，得到的 20 字节视为这样一个结构体：`8 bytes time | 8 bytes uid | 4 bytes hash`；
- 对解密得到的 20 字节的前 16 字节作哈希操作，最终得到 4 字节哈希结果 `H`；
- 获取由当前时间计算得到的量（半小时更新一次）：`T = (time() // 1800) * 1800`；

最终 `check` 逻辑：检查输入结构体中的时间和 `T` 是否相同，输入结构体中的 `uid` 和 `DialogBox` 里用户 `uid` 是否相同，输入结构体里的哈希和计算得到 `H` 是否相同。

hash 算法为魔改版 `md5`，魔改点为在填充时给 `msg_len_in_bits` 值加了 2。

## EXP

```
import time
from arc4 import ARC4

def p64(v):
    return int.to_bytes(v, 8, "little")
def p32(v):
    return int.to_bytes(v, 4, "little")
def toi32arr(data):
    return [int.from_bytes(data[i:i+4], "little") for i in range(0, len(data), 4)]
def fri32arr(data):
    return b"".join([int.to_bytes(a, 4, "little") for a in data])

def create_msg(t, u, p):
    return p64(t) + p64(u) + p32(p) + b"\x04" * 4

rc4_key = lambda c: int.to_bytes(c, 8, "little") + bytes.fromhex('E87964BA0EF1B4A8A175E934E143DFBAF08755F4C18D5AA6')

def enc(data):
    assert len(data) % 8 == 0
    mask = (1> 5)) ^ (key[0] + 16 * inp[0])
            inp[1] &= mask
            inp[0] += (inp[1] - acc) ^ (key[3] + (inp[1] >> 5)) ^ (key[2] + 16 * inp[1])
            inp[0] &= mask
        assert acc == 0x4E516498
        inpall[i*2:i*2+2] = inp
    return fri32arr(inpall)

# https://github.com/Utkarsh87/md5-hashing/blob/master/md5.py
from md5 import *

def epad(msg):
    msg_len_in_bits = (8*len(msg)) & 0xffffffffffffffff
    msg.append(0x80)
    while len(msg)%64 != 56:
          msg.append(0)
    msg_len_in_bits += 2    # !!!
    msg += msg_len_in_bits.to_bytes(8, byteorder='little') # little endian convention
    return msg

t = int((time.time() // 1800) * 1800)
uid = 0

salt = bytes.fromhex('3532706F6A6965203230323520E788B1E9A39EE79A84E78CAB00')
m = salt + create_msg(t, uid, 0)[:0x10] * 10
m = epad(bytearray(m))
processed_msg = processMessage(m)
digest = int.to_bytes(processed_msg, 16, "little")
p = digest[15] + (digest[11]
```

## 3> 6

这个题 `Windows` 版应该可以直接用 `upx` 脱壳。里面是一个简单的 `vm`，`vm` 的 `opcode` 指令数目也不多。我的解法是用 `R1+Python` 重新实现了一遍 `vm`，打印 `trace`，然后一点点看 `trace` 完成逆向。

输入 `flag` 括号内的每段先 `base36` 解码，然后和 `uid` 一起算了一个哈希（看到其它wp说是 `crc32`），要求哈希是一个指定常量。很容易逆向求得一组输入：

### EXP

```
def fc128(a0, a1):
    a0 &= 0xff
    v1 = a0 ^ a1
    for i in range(7, -1, -1):
        f = v1 & 1
        v1 >>= 1
        v1 ^= 0xedb88320 if f else 0
    return v1

def fc0b6(uid):
    # 202552pojieafdm2025
    v1 = 0xffffffff
    v1 = fc128(0x32, v1)
    v1 = fc128(0x30, v1)
    v1 = fc128(0x32, v1)
    v1 = fc128(0x35, v1)
    # print(hex(v1))
    v2 = v1
    v2 = fc128(uid >> 24, v2)
    v2 = fc128(0x35, v2)
    v2 = fc128(0x32, v2)
    v2 = fc128(0x70, v2)
    v2 = fc128(0x6f, v2)
    v2 = fc128(0x6a, v2)
    v2 = fc128(0x69, v2)
    v2 = fc128(0x65, v2)
    # print(hex(v2))
    v2 = fc128(uid >> 16, v2)
    v2 = fc128(0x61, v2)
    v2 = fc128(0x66, v2)
    v2 = fc128(0x64, v2)
    v2 = fc128(0x6d, v2)
    # print(hex(v2))
    v2 = fc128(uid >> 8, v2)
    v2 = fc128(0x32, v2)
    v2 = fc128(0x30, v2)
    v2 = fc128(0x32, v2)
    v2 = fc128(0x35, v2)
    # print(hex(v2))
    v2 = fc128(uid, v2)
    # print(hex(v2))
    return ((~v2) & 0xffffffff) | 0x80808080

def fc089p(data):
    return data[:5] == b"flag{" and data[0x1c] == b"}"[0]

def fc003(tab, inp, off, k):
    vv = 0
    for i in range(5):
        v = tab[inp[off + i]]
        assert v != 0
        vv *= 0x24
        vv += v - 1
    if vv >> 25:
        vv += 1
        vv %= (off & 0xff) + k * 0x13541
    else:
        vv |= 1
    return vv

tab = bytearray(b"\x00"*0x100)
tab[0x30:0x38] = int.to_bytes(0x40C132115100A09, 8, 'little')
tab[0x38:0x40] = int.to_bytes(0x1C11, 8, 'little')
tab[0x40:0x48] = int.to_bytes(0x623020D200F0300, 8, 'little')
tab[0x48:0x50] = int.to_bytes(0x12081916010E141B, 8, 'little')
tab[0x50:0x58] = int.to_bytes(0x51A071E0B24171F, 8, 'little')
tab[0x58:0x60] = int.to_bytes(0x221D18, 8, 'little')

def fc089(uid, passwd):
    kk = fc0b6(uid)
    v = 0
    for i in range(4):
        v |= fc003(tab, passwd, 5 + 6 * i, kk & 0xff)
        kk >>= 8
    v -= 1
    v &= 0xffffffff
    return v ^ 0xc15303fb

def rfc089(uid, expected):
    kk = fc0b6(uid)
    # print(hex(kk))
    v = expected ^ 0xc15303fb
    v += 1
    v &= 0xffffffff
    # print(hex(v))
    flag = b"flag{"
    for i in range(4):
        off = 5 + 6 * i
        k = kk & 0xff
        m = (off & 0xff) + k * 0x13541
        e = (((1 >= 8
    return flag[:-1] + b"}"

uid = 0
expected = 0x3EACFC04
inp = rfc089(uid, expected)
print(inp)
r = fc089(uid, inp)
print(hex(r), r == expected)
```

## 3> 7

`First` 数组里有两个有用的初始化函数。作了简单的反调和字符串解密。反调函数很容易识别并 `patch` 掉，字符串解密也容易静态实现。

实现一些字符串解密的 `ida` 脚本：

```
import ida_bytes
def patch_add(addr, size, value):
    org = ida_bytes.get_bytes(addr, size)
    p = bytes([(a+value)&0xff for a in org])
    ida_bytes.patch_bytes(addr, p)
    print(f"done {hex(addr)} {hex(size)} {value}")

patch_add(0x140026ad0, 8, 0x10)
patch_add(0x140026ac4, 6, 0x10)
patch_add(0x140026aac, 6, 0x10)
patch_add(0x140026A30, 0x11, 0x10)
patch_add(0x140026A80, 0x29, 0x10)
patch_add(0x140026B28, 0x11, 0x10)
patch_add(0x140026B40, 13, 0x10)
patch_add(0x140026AB8, 11, 0x10)
```

第二个有用的初始化函数里面修改了 `main` 函数的地址，真实地址为 `1400017A0`。

刚开始做这题时，并没有认真看 `sub_1400013B0` 函数。发现只需要输入和程序由时间戳生成而来的一些常量满足一些简单的算数关系即可。通过 `trace` 获取生成的量，构造满足条件的输入后本地就过了，但远程连挂了三组（没错，又要等第二天了）。

后面认真看了下 `sub_1400013B0` 函数，发现是椭圆曲线的点加操作，初始化函数里也是在初始化一个阶小于 200 的非退化的椭圆曲线。于是猜测对输入数据实际上还有一些额外的要求。将输入的最后两个数（用作椭圆曲线的 `a`、`p` 参数改成和题目生成的一样），再去构造剩余输入，就过了。

### EXP

```
A = [
(0xFFFFFFFFFFFFE12E, 0x237),
# ......
]
```
