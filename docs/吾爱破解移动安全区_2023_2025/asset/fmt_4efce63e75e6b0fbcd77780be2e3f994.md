
# 加密函数

- `jgbjkb()==0`

```
//调用代码fun_enc(v21, key1, 19, v16);
//这里times为19
void __fastcall fun_enc(_BYTE *a1, char *a2, int times, void *a4)
{
  __int64 v5; // d17
  int i; // r6
  int v9; // r5
  char v10; // r0
  _BYTE v11[16]; // [sp+0h] [bp-30h] BYREF

  v5 = *((_QWORD *)a1 + 1);
  *(_QWORD *)v11 = *(_QWORD *)a1;        //把a1赋值给v11其实就是上面父函数的v21，输入的时候v21是全0的
  *(_QWORD *)&v11[8] = v5;
  if ( times )
  {
    for ( i = 0; i != times; ++i )
    {
      v9 = i & 0xF;        //当循环到i=0、i=16时v9会等于0进入下面的if
      if ( (i & 0xF) == 0 )
        fun_enc2(v11);        //这个函数十分复杂(分析了大半天看不懂放弃了)，会改变v11的内容
      v10 = a2[i] ^ v11[v9];        //v10的值为a2[i]异或v11[v9]，其中a2就是flag，v9是0到15，v11是一个16字节的数组，会在v9=0时发生变化。
      *((_BYTE *)a4 + i) = v10;        //赋值a4，就是父函数的v16
      v11[v9] = v10;        //改变v11的第v9位
    }
  }
}
```

`&unk_CA73F0FC`的值如下：

```
.rodata:0003A0FC unk_3A0FC       DCB 0x48 ; H            ; DATA XREF: sub_BE440+216↓o

.rodata:0003A0FD                 DCB 0x27 ; '

.rodata:0003A0FE                 DCB 0x8F

.rodata:0003A0FF                 DCB 0xAF

.rodata:0003A100                 DCB 0x9B

.rodata:0003A101                 DCB 0xF8

.rodata:0003A102                 DCB 0xEC

.rodata:0003A103                 DCB 0x72 ; r

.rodata:0003A104                 DCB 0x98

.rodata:0003A105                 DCB    7

.rodata:0003A106                 DCB 0x72 ; r

.rodata:0003A107                 DCB  0xC

.rodata:0003A108                 DCB 0x6B ; k

.rodata:0003A109                 DCB 0xE2

.rodata:0003A10A                 DCB 0x3A ; :

.rodata:0003A10B                 DCB 0xB6

.rodata:0003A10C                 DCB 0x42 ; B

.rodata:0003A10D                 DCB 0x59 ; Y

.rodata:0003A10E                 DCB 0xF7

//48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6 42 59 F7
```

## 结论

要想`Check`返回`True`需要`v16=48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6 42 59 F7`

`v16`的值在函数`fun_enc`中赋值，其赋值形式是`flag`与`fun_enc`内的数组`v11`异或（可逆）

那么只需要知道`v11`的内容与48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6 42 59 F7异或回去即可得到`flag`。

需注意！！！`v11`是一个16个字节元素的数组，48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6 42 59 F7（有19个字节）。而`v11`在第1次和17次使用时会被`fun_enc2`改变，所以动态调试的关键点在`v11`的两次变化过程的值

## 0x03 上动态调试

### 第1步：改apk包，`lib`目录只留下`armeabi-v7a`那个文件夹，然后更改权限增加下面两个权限。

```
android:debuggable="true"
android:extractNativeLibs="true"
```

### 第2步：IDA动态调试

