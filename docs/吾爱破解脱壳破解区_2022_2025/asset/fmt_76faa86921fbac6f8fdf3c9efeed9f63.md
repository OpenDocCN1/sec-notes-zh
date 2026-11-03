
```python
def crc32_2(data):

    crc = 0xFFFFFFFF  # 初始值

    for i in data:

        crc = crc ^ i

        for j in range(8):

            if crc & 0x00000001:

                crc = (crc >> 1) ^ 0xEDB88320  # 逆序后的多项式 0xEDB88320= reverse 0x04C11DB7

            else:

                crc = (crc >> 1)

            # print(hex(crc))

    return (crc ^ 0xFFFFFFFF) | 0x80808080  # 或者写成~crc&0xffffffff   结果异或值

print(bytes([50,48,50,53,53,50,112,111,106,105,101,97,102,100,109,50,48,50,53]))

print(bytes([102,108,97,103]))

sbox = [0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x09, 0x0A, 0x10, 0x15, 0x21, 0x13, 0x0C, 0x04, 0x11, 0x1C, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x03, 0x0F, 0x20, 0x0D, 0x02, 0x23, 0x06, 0x1B, 0x14, 0x0E, 0x01, 0x16, 0x19, 0x08, 0x12, 0x1F, 0x17, 0x24, 0x0B, 0x1E, 0x07, 0x1A, 0x05, 0x18, 0x1D, 0x22, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]

sum = 0

for i in b"11111":

    sum*=36

    sum+=sbox[i]-1

    # print(hex(sum))

print(hex(sum))

sum=0

for i in b"22222":

    sum*=36

    sum+=sbox[i]-1

    # print(hex(sum))

print(hex(sum))

crc32_val = [0xbd,0xc3,0x90,0x8d]

flag_off = [5,11,17,23]

sum = 0

table = b"0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"

for i in range(4):

    print(hex(0x13541*crc32_val[i] + flag_off[i]))

data = [0xe45102,

0xeb908e,

0xadf4a1,

0xaa54e4]

for i in range(4):

    s=chr(sbox.index(data[i]%36+1))+chr(sbox.index(data[i]//36%36+1))+chr(sbox.index(data[i]//36//36%36+1))+chr(sbox.index(data[i]//36//36//36%36+1))+chr(sbox.index(data[i]//36//36//36//36%36+1))

    s=s[::-1]

    print(s,end="-")

print()

for i in b"HHW8A-5JYKV-3JEV6-HLE09":

    print(i,end=",")

print()

for _ in range(4):

    for t1 in table:

        for t2 in table:

            for t3 in table:

                for t4 in table:

                    for t5 in table:

                        t = [t1,t2,t3,t4,t5]

                        sum = 0

                        for i in t:

                            sum*=36

                            sum+=sbox[i]-1

                        if (sum+1) % data[_] == 0 and sum>>25!=0:

                            print(bytes(t),_)
```

## 3> 解题领红包之七 {Windows 高级题}

在`First`数组中添加了初始化和反调试

base `0x7FF7A5930000`

