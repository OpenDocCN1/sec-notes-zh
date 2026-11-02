
# [CTF] 算法分析：西湖论剑2019预赛题

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1605552
- 抓取时间: 2026-11-01T23:32:22

## 算法分析：西湖论剑2019预赛题

### Tip: CTF普遍做题思维

- 尽可能查找可疑字符串，并分析所引用函数
- 找到关键验证点，比如提示失败成功之类的逻辑处
- 查看`main`函数逻辑结合运行`cm`大致分析得到程序整体运行逻辑
- 跟踪分析`flag`最后一次加密结果作位左值的任何地方
- 查找分析最后一次`flag`加密结果与任何我们人为操作数据的关系（比如最后一次`flag`加密结果和我们输入的字符串的关系）
- 综合上面的方法便可排除干扰项从而利用最后一次`flag`加密结果逆向推导得到人为操作数据（通常就是正确的`flag`）

#### 0x1: 下载附件查壳发现是一个64位ELF文件，那么我们用虚拟机跑一下

![](https://attach.52pojie.cn/forum/202203/17/212127ldo28zewzn2hpfmo.png)

image-20220317191723515.png (23.89 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNDQ3M3wxNWZjYTU4MnwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ3M3wxNWZjYTU4MnwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

1

2022-3-17 21:21 上传

运行了一下，发现有输入，输入之后回车就结束了进程

#### 0x2: 使用IDA Pro 进行静态分析

`shift+F12`查找是否存在关键字符串

![](https://attach.52pojie.cn/forum/202203/17/212142b7944e2ht5wvayek.png)

image-20220317192023180.png (15.16 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUwNDQ3NXwwY2JkMmI4NHwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ3NXwwY2JkMmI4NHwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

2

2022-3-17 21:21 上传

发现字符串“correct!”,按照以往的经验来看这就是`flag`正确后的提示字符，那我们查看应用函数

![](https://attach.52pojie.cn/forum/202203/17/212154ijn2oqinq2yqgayu.png)

image-20220317192210739.png (14 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjUwNDQ3N3w3OGRlNzJiOXwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ3N3w3OGRlNzJiOXwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

3

2022-3-17 21:21 上传

发现关键验证逻辑：只要`if`满足那么就会输出“correct！”，否则执行一系列运算后返回到了调用者`main`函数

![](https://attach.52pojie.cn/forum/202203/17/212241ob8u4zfu7rhhq9rz.png)

image-20220317192344610.png (16.69 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUwNDQ3OHwxODZkMzM0ZHwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ3OHwxODZkMzM0ZHwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

4

2022-3-17 21:22 上传

![](https://attach.52pojie.cn/forum/202203/17/212257qn533zxqho1nj3om.png)

image-20220317192447179.png (9.62 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwNDQ3OXwyOGI0OTVkZnwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ3OXwyOGI0OTVkZnwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

5

2022-3-17 21:22 上传

![](https://attach.52pojie.cn/forum/202203/17/212311gp5mo4kvlm03mdja.png)

image-20220317192534492.png (21.44 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjUwNDQ4MXxkYzRhN2JjMHwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ4MXxkYzRhN2JjMHwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

6

2022-3-17 21:23 上传

回到`main`函数后发现：ELF释放了内存后就结束了进程，也就证明了我们刚才在`kali`上跑的时候为啥输入了以后直接结束了进程而没有提示任何错误信息；

#### 0x3: 回到`sub_400700`函数分析刚才的关键验证判断

```
if ( !strncmp((const char *)s, "D9", 2uLL)
      && !strncmp((const char *)s + 20, "Mp", 2uLL)
      && !strncmp((const char *)s + 18, "MR", 2uLL)
      && !strncmp((const char *)s + 2, "cS9N", 4uLL)
      && !strncmp((const char *)s + 6, "9iHjM", 5uLL)
      && !strncmp((const char *)s + 11, "LTdA8YS", 7uLL) )
    {
      v6[1] = puts("correct!");
    }
```

6> 可以看到`s`就是`flag`最后一轮加密的结果，也就是只要`s`满足以上条件则就证明了`s`就是正确的`flag`加密后得到的结果，那么可以先将`s`的正确字符写出(注意顺序)：

5> `char s = "D9cS9N9iHjMLTdA8YSMRMp"`

4> 0x4:老规矩，根据已知推未知，最后的目的都是找到`s`（最后一次`flag`加密结果）与我们的输入字符串之间的关系

6> 所以此时我们需要先找到我们输入的字符串放到了哪里？

```
__int64 __fastcall sub_400700(void *a1, _QWORD *a2, __int64 a3, size_t a4)
{
  unsigned __int8 *v4; // rcx
  _DWORD v6[2]; // [rsp+0h] [rbp-C0h] BYREF
  int c; // [rsp+8h] [rbp-B8h]
  char v8; // [rsp+Fh] [rbp-B1h]
  int v9; // [rsp+10h] [rbp-B0h]
  bool v10; // [rsp+17h] [rbp-A9h]
  unsigned __int8 *v11; // [rsp+18h] [rbp-A8h]
  char v12; // [rsp+27h] [rbp-99h]
  int v13; // [rsp+28h] [rbp-98h]
  int v14; // [rsp+2Ch] [rbp-94h]
  unsigned __int64 i; // [rsp+30h] [rbp-90h]
  size_t n; // [rsp+38h] [rbp-88h]
  size_t v17; // [rsp+40h] [rbp-80h]
  size_t v18; // [rsp+48h] [rbp-78h]
  size_t j; // [rsp+50h] [rbp-70h]
  size_t v20; // [rsp+58h] [rbp-68h]
  int v21; // [rsp+64h] [rbp-5Ch]
  unsigned __int64 v22; // [rsp+68h] [rbp-58h]
  int v23; // [rsp+74h] [rbp-4Ch]
  _DWORD *v24; // [rsp+78h] [rbp-48h]
  __int64 v25; // [rsp+80h] [rbp-40h]
  void *v26; // [rsp+88h] [rbp-38h]
  int v27; // [rsp+94h] [rbp-2Ch]
  size_t v28; // [rsp+98h] [rbp-28h]
  __int64 v29; // [rsp+A0h] [rbp-20h]
  _QWORD *v30; // [rsp+A8h] [rbp-18h]
  void *s; // [rsp+B0h] [rbp-10h]
  char v32; // [rsp+BFh] [rbp-1h]

  s = a1;
  v30 = a2;
  v29 = a3;
  v28 = a4;
  v27 = -559038737;
  v26 = malloc(0x100uLL);
  v25 = v29; // v25是用户输入
  v24 = v6;
  v22 = 0LL;
  v17 = 0LL;
  for ( i = 0LL; i >= 6;
      v21 /= 58;
      v27 /= v9;
      if ( !j )
        break;
    }
    ++v20;
    v18 = j;
  }
  for ( j = 0LL; ; ++j )
  {
    v8 = 0;
    if ( j  n + v17 - j )
  {
    if ( v17 )
    {
      c = 61;
      memset(s, 49, v17);
      memset(v26, c, v17);
    }
    v20 = v17;
    while ( j 
```

6> 阅读整个`sub_400700`函数并没有找到输入功能逻辑，所以回到`main`中去找

```
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  void *ptr; // [rsp+10h] [rbp-30h]
  __int64 v5; // [rsp+18h] [rbp-28h] BYREF
  char v6[28]; // [rsp+20h] [rbp-20h] BYREF
  int v7; // [rsp+3Ch] [rbp-4h]

  v7 = 0;
  v5 = 256LL;
  sub_400D00(v6, 17LL, a3);
  ptr = malloc(0x100uLL);
  sub_400700(ptr, &v5, (__int64)v6, 0x10uLL);
  free(ptr);
  return 0LL;
}
```

6> 阅读`main`函数发现并没有直观看到任何输入逻辑，那还剩下一个`sub_400D00`函数，那十有八九可以确定输入逻辑在这个里面，那我们就进去看看；

```
__int64 __fastcall sub_400D00(__int64 a1, unsigned __int64 a2)
{
  char buf; // [rsp+17h] [rbp-19h] BYREF
  unsigned __int64 i; // [rsp+18h] [rbp-18h]
  unsigned __int64 v5; // [rsp+20h] [rbp-10h]
  __int64 v6; // [rsp+28h] [rbp-8h]
  v6 = a1;
  v5 = a2;
  for ( i = 0LL; i 
```

6> 果然循环`read(0, &buf, 1uLL)`了`v5-1(a2 = 17-1)`次后将我们的输入字符串存到了`v6(a1)`中，回到`main`中查看便可知道我们输入的字符串存在了`main`局部变量的`v6`中；

6> 再看`sub_400700(ptr, &v5, (__int64)v6, 0x10uLL)`，第一个参数是申请得到256大小内存的首地址，`&v5`是一个变量256的地址，`v6`是我们输入的字符串所在地址，最后一个参数便是16，好，此时已经找到了我们输入的字符串和刚才的`flag`最后一次加密结果`s`的最基本关系在`sub_400700`函数中；

4> 0x5:到`sub_400700`中顺藤摸瓜找到`flag`最后加密结果与我们输入字符的具体关系

6> 找到`s`作为左值的所有逻辑，只有这三处

![](https://attach.52pojie.cn/forum/202203/17/212336mp9enfealreekpa5.png)

image-20220317195211183.png (17.93 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUwNDQ4NXxjZGIwYTJkMXwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNDQ4NXxjZGIwYTJkMXwxNzkzNTc1OTM5fDE2ODA4NnwxNjA1NTUy&nothumb=yes)

7

2022-3-17 21:23 上传

6> 列出与当`s`作为左值时右值变量有哪些：

6> `byte_400EB0`：123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz；

6> `v11`：未知的变量；

6> 寻找我们输入字符串作右值；

6> 首先，我们先看我们输入的字符串在`main`中的`v6`在`sub_400700`叫啥名；

6> `v25 = v29 = a3 = v6`；

6> 好，已经知道我们输入的字符串地址在这里叫`v25`,那开始找其作右值的地方；

```
//第一处
for ( i = 0LL; i >= 6;
    v21 /= 58;
    v27 /= v9;
    if ( !j )
      break;
  }
  ++v20;
  v18 = j;
}
```

6> 既然找到了这两个变量作左右值对应逻辑位置后，那我们就开始寻找`s`和`v25`的关联了；

6> 从上面可以知道，`s`作左值的时的未知右值只有`v11`，`v11`是一个数组

```
*((_BYTE *)s + v20) = byte_400EB0[v11[j]];
*((_BYTE *)v26 + v20++) = byte_400EF0[v4[j++]];
```

6> 从以上两行代码可以知道`s`数组的值来自于`byte_400EB0`中下标为`v11[j]`的值，这里其实密码学做得多的就可以看出，这属于一种编码,其实就是`base`系列的编码，我们可以查看编码表的长度确定是什么编码，那我们就去算一下`byte_400EB0`的长度123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz可以知道`byte_400EB0`的长度是58，而且这串编码表中没有O0l/+，所以可以直接确定以上两行代码输入`base58`加密的最后一步,为了严谨我们只需验证`v11`的由来是`v25`产生的（`base58`加密过程中产生编码表数组的下角标）便可以验证此加密就是`base58`加密，从而便验证知道`s`就是我们输入的字符`base58`加密后的结果

6> 此时，我们需要先知道`base58`的加密过程：

“将`ascii`编码的字符串（256进制），转换成58进制。然后按照58进制的码表转换成相应的字符。

1、将字符串的每个字节换算成ASCII(0-255) (字符串实际上就是256进制的数字组合)：

- 源字符串为：ABD
