
```python
        buf1.append(
            ord(inp_name[i]) ^ (i + 1) if i > 1)

    v3 = 0

    for i, v in enumerate(buf2):
        v3 = (v3 & 0xffffff00) | v
        v3 = ror4_1(v3)
        buf2[i] = v3 & 0xff

    return buf2

def solve(inp_name: str):
    ans16_19 = ''.join([chr(i ^ 0x63).upper()
                       for i in [0x16, 0x0f, 0x17]]) + '-'
    print(ans16_19)  # ULT-

    ans_tail = '-2413'
    serial_tail = ''.join([chr(i ^ 0x63) for i in [0x53, 0x52, 0x57, 0x52]])
    print(serial_tail)  # 0141

    buf1 = get_buf1(inp_name)
    buf2 = get_buf2(buf1)

    v14 = []

    def as_signed8(v): return v if v 
```

### 法二

```
#include 
#include 
#include "defs.h"
using namespace std;
#define rep(i,a,b) for(int i = (a);i = (b);--i)

void dbg() {
    puts ("");
}
templatevoid dbg (const T &f, const R &... r) {
    cout inline void read (Type &xx) {
    Type f = 1;
    char ch;
    xx = 0;
    for (ch = getchar(); ch  '9'; ch = getchar() ) if (ch == '-') f = -1;
    for (; ch >= '0' && ch void read (T &x, R &...r) {
    read (x);
    read (r...);
}

string solve (string inpName) {
    int v3; // eax
    int v5; // ecx
    char *v6; // edi
    int v7; // edx
    int v8; // esi
    char *v9; // eax
    int *v12; // edi
    char v14[120]; // [esp+8h] [ebp-328h] BYREF
    char Buf1[38]; // [esp+2A0h] [ebp-90h] BYREF
    char v21; // [esp+2BEh] [ebp-72h]
    char Buf2[38]; // [esp+2C0h] [ebp-70h] BYREF
    char v24; // [esp+2DEh] [ebp-52h]
    int v26; // [esp+2E4h] [ebp-4Ch]
    char v29; // [esp+32Fh] [ebp-1h]
    int dword_407030[120] = {
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
    };
    memset (Buf1, 0, sizeof (Buf1) );
    memset (Buf2, 0, sizeof (Buf2) );
    memset (v14, 0, sizeof (v14) );
    re_ (i, 0, 30) {
        if (i >= inpName.size() )
            Buf1[i] = (i + (dword_407030[4 * i] ^ 0x5A) ) % 57 + 65;
        else
            Buf1[i] = (i + 1) ^ inpName[i];
    }
    v21 = 0;
    for (int i = 0; i 
```

### 参考资料

- [`movsx`命令](https://blog.csdn.net/cswangbin/article/details/3955395)
- [`BV16d4y1d72A`](https://www.bilibili.com/video/BV16d4y1d72A)