![](https://attach.52pojie.cn/forum/202502/14/024932jn3g5fpf53nno0do.jpg)

### 第3步：先异或获取前16位`flag`

```
#i=0时v11的值为
#2E 4B EE C8 E0 95 88 47 B0 72 1B 68 40 D0 0A 84
#与&unk_CA73F0FC异或
#48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6 42 59 F7

# 输入的16进制字符串
hex_str1 = "2E 4B EE C8 E0 95 88 47 B0 72 1B 68 40 D0 0A 84"
hex_str2 = "48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6"

# 将16进制字符串转换为字节列表
bytes1 = bytes.fromhex(hex_str1.replace(" ", ""))
bytes2 = bytes.fromhex(hex_str2.replace(" ", ""))

# 确保两个字节序列长度相同
assert len(bytes1) == len(bytes2), "字节序列长度不匹配"

# 异或操作并转换为字符
result = ''.join(chr(b1 ^ b2) for b1, b2 in zip(bytes1, bytes2))

print("XOR结果转换为字符:", result)
#输出结果：XOR结果转换为字符: flag{md5(uid+202

xor_bytes = bytes(b1 ^ b2 for b1, b2 in zip(bytes1, bytes2))
print("XOR结果的16进制表示:", ' '.join(f'{b:02X}' for b in xor_bytes))
#输出结果：XOR结果的16进制表示: 66 6C 61 67 7B 6D 64 35 28 75 69 64 2B 32 30 32
```

获取到前16位`flag`为`flag{md5(uid+202`：

### 第4步：获取剩下的3位`flag`

把“`flag{md5(uid+202123`”加上随机的3位输入到程序中，查看第17轮即`i=16`时`fun_enc`会把`v11`改成什么

![](https://attach.52pojie.cn/forum/202502/14/024936n88hl3s9ms3r0we9.jpg)

发现`v11`变成了77 70 8A 5F D8 7D B0 5C 90 E6 35 8C D0 4C F9 BB，把新3位放回脚本算出答案

```
# 输入的16进制字符串

hex_str1 = "2E 4B EE C8 E0 95 88 47 B0 72 1B 68 40 D0 0A 84 77 70 8A"

hex_str2 = "48 27 8F AF 9B F8 EC 72 98 07 72 0C 6B E2 3A B6 42 59 F7"

# 将16进制字符串转换为字节列表

bytes1 = bytes.fromhex(hex_str1.replace(" ", ""))

bytes2 = bytes.fromhex(hex_str2.replace(" ", ""))

# 确保两个字节序列长度相同

assert len(bytes1) == len(bytes2), "字节序列长度不匹配"

# 异或操作并转换为字符

result = ''.join(chr(b1 ^ b2) for b1, b2 in zip(bytes1, bytes2))

print("XOR结果转换为字符:", result)

#XOR结果转换为字符: flag{md5(uid+2025)}
```

得出最后答案`flag{md5(uid+2025)}`

## 0x04 拿下`flag`

### `flag`的坑

算出答案非常开心，就去算`uid+2025`的`md5`

```
#UID2121487+2025即2123512
hashlib.md5(str(2123512).encode('utf8')).hexdigest()
'4ac37b3b46367455af865e516fdef7d0'
#得出flag{4ac37b3b46367455af865e516fdef7d0}
#结果提交错误了
```

最后琢磨了以下应该是字符串相加

```
#UID2121487+2025即21214872025

hashlib.md5(str(21214872025).encode('utf8')).hexdigest()

'c9ca680ed9f6b0a6f0cd49cebb626bc0'
```

最终答案`flag{c9ca680ed9f6b0a6f0cd49cebb626bc0}`

## 0x05 推广

- [【2025春节】解题领红包之番外篇一二三WriteUp](https://www.52pojie.cn/thread-2005805-1-1.html)
- [《安卓逆向这档事》十二、大佬帮我分析一下](https://www.52pojie.cn/thread-1809646-1-1.html)

## 0x06 【新年祝福】致52pojie的所有逆向勇士们

转眼又到新年钟声响起时，回想这一年我们熬过的夜——从OD到x64dbg的丝滑切换，从花指令对抗到反调试过招，发际线在堆栈中稳步后移（bushi）。愿新的一年里：

> ① 功力暴涨如IDA反编译F5秒出伪代码
>
> ② 灵感迸发像VS装好了Resharper般丝滑
>
> ③ 遇坑必填，玄学bug自动退散
>
> ④ Hook人生如`Monitor`精准捕获每个高光时刻
>
> ⑤ 在逆向的星辰大海里，永远做那个手握IDA和WinHex的追风少年

祝各位：

永无蓝屏，一调就过！

栈不溢出，堆不泄漏！

早日拿下内核，轻松搞定协议栈！

（防杠声明：本祝福已通过ASLR随机化处理，祝大家逆向功力突破熵增定律）

让咱们继续用代码改变世界，用逆向探索未知！新年卷起来~（手动狗头保命）

❤❤最后感谢论坛大佬的无私奉献❤❤
