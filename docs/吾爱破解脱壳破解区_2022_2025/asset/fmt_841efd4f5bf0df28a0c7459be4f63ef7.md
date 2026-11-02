
# [CTF] 【reverse】新160个CrackMe之154-cpp_crackme1——MFC+纯算法逆向

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1683826
- 抓取时间：2026-11-01T23:10:16

## 依赖

IDA版本为7.7。

## PETools查看概况

32位程序，入口点 `Section: [.text]`，`EP: 0x00001DB9`，故无壳。

## 正文

作者：[hans774882968](https://blog.csdn.net/hans774882968) 以及 [juejin](https://juejin.cn/user/1464964842528888) 以及 [52pojie](https://www.52pojie.cn/home.php?mod=space&uid=1906177)

本文juejin：[juejin.cn/post/7139136489585115173](https://juejin.cn/post/7139136489585115173/)

本文csdn：[blog.csdn.net/hans774882968/article/details/126682443](https://blog.csdn.net/hans774882968/article/details/126682443)

本文52pojie：[www.52pojie.cn/thread-1683826-1-1.html](https://www.52pojie.cn/thread-1683826-1-1.html)

### 如何定位关键函数

- 因为函数很少，所以一个一个点开来看，即可定位到关键函数 `sub_401660`。
- 打开x64dbg，正常地触发一次输入序列号错误，点击暂停键，查看调用堆栈，即可定位。

### 分析

`sub_401660`：

```
int sub_401660()
{
  int iii; // ebx
  int *v2; // edi
  int v3; // eax
  int i; // ecx
  int v5; // ecx
  char *v6; // edi
  int v7; // edx
  int v8; // esi
  char *v9; // eax
  int v10; // ecx
  int v11; // ecx
  int *v12; // edi
  int WindowTextA; // eax
  char v14[120]; // [esp+8h] [ebp-328h] BYREF
  CHAR Caption[256]; // [esp+80h] [ebp-2B0h] BYREF
  CHAR Text[256]; // [esp+180h] [ebp-1B0h] BYREF
  char String[28]; // [esp+280h] [ebp-B0h] BYREF
  __int16 v18; // [esp+29Ch] [ebp-94h]
  char Buf1[28]; // [esp+2A0h] [ebp-90h] BYREF
  __int16 v20; // [esp+2BCh] [ebp-74h]
  char v21; // [esp+2BEh] [ebp-72h]
  char Buf2[28]; // [esp+2C0h] [ebp-70h] BYREF
  __int16 v23; // [esp+2DCh] [ebp-54h]
  char v24; // [esp+2DEh] [ebp-52h]
  int v25; // [esp+2E0h] [ebp-50h]
  int v26; // [esp+2E4h] [ebp-4Ch]
  CHAR Str1[28]; // [esp+2E8h] [ebp-48h] BYREF
  char Source[43]; // [esp+304h] [ebp-2Ch] BYREF
  char v29; // [esp+32Fh] [ebp-1h]

  memset(String, 0, sizeof(String));
  v18 = 0;
  memset(Buf1, 0, sizeof(Buf1));
  v20 = 0;
  memset(Buf2, 0, sizeof(Buf2));
  v23 = 0;
  memset(v14, 0, sizeof(v14));
  memset(Source, 0, 30);
  if ( GetWindowTextA(dword_40974C, String, 30) >= 1 )
  {
    iii = 0;
    v2 = &dword_407030;
    do
    {
      if ( iii >= lstrlenA(String) )
        Buf1[iii] = (iii + (*v2 ^ 0x5A)) % 57 + 65;
      else
        Buf1[iii] = (iii + 1) ^ String[iii];
      ++v2;
      ++iii;
    }
    while ( (int)v2 = 1 )
    {
      if ( WindowTextA >= 24
        && !strncmp(Str1, &Source[16], 0x14u)
        && (Str1[20] ^ 0x63) == 0x53
        && (Str1[21] ^ 0x63) == 0x52
        && (Str1[22] ^ 0x63) == 0x57
        && (Str1[23] ^ 0x63) == 0x52 )
      {
        sub_401B20(byte_4071CC, 12);
        qmemcpy(Caption, byte_409754, 0xFFu);
        sub_401B20(byte_4071FC, 10);
        qmemcpy(Text, byte_409754, 0xFFu);
        return MessageBoxA(hWnd, Caption, Text, 0x40u);
      }
      else
      {
        sub_401B20(byte_407190, 15);
        qmemcpy(Caption, byte_409754, 0xFFu);
        sub_401B20(byte_407178, 6);
        qmemcpy(Text, byte_409754, 0xFFu);
        return MessageBoxA(hWnd, Caption, Text, 0x10u);
      }
    }
    else
    {
      sub_401B20(byte_407118, 24);
      qmemcpy(Caption, byte_409754, 0xFFu);
      sub_401B20(byte_407178, 6);
      qmemcpy(Text, byte_409754, 0xFFu);
      return MessageBoxA(hWnd, Caption, Text, 0x10u);
    }
  }
  else
  {
    sub_401B20(byte_4070C0, 22);
    qmemcpy(Text, byte_409754, 0xFFu);
    sub_401B20(byte_407178, 6);
    qmemcpy(Caption, byte_409754, 0xFFu);
    return MessageBoxA(hWnd, Text, Caption, 0x10u);
  }
}
```

这里有两个全局变量，`dword_40974C` 和 `dword_409750` 分别对应你的Name和你输入的序列号。

#### 常量串隐藏

为什么不能通过常量串定位关键函数？因为做了常量串隐藏。看到 `sub_401B20` 就是一个字符串解密操作，我们写个IDAPython脚本看看各个常量串：

```
import idc

def get_dec_str(enc):

    ans = ''

    for i in range(0, len(enc), 4):

        ans += chr(enc[i] ^ 0x63)

    return ans

a = [

    [0x4070c0, 88], [0x407118, 96], [0x407178, 24],

    [0x407190, 60], [0x4071cc, 48], [0x4071fc, 40]

]

mp = {}

for addr, c in a:

    enc_s = get_bytes(addr, c)

    dec_s = get_dec_str(enc_s)

    mp[hex(addr)] = dec_s

print(mp)  # {'0x4070c0': 'You must enter a name!', '0x407118': 'You must enter a serial!', '0x407178': 'Error!', '0x407190': 'Invalid serial!', '0x4071cc': 'Good serial!', '0x4071fc': 'Well Done!'}
```

接下来我们从下到上分析每一个小模块。

##### 判定

```
    WindowTextA = GetWindowTextA(dword_409750, Str1, 25);
    if ( WindowTextA >= 24
        && !strncmp(Str1, &Source[16], 0x14u) // Str1是你输入的序列号
        && (Str1[20] ^ 0x63) == 0x53
        && (Str1[21] ^ 0x63) == 0x52
        && (Str1[22] ^ 0x63) == 0x57
        && (Str1[23] ^ 0x63) == 0x52 )
```

需要关注 `Source + 16` 字符串怎么来：

```
    v12 = (int *)v14;

    do

    {

      v25 = *v12;

      LOBYTE(v26) = v25;

      ++v12;

      Source[j++] = (unsigned __int8)v25 % 9 + 48;

    }

    while ( j 
```

所以重点关注对 `v14` 所在内存空间的修改：

```
    v6 = v14;
    do
    {
      v7 = Buf2[ii]; // 注意这里是有符号扩展
      v8 = Buf1[ii];
      v6 += 4;
      ++ii;
      *((_DWORD *)v6 - 1) = v7 * v8 + (v7 ^ v8); // v6+4-1*4
    }
    while ( ii 
```

值得注意的是，`v7` 是 `int`，`Buf2[ii]` 是 `char`，C++里把 `char` 赋值给 `int` 的隐式类型转换是“有符号扩展”，即把 `Buf2[ii]` 解释为8位有符号整数。对应的汇编指令为：`movsx`。验证demo：

```
#include 

#include 

int main() {

    char c1, c2, c3;

    c1 = 0xfe;

    c2 = 0xfc;

    c3 = 0xfa;

    int v1 = c1, v2 = c2, v3 = c3;

    printf ("%d %d %d\n", v1, v2, v3); // -2 -4 -6

    return 0;

}
```

接下来看 `Buf1` 和 `Buf2` 怎么求得。

##### 求Buf2

```
    _swab(Buf1, Buf2, 30);    
    v3 = 0;
    for ( i = 0; i 
```

- `_swab` 即相邻字符串为一组，交换位置。
- 易错点：`v3 = __ROR4__(v3, 1)` 不能理解成右移1位，因为 `v3` 是 `int` 类型，在操作24次后将不等价于右移1位。

`_swab`：

```
void __cdecl _swab(char *Buf1, char *Buf2, int SizeInBytes)

{

  unsigned int v4; // esi

  char v6; // dl

  char *v7; // ecx

  if ( SizeInBytes > 1 )

  {

    v4 = (unsigned int)SizeInBytes >> 1;

    do

    {

      v6 = *Buf1;

      *Buf2 = Buf1[1];

      Buf1 += 2;

      v7 = Buf2 + 1;

      *v7 = v6;

      Buf2 = v7 + 1;

      --v4;

      // Buf2[i] = Buf1[i+1], Buf2[i+1] = Buf1[i], Buf1 += 2, Buf2 += 2

    }

    while ( v4 );

  }

}
```

##### 求Buf1

```
    iii = 0;
    v2 = &dword_407030; // GetWindowTextA(dword_40974C, String, 30)，所以String就是你的Name
    do
    {
      if ( iii >= lstrlenA(String) )
        Buf1[iii] = (iii + (*v2 ^ 0x5A)) % 57 + 65;
      else
        Buf1[iii] = (iii + 1) ^ String[iii];
      ++v2;
      ++iii;
    }
    while ( (int)v2 
```

因此，我们要做的，是把生成序列号的算法复现出来，算法参数：你输入的Name。

### 踩坑总结

- 把 `char` 赋值给 `int` 的隐式类型转换是“有符号扩展”。
- 不要想当然地把 `__ROR4__` 简化为右移一位。Python实现时不要怕麻烦。

### 代码

这里我们有两个看上去不错的选择：

- 用Python来复现。
- 用C++来复现，并用IDA的 `defs.h` 来减少代码的改动。`defs.h` 在IDA安装目录下可搜索到。

我把两种方法都做了，法二的心智负担明显更小。

#### 法一

```
def get_buf1(inp_name):

    dword_407030 = [

        0x16, 0x00, 0x00, 0x00, 0x0F, 0x00, 0x00, 0x00, 0x17, 0x00,

        0x00, 0x00, 0x11, 0x00, 0x00, 0x00, 0x02, 0x00, 0x00, 0x00,

        0x10, 0x00, 0x00, 0x00, 0x0C, 0x00, 0x00, 0x00, 0x16, 0x00,

        0x00, 0x00, 0x0D, 0x00, 0x00, 0x00, 0x07, 0x00, 0x00, 0x00,

        0x00, 0x00, 0x00, 0x00, 0x11, 0x00, 0x00, 0x00, 0x02, 0x00,

        0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08, 0x00, 0x00, 0x00,

        0x0E, 0x00, 0x00, 0x00, 0x06, 0x00, 0x00, 0x00, 0x52, 0x00,

        0x00, 0x00, 0x16, 0x00, 0x00, 0x00, 0x0F, 0x00, 0x00, 0x00,

        0x17, 0x00, 0x00, 0x00, 0x11, 0x00, 0x00, 0x00, 0x02, 0x00,

        0x00, 0x00, 0x10, 0x00, 0x00, 0x00, 0x0C, 0x00, 0x00, 0x00,

        0x16, 0x00, 0x00, 0x00, 0x0D, 0x00, 0x00, 0x00, 0x07, 0x00,

        0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x11, 0x00, 0x00, 0x00

    ]

    buf1 = []

    for i in range(30):
```
