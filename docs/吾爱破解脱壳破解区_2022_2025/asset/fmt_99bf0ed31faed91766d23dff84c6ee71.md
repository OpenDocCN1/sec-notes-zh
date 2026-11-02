
# [CTF] 算法分析：Snake

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1609759
- 抓取时间: `2026-11-01T23:31:45`

## 算法分析：Snake

### 1. 先下载附件，查壳运行得知初步逻辑

查壳得知是 `UPX` 壳，那直接使用 `UPX` 脱壳工具一键脱壳，得到原程序信息，32 位控制台程序

![](https://attach.52pojie.cn/forum/202203/24/113850htzhv9kayajjh1jh.png)

image-20220323193407584.png (40.23 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzA2NHxhMmNmMWNiMHwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzA2NHxhMmNmMWNiMHwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes)

1

2022-3-24 11:38 上传

运行一下 `exe`，查看运行逻辑

需要先输入名字

![](https://attach.52pojie.cn/forum/202203/24/113903xppeex7c979exbx7.png)

image-20220323193512763.png (23.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwNzA2NXxkNGJhNjI2OXwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzA2NXxkNGJhNjI2OXwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes)

2

2022-3-24 11:39 上传

发现就是一个贪吃蛇游戏，死亡以后便得到了关于 `flag` 的信息，要求我们分数高才能得到 `flag`，

![](https://attach.52pojie.cn/forum/202203/24/113914heers9x72eaharhn.png)

image-20220323193542952.png (28.71 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwNzA2NnxlNzY3Y2I5ZnwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzA2NnxlNzY3Y2I5ZnwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes)

3

2022-3-24 11:39 上传

![](https://attach.52pojie.cn/forum/202203/24/113927ezntng6xf0gdgezo.png)

image-20220323193610454.png (26.39 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwNzA2N3xhNzkxYWJkMnwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzA2N3xhNzkxYWJkMnwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes)

4

2022-3-24 11:39 上传

按照以往经验猜测这种基本上要么要求的分数很高，要么就是一个骗局，那么我们直接采用 `IDA Pro` 来进行分析

### 2. 静态分析

老规矩，先查看是否存在可以字符串

![](https://attach.52pojie.cn/forum/202203/24/113935x5iyyvbabbujfux1.png)

image-20220323194247136.png (59.84 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUwNzA2OHwxMjYyYjMzZnwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzA2OHwxMjYyYjMzZnwxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes)

5

2022-3-24 11:39 上传

发现三个可疑的字符串，第一个：通过以往经验可以猜测可能是某一轮 `Flag` 加密的结果，第二个：64 位长度，每个字符都是 `Base64` 编码表中的字符，所以 99% 就是 `Base64` 编码表，那可以先预料到可能是使用了 `Base64` 加密，第三个：通常这种提示厉害正确类似的话便可以很大程度确定这串字符所在的位置就是 `Flag` 判断位正确的位置

老规矩，顺藤摸瓜，我们直接定位到提示正确的函数取查看验证，因为通常在最后一次验证出结果的地方也能找到最后一次的 `Flag` 加密字符，那双击字符便来到了 `sub_40186F` 函数，我们先按下 `N` 把 `sub_40186F` 函数名全局改为 `Check`，接下来我们就称 `sub_40186F` 函数为 `Check`

```
int Check()
{
  char v1[256]; // [sp+18h] [bp-910h]@1
  char Dst[2048]; // [sp+118h] [bp-810h]@4
  int j; // [sp+918h] [bp-10h]@4
  int i; // [sp+91Ch] [bp-Ch]@1

  sub_4021AD(22, 18);
  scanf("%s", v1);
  for ( i = 0; v1[i]; ++i )
    ;
  sub_4017D2(v1, i);
  memset(Dst, 0, 0x800u);
  sub_4015F7(v1, Dst, i);
  sub_4021AD(22, 20);
  for ( j = 0; Dst[j]; ++j )
  {
    if ( Dst[j] != a7g5d5bayTmdlwl[j] )
      return puts("不对哦~下次再来吧~");
  }
  return puts("哇！你真厉害！");
}
```

阅读代码可以知道只要 `if` 条件 `Dst` 中的每一个元素的值不等于 `a7g5d5bayTmdlwl` 中的对应的每一个元素的值那么便提示不对，那么只要相等便会循环结束且提示正确，此时可以确定 `a7g5d5bayTmdlwl` 数组就是最后一轮 `Flag` 的加密结果，双击 `a7g5d5bayTmdlwl` 便可得到其值为：`7G5d5bAy+TMdLWlu5CdkMTlcJnwkNUgb2AQL3CcmPpVf6DAp72scOSlb`

那我们就找已知和未知的关系了，这里的已知条件是最后一轮 `Flag` 加密的结果，未知的条件便是我们的正确 `Flag`

之前的帖子我说过找已知和未知的技巧，就是找左值和右值，我们读 `Check` 代码知道 `v1` 是我们输入字符串所在地址，`i` 是输入字符串的长度，那从下往上找左右值关系

```
// 十四行
sub_4015F7(v1, Dst, i);
// 十二行
sub_4017D2(v1, i);
```

那就先分析第十四行的 `sub_4015F7(v1, Dst, i)`

```
int __cdecl sub_4015F7(int a1, int a2, int a3)
{
  int v3; // eax@2
  int v4; // eax@3
  int v5; // ST08_4@3
  int v6; // eax@3
  int v7; // eax@4
  int v8; // eax@5
  int v9; // eax@6
  unsigned __int8 v11; // [sp+7h] [bp-9h]@2
  unsigned __int8 v12; // [sp+7h] [bp-9h]@4
  int v13; // [sp+8h] [bp-8h]@1
  int v14; // [sp+8h] [bp-8h]@2
  int v15; // [sp+8h] [bp-8h]@4
  int v16; // [sp+Ch] [bp-4h]@1

  v16 = 0;
  v13 = 0;
  while ( v16 > 2) & 0x3F];
    v11 = 16 * *(_BYTE *)(v16 + a1) & 0x30;
    if ( v16 + 1 >= a3 )
    {
      v4 = v14;
      v5 = v14 + 1;
      *(_BYTE *)(a2 + v4) = Str[v11];
      *(_BYTE *)(v5 + a2) = 61;
      v6 = v5 + 1;
      v13 = v5 + 2;
      *(_BYTE *)(v6 + a2) = 61;
      break;
    }
    v7 = v14;
    v15 = v14 + 1;
    *(_BYTE *)(a2 + v7) = Str[((signed int)*(_BYTE *)(v16 + 1 + a1) >> 4) & 0xF | v11];
    v12 = 4 * *(_BYTE *)(v16 + 1 + a1) & 0x3C;
    if ( v16 + 2 >= a3 )
    {
      *(_BYTE *)(a2 + v15) = Str[v12];
      v8 = v15 + 1;
      v13 = v15 + 2;
      *(_BYTE *)(v8 + a2) = 61;
      break;
    }
    *(_BYTE *)(a2 + v15) = Str[((signed int)*(_BYTE *)(v16 + 2 + a1) >> 6) & 3 | v12];
    v9 = v15 + 1;
    v13 = v15 + 2;
    *(_BYTE *)(a2 + v9) = Str[*(_BYTE *)(v16 + 2 + a1) & 0x3F];
    v16 += 3;
  }
  *(_BYTE *)(v13 + a2) = 0;
  return a2;
}
```

`a1` 是我们输入字符串所在的地址，`a2` 是最后一轮 `Flag` 加密结果，`a3` 是输入字符串的长度

分析代码 `*(_BYTE *)(v13 + a2) = 0;` 可以知道这里是做最后一轮 `Flag` 加密字符串结尾的工作，但是结尾的这个 `0` 是不被算入长度的，所以 `v13` 是第 57 位，下角标位 56，所以 `v13` 在该函数中最后等于 56；

阅读代码，得知最后一轮 `Flag` 加密字符串 `a2` 与输入字符 `a1` 无条件做左右值的地方有：

```
*(_BYTE *)(a2 + v3) = Str[((signed int)*(_BYTE *)(v16 + a1) >> 2) & 0x3F];
*(_BYTE *)(a2 + v7) = Str[((signed int)*(_BYTE *)(v16 + 1 + a1) >> 4) & 0xF
*(_BYTE *)(a2 + v15) = Str[((signed int)*(_BYTE *)(v16 + 2 + a1) >> 6) & 3 | v12];
*(_BYTE *)(a2 + v9) = Str[*(_BYTE *)(v16 + 2 + a1) & 0x3F];
```

分析 `v3`、`v7`、`v15`、`v9`、`v16`、`v11`、`v12` 的关系：

```
// 循环外初始值
v16 = 0;
v13 = 0;
// 单次循环内变换
v3 = v13;
v14 = v13 + 1;
v11 = 16 * *(_BYTE *)(v16 + a1) & 0x30;
v7 = v14;
v15 = v14 + 1;
v12 = 4 * *(_BYTE *)(v16 + 1 + a1) & 0x3C;
v9 = v15 + 1;
v13 = v15 + 2;
v16 += 3;
```

那么到这里可以化简以上代码为：

```
*(_BYTE *)(a2 + i) = Str[((signed int)*(_BYTE *)(v16 + a1) >> 2) & 0x3F];
*(_BYTE *)(a2 + i+1) = Str[((signed int)*(_BYTE *)(v16 + 1 + a1) >> 4) & 0xF
*(_BYTE *)(a2 + i+2) = Str[((signed int)*(_BYTE *)(v16 + 2 + a1) >> 6) & 3 | v12];
*(_BYTE *)(a2 + i+3) = Str[*(_BYTE *)(v16 + 2 + a1) & 0x3F];
```

也就是说在循环内不受任何条件影响每次有序（划重点）操作四个 `a2` 字符，那么 `a2` 总共有 57 个字符，循环外操作了最后一个，那循环内只能操作前 56 个了，也就是循环了 56/4=14 轮，做多也只能操作 14 轮，那经过 14 轮循环以后 `v16` 等于 42，也就是说 `while` 的条件中的 `a3` 便是 42 两个 `if` 条件分别是：`if ( v16 + 1 >= a3 )`、`if ( v16 + 2 >= a3 )`，我们可推算在 `v16` 在满足循环条件内在两个 `if` 前的最大值为 3*13= 39，那 `v16+1` 和 `v16+2` 的最大值分别为 40、41，所以两个 `if` 条件是永远不会满足的，那么整个函数的关键逻辑便可化简为

```
while(v16> 2) & 0x3F];
        *(_BYTE *)(a2 + v7) = Str[((signed int)*(_BYTE *)(v16 + 1 + a1) >> 4) & 0xF | v11];
        *(_BYTE *)(a2 + i+2) = Str[((signed int)*(_BYTE *)(v16 + 2 + a1) >> 6) & 3         | v12];
        *(_BYTE *)(a2 + i+3) = Str[*(_BYTE *)(v16 + 2 + a1) & 0x3F];
        i += 4;
        v16 += 3;
}
```

那此时我们双击 `Str` 查看具体内容是啥：

![](https://attach.52pojie.cn/forum/202203/24/113957p9fiins8z0li8cmm.png)

image-20220323203210172.png (17.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNzA2OXxjZjVjMzUzN3wxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzA2OXxjZjVjMzUzN3wxNzkzNTc1OTAyfDE2ODA4NnwxNjA5NzU5&nothumb=yes)

6

2022-3-24 11:39 上传

这不就是我们的 `Base6` 编码表码？先说一下什么是 `Base64` 编码以及如何实现的：

`Base64` 是一种以 64 个可打印字符对二进制数据进行编码的编码算法。`Base64` 在对数据进行编码时以三个 8 位字符型数据为一组，取这三个字符型数据的 `ASCII` 码，然后以 6 位为一组组成 4 个新的数据，这 4 个新的数据有 6 位，所以它的最大值为 2^6=64。我们以 4 个 6 位数据的十进制数从 `Base64` 表中得到最终编码后的字符。(引用 `CSDN@Alen.Wang`)

`Base64` 编码表：

Value |
Char |
Value |
Char |
Value |
Char |
Value |
Char |

0
```
