
# 逆向分析

## `main` 函数分析

```cpp
__int64 __fastcall main(int a1, char ** a2, char ** a3) {
    __int64 v3; // rax
    std::ostream * v4; // rbx

    __int64 v5; // rax
    _BYTE * v6; // rbp
    char v7; // si
    std::ostream * v8; // rax
    __int64 v9; // rdx
    size_t v10; // rbx

    __int64 v11; // r13
    __int64 v12; // rdx

    __int64 v13; // rbx
    __int64 v14; // rdx

    __int64(__fastcall * v16)(); // rax
    _BYTE v17[32]; // [rsp+0h] [rbp-68h] BYREF
    void * s1; // [rsp+20h] [rbp-48h] BYREF
    size_t n; // [rsp+28h] [rbp-40h]

    v3 = std::operator > ( & std::cout, "Hello! ", a3);
    v4 = (std::ostream * ) std::__ostream_insert > (v3, qword_607AF3F30340, qword_607AF3F30348);
    v5 = * (_QWORD * )( * (_QWORD * ) v4 - 24 LL);
    v6 = * (_BYTE ** )((char * ) v4 + v5 + 240);
    if (!v6)
        std::__throw_bad_cast();
    if (v6[56]) {
        v7 = v6[67];
    } else {
        std::ctype  ::_M_widen_init( * (_QWORD * )((char * ) v4 + v5 + 240));
        v7 = 10;
        v16 = * (__int64(__fastcall ** )())( * (_QWORD * ) v6 + 48 LL);
        if (v16 != std::ctype  ::do_widen)
            v7 = ((__int64(__fastcall * )(_BYTE * , __int64)) v16)(v6, 10 LL);
    }
    v8 = (std::ostream * ) std::ostream::put(v4, v7);
    std::ostream::flush(v8);
    std::operator > ( & std::cout, "Check Flag:", v9);
    std::operator >>  ( & std::cin, & unk_607AF3F30360);
    sub_607AF3F2D5C0(v17, & unk_607AF3F30360, & qword_607AF3F30340);
    sub_607AF3F2D6C0((__int64) & s1, (__int64) v17);
    v10 = n;
    v11 = qword_607AF3F30388;
    v12 = qword_607AF3F30388;
    if (n  0x7FFFFFFF) || v13 > ( & std::cout, "Wrong!", v14);
        goto LABEL_12;
    }
    std::operator > ( & std::cout, "Right!", v14);
    LABEL_12:
        std::string::_M_dispose(v17);
    return 0 LL;
}
```

观察一下，我们可以发现 `sub_607AF3F2D5C0(v17, &unk_607AF3F30360, &qword_607AF3F30340);` 就是加密函数

## 加密函数分析

```cpp
__int64 * __fastcall sub_607AF3F2D5C0(__int64 * a1, char ** a2, _QWORD * a3) {

    __int64 * v3; // rcx

    char * v4; // r12

    __int64 v6; // r15

    int v7; // ebp

    __int64 v8; // rax

    unsigned __int64 v9; // r13

    unsigned __int64 v10; // rax

    int v12; // [rsp+Ch] [rbp-4Ch]

    char * v13; // [rsp+10h] [rbp-48h]

    v3 = a1 + 2;

    *((_BYTE * ) a1 + 16) = 0;

    * a1 = (__int64)(a1 + 2);

    v4 = * a2;

    a1[1] = 0 LL;

    v13 = & a2[1][(_QWORD) v4];

    if (v4 != v13) {

        v6 = 0 LL;

        v7 = 0;

        while (1) {

            v9 = v6 + 1;

            v12 = ( * v4 + * (char * )( * a3 + v7)) % 128;

            v10 = a1 + 2 == v3 ? 15 LL : a1[2];

            if (v10 
```

`v12 = (*v4 + *(char [i])(a3 + v7)) % 128;` 这一句就是主要的加密函数可以简单分析一下各个变量的意思

- `v4` 就是我们输入的值，也就是 `flag(enc)`
- `a3` 就是传入的 `key`，我们可以找到他也就是 `v12=（enc[i]+key[i]）% 128`

所以我们进行解密：

```python
enc=bytes.fromhex("364d4d5c3e387e00421c597a0a7302144d5b70087e064619567336297d151f56770a7935424f2a780643")
key=b'PaluCTF'
flag=''
for i in range(len(enc)):
    flag+=chr((enc[i]-key[i%7])%128)

print(flag)
```

解释一下代码，方便下次自己使用。

`bytes.fromhex` 是将连续的两个数转换为 hex：

```python
hex_string = '616263'

byte_array = bytearray.fromhex(hex_string)

print(byte_array)

bytearray(b'abc')
```

还有一种解法就是爆破，学习一下大佬的答案：

```python
from string import printable

enc = bytearray.fromhex("364d4d5c3e387e00421c597a0a7302144d5b70087e064619567336297d151f56770a7935424f2a780643")

key = "PaluCTF"

for i in range(len(enc)):
    for c in printable:
        if (ord(c) + ord(key[i % len(key)])) % 128 == enc[i]:
            print(c, end="")
            break
```

这里的 `from string import printable`：

```python
# import string library function  

import string  

# Storing the sets of punctuation, 

# digits, ascii_letters and whitespace 

# in variable result  

result = string.printable 

# Printing the set of values  

print(result) 
```

输出结果：

```text
0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&'()*+, -./:;?@[\]^_`{|}~ 
```

返回一系列可打印的值，所以我们使用双重循环进行爆破就ok了

答案是 `flag{d80a0d76-23af-486e-a0bc-43a463eac552}` 剩下两道不是很懂，再分析分析再写吧，帕鲁杯收获还是挺大的，对于加密算法也是又了解到一种。

## 修改记录

- 20240429: 修正排版