![](https://attach.52pojie.cn/forum/202502/13/145407auoxuf9qsulzsz92.png)

image-20250210232532079.png (70.01 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NDY2M3wyMmE1Y2Q4YXwxNzkzNTcxMzEwfDE2ODA4NnwyMDA1OTY2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDY2M3wyMmE1Y2Q4YXwxNzkzNTcxMzEwfDE2ODA4NnwyMDA1OTY2&nothumb=yes)

2025-2-13 14:54 上传

`check`函数逻辑根据时间创建`ecc`的`a` `p`

![](https://attach.52pojie.cn/forum/202502/13/145409tikavepiyzcae444.png)

image-20250210235927709.png (72.88 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NDY2NHw5MWVlNGEzZXwxNzkzNTcxMzEwfDE2ODA4NnwyMDA1OTY2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDY2NHw5MWVlNGEzZXwxNzkzNTcxMzEwfDE2ODA4NnwyMDA1OTY2&nothumb=yes)

2025-2-13 14:54 上传

但是跟`ecc`没什么关系

只需要在动调的过程中拿到`a`和`p`放到`key`的后面即可

`ida`写个条件断点直接动态赋值

地址`07FF7A5931AD4`

```python
import ida_dbg
import idaapi
p = idaapi.get_qword(0x7FF7A5956AF8)
a = idaapi.get_qword(0x7FF7A5956B08)
ida_dbg.set_reg_val("r14",p)
ida_dbg.set_reg_val("r15",a)
```

地址`07FF7A5931B37`

```python
import ida_dbg
import idaapi
py = ida_dbg.get_reg_val("rax")
px = ida_dbg.get_reg_val("rcx")
r12 = ida_dbg.get_reg_val("r12")
print(py,px,r12)
p = idaapi.get_qword(0x7FF7A5956AF8)
a = idaapi.get_qword(0x7FF7A5956B08)
print(f"{a=} {p=}")
```

```python
import ida_dbg
import idaapi
rbp = ida_dbg.get_reg_val("rbp")
enc = idaapi.get_bytes(rbp+0x30,0x10)
print(enc)
t = 0
for i in range(16):
    for bf in range(1000000):
        temp = ((bf+(-10*py)-t)//px)&0xff
        if temp == enc[i]:
            print(str(bf).zfill(6),end='')
            break
    t+=0x27DA
print(str(a).zfill(6)+str(p).zfill(6))
```

但是实际测试时发现有时输出不正确，有时正确。存在点小bug

## 3> 解题领红包之八 {Android 高级题}

主要逻辑在`so`层，一堆混淆不过可以找到动态注册的`checkSn`函数

主要逻辑：`bit`倒序+`3des`

```c
__int64 __fastcall sub_7A09998454(__int64 a1, unsigned __int64 a2)
{
  __int64 v3; // x0
  __int64 v4; // x0
  __int64 v5; // x0

  v3 = reverse_bit(a2);
  v4 = des_enc(a1, v3);
  v5 = des_dec(a1 + 128, v4);
  return des_enc(a1 + 256, v5);
}
```

因为`des`的第一个参数是48个`dword` 且前16个`dword`与后16个字节相同，`des_enc` `des_dec`内部调用同一个函数，只是第三个参数不同，故推测是`3des`

```python
import re
# ========================================
# 一、子密钥生成
# (1) 初始置换 64->56
# 64位的种子密钥经过PC_1置换后，生成56位的密钥
# (2) 划分 56->(28,28)
# 经过初始置换后的56位密钥被均分成C0和D0两部分
# (3) 循环左移
# 第一轮，C0和D0根据移位次数表各自进行循环左移
# 得到C1和D1
# 每一轮的C和D值是由上一轮的C和D值循环左移得到的
# (4) 合并 (28,28)->56->48
# 左移后的两部分再次合并,通过一个选择压缩表(PC_2)
# 得到这一轮的子密钥
# (5)重复3、4操作,最终得到16个子密钥
# ========================================
# 置换选择表1(PC_1) 64->56
PC_1 = [57, 49, 41, 33, 25, 17, 9,
        1, 58, 50, 42, 34, 26, 18,
        10, 2, 59, 51, 43, 35, 27,
        19, 11, 3, 60, 52, 44, 36,
        63, 55, 47, 39, 31, 23, 15,
        7, 62, 54, 46, 38, 30, 22,
        14, 6, 61, 53, 45, 37, 29,
        21, 13, 5, 28, 20, 12, 4
        ]
# 选择压缩表2(PC_2) 56->48
PC_2 = [14, 17, 11, 24, 1, 5, 3, 28,
        15, 6, 21, 10, 23, 19, 12, 4,
        26, 8, 16, 7, 27, 20, 13, 2,
        41, 52, 31, 37, 47, 55, 30, 40,
        51, 45, 33, 48, 44, 49, 39, 56,
        34, 53, 46, 42, 50, 36, 29, 32
        ]
# 移位次数表
shift_num = [1, 1, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 2, 1]
def pc_1_change(bin_key):
    """初始置换
    64位的种子密钥经过PC_1置换后，生成56位的密钥
    """
    return [bin_key[i - 1] for i in PC_1]
def shift_left(bin_key, num):
    """实现C和D的循环左移"""
    return bin_key[num:] + bin_key[:num]
def pc_2_change(bin_key):
    """选择压缩
    56位的密钥经过PC_2压缩，生成48位子密钥
    """
    return ''.join([bin_key[i - 1] for i in PC_2])
def get_subkey_list(bin_key):
    """生成16轮的加解子密钥"""
    subkey_list = []
    temp = pc_1_change(bin_key)
    for i in shift_num:
        temp[:28] = shift_left(temp[:28], i)
        temp[28:] = shift_left(temp[28:], i)
        subkey_list.append(pc_2_change(temp))
    return subkey_list
# ========================================
# 二、DES加解密实现
# ========================================
# 初始置换表IP 64->64
IP = [58, 50, 42, 34, 26, 18, 10, 2,
      60, 52, 44, 36, 28, 20, 12, 4,
      62, 54, 46, 38, 30, 22, 14, 6,
      64, 56, 48, 40, 32, 24, 16, 8,
      57, 49, 41, 33, 25, 17, 9, 1,
      59, 51, 43, 35, 27, 19, 11, 3,
      61, 53, 45, 37, 29, 21, 13, 5,
      63, 55, 47, 39, 31, 23, 15, 7
      ]
# 逆置换表_IP 64->64
_IP = [40, 8, 48, 16, 56, 24, 64, 32, 39,
       7, 47, 15, 55, 23, 63, 31, 38, 6,
       46, 14, 54, 22, 62, 30, 37, 5, 45,
       13, 53, 21, 61, 29, 36, 4, 44, 12,
       52, 20, 60, 28, 35, 3, 43, 11, 51,
```
