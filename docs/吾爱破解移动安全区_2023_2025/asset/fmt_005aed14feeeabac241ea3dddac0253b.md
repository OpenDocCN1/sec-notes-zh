
有一串长为135的字符串：`******************@******#*******.******.****...******.*****.********.*****.****.....*****.****.*********......**********************`

从 `@` 所在的下标 `[18]` 开始，直到 `[25]` 的 `#` 结束，下标可以 `+-1` 或 `+-15`；必须执行23次这样的运算，且不能落在是 `*` 的地方。

`-1` 称为 `'a'` 操作，`+1` 称为 `'d'` 操作......

我原先是想用搜索算法，不过既然已经写到这里，我意识到了其中的玄只因：135可以被15整除，那串字符串可以格式化为如下：

```
***************
***@******#****
***.******.****
*...******.****
*.********.****
*.****.....****
*.****.********
*......********
***************
```

从 `@` 开始走，`wsad` 是游戏常见的控制上下左右移动，走到终点 `#`，恰好走23步。

于是输入的字符串就是 `ssaassssdddddwwddddwwww`。

但是提示 `flag` 不对。

原来在 `JNI_Onload` 那里，`asc_3B50` 被更改了。

```
jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
  jint v4; // [xsp+44h] [xbp-CCh]
  __int64 v5; // [xsp+48h] [xbp-C8h]
  __int64 env; // [xsp+78h] [xbp-98h] BYREF
  char v7[136]; // [xsp+80h] [xbp-90h] BYREF
  __int64 v8; // [xsp+108h] [xbp-8h]

  v8 = *(_ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2)) + 40);
  strcpy(
    v7,
    "******************@**************.************...****#..*****.********.*****.****.....*****.****.*********......****"
    "*******************");
  v5 = __strlen_chk(v7, 0x88u);
  __memcpy_chk(asc_3B50, v7, v5, 136LL);
  env = 0LL;
  if ( sub_1700(vm, &env, 65540LL) )            // return (a1->functions->FindClass)(a1, a2, a3);
  {
    v4 = -1;
  }
  else
  {
    clazz = sub_173C(env, "com/doctor3/ezandroid/MainActivity");// return a1->functions->FindClass(a1, a2);
    if ( clazz )
    {
      if ( (sub_1770(env, clazz, off_2938, 1u) & 0x80000000) != 0 )// RegisterNatives
        v4 = -1;
      else
        v4 = 65540;
    }
    else
    {
      v4 = -1;
    }
  }
  _ReadStatusReg(ARM64_SYSREG(3, 3, 13, 0, 2));
  return v4;
}
```

真正的 `asc_3B50` 如下：

```
***************
***@***********
***.***********
*...****#..****
*.********.****
*.****.....****
*.****.********
*......********
***************
```

输入字符串应该是 `ssaassssdddddwwddddwwaa`。

结合 `java` 代码，`flag` 是 `moectf{ssaassssdddddwwddddwwaa}`。

## 3> RC4

全局搜索字符串 `flag` 定位到 `sub_140079A70` 函数。

![image.png](https://attach.52pojie.cn/forum/202310/19/213025koeondrxycf7zfn7.png)

[image.png（32.97 KB，下载次数: 2）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MDM0NXxhODhhMDI2NXwxNzkzNTY5MTE3fDE2ODA4NnwxODQ2NDA1&nothumb=yes)

2023-10-19 21:30 上传

```
__int64 sub_140079A70()
{
  char *v0; // rdi
  __int64 i; // rcx
  char v3[32]; // [rsp+0h] [rbp-30h] BYREF
  char v4; // [rsp+30h] [rbp+0h] BYREF
  char v5[256]; // [rsp+40h] [rbp+10h] BYREF
  char v6[256]; // [rsp+160h] [rbp+130h] BYREF
  char v7[44]; // [rsp+278h] [rbp+248h] BYREF
  int v8; // [rsp+2A4h] [rbp+274h]
  int j; // [rsp+2C4h] [rbp+294h]

  v0 = &v4;
  for ( i = 172i64; i; --i )
  {
    *v0 = -858993460;
    v0 += 4;
  }
  sub_14007555C(&unk_1401A7007);
  memset(v5, 0, sizeof(v5));
  memset(v6, 0, sizeof(v6));
  strcpy(v7, "moectf2023");
  v8 = 0;
  sub_140073581("welcome to moectf!!!");
  sub_140073581("This is a very common algorithm ");
  sub_140073581("show your flag:");
  sub_1400727F8("%s", byte_140197260);
  if ( sub_140073829(byte_140197260) == 37 )
  {
    sub_140075052(v5, v6, byte_140197260, 38, v7, 10);  // 看提示，应该在进行rc4加密
    for ( j = 0; j 
```

一共37个字符串，`byte_140197260` 应该是输入的 `flag`。

在经过 `sub_140075052(v5, v6, byte_140197260, 38, v7, 10);` 的处理后：

`byte_140196000[j] == byte_140197260[j]`

看下加密算法：

```
__int64 __fastcall sub_1400795E0(__int64 a1, __int64 a2, __int64 input_flag, int _38, __int64 moectf2023, unsigned int _10)
{
  __int64 result; // rax 10是密钥"moectf2023"的长度
  int i; // [rsp+24h] [rbp+4h] a1 a2 一个是s盒 一个是T盒
  int j; // [rsp+24h] [rbp+4h]
  int v9; // [rsp+24h] [rbp+4h]
  int v10; // [rsp+44h] [rbp+24h]
  int v11; // [rsp+44h] [rbp+24h]
  char v12; // [rsp+64h] [rbp+44h]
  char v13; // [rsp+64h] [rbp+44h]
  int v14; // [rsp+A4h] [rbp+84h]

  result = sub_14007555C(&unk_1401A7007);
  v10 = 0;
  v14 = 0;
  for ( i = 0; i 
```

结合提示，上网搜索 `rc4` 算法，看样子就是它了。

### 解密

```python
def rc4_decrypt(ciphertext, key):
    S = list(range(256))
    j = 0
    out = []

    # Key-scheduling algorithm
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]

    # Pseudo-random generation algorithm
    i = j = 0
    for byte in ciphertext:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(byte ^ S[(S[i] + S[j]) % 256])

    return bytes(out)

ciphertext = [0x1B, 0x9B, 0xFB, 0x19, 0x06, 0x6A, 0xB5, 0x3B, 0x7C, 0xBA, 0x03,
              0xF3, 0x91, 0xB8, 0xB6, 0x3D, 0x8A, 0xC1, 0x48, 0x2E, 0x50,
              0x11, 0xE7, 0xC7, 0x4F, 0xB1, 0x27, 0xCF, 0xF3, 0xAE, 0x03,
              0x09, 0xB2, 0x08, 0xFB, 0xDC, 0x22]

key = "moectf2023"

plaintext = rc4_decrypt(ciphertext, key.encode())
print(plaintext)
```

得到 `moectf{y0u_r3a11y_understand_rc4!!!!}`

## 2> Jail

## 3> level 0

摘自知乎：[https://zhuanlan.zhihu.com/p/60257325](https://zhuanlan.zhihu.com/p/60257325)

```
>>> eval("__import__('os').system('whoami')")

desktop-fa4b888\pythoncat

>>> eval("__import__('subprocess').getoutput('ls ~')")

#结果略，内容是当前路径的文件信息
```

用了第二个命令，输出：

```
Answer result: flag
server.py
```

打开 `flag` 文件再读取：

```
compile("with open('./flag', 'r') as file:\n\tflag_content=file.read(); print(flag_content)", '','exec')
```

得到 `flag` :D
