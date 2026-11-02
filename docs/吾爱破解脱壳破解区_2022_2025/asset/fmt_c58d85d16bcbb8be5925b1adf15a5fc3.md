
# [CTF] 【2023春节】解题领红包活动 二、三、四、五、WEB一部分 Writeup

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1742345
- 抓取时间: 2026-11-01T23:01:02

## 解题领红包之二 {Windows 初级题}

[//down.52pojie.cn/dHmjGF.7z" target="_blank" rel="noopener noreferrer nofollow"](https://down.52pojie.cn/dHmjGF.7z:href="https) | PassWord：vX5lHs8adoRk

运行一下：

```
>【2023春节】解题领红包之二.exe
Please input password: 
hello
Length Error, please try again
Press any key to continue . . .
```

IDA看一下：

```
v4 = _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc((int)&dword_4433D0, "Please input password: ");
_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_(v4);
_ZStrsIcSt11char_traitsIcESaIcEERSt13basic_istreamIT_T0_ES7_RSbIS4_S5_T1_E(&dword_443470, v14);
if ( *(_DWORD *)(v14[0] - 12) == 29 )
{
    v13 = 0;
    while ( 1 )
    {
      if ( *(int *)(v14[0] - 12 + 8) >= 0 )
        _ZNSs12_M_leak_hardEv(v14);
      if ( *(_BYTE *)(v14[0] + v13) != (unsigned __int8)(dword_43F000[v13] >> 2) )
        break;
      if ( (unsigned int)++v13 >= *(_DWORD *)(v14[0] - 12) )
      {
        v7 = _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc((int)&dword_4433D0, "Success");
        _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_(v7);
        system("Pause");
```

也就是长度等于29，然后列表`dword_43F000`的每个元素右移2位和输入比较

写个脚本：

```
encoded_list = [408, 432, 388, 412, 492, 212, 200, 320, 444, 296, 420, 404, 200, 192, 200, 204, 288, 388, 448, 448, 484, 312, 404, 476, 356, 404, 388, 456, 500]

result = []
for a in encoded_list:
    result.append(a >> 2)

result = bytes(result).decode()
print(result)

# flag{52PoJie2023HappyNewYear}
```

## 解题领红包之三 {Android 初级题}

[//down.52pojie.cn/nUvaFj.7z" target="_blank" rel="noopener noreferrer nofollow"](https://down.52pojie.cn/nUvaFj.7z:href="https) | PassWord：zYchSGxanOOx

jadx打开看看，关键逻辑：

```
key.setText(this$0.decrypt("hnci}|jwfclkczkppkcpmwckng\u007f", 2));

public final String decrypt(String encryptTxt, int i) {
    Intrinsics.checkNotNullParameter(encryptTxt, "encryptTxt");
    char[] charArray = encryptTxt.toCharArray();
    StringBuilder sb = new StringBuilder();
    for (char c : charArray) {
        sb.append((char) (c - i));
    }
    String sb2 = sb.toString();
    return sb2;
}
```

写脚本解密即可：

```
encrypted = "hnci}|jwfclkczkppkcpmwckng\u007f"
result = []
for a in encrypted:
    result.append(ord(a) - 2)

result = bytes(result).decode()
print(result)

# flag{zhudajiaxinniankuaile}
```

## 解题领红包之四 {Android 初级题}

[//down.52pojie.cn/JfCdrX.7z" target="_blank" rel="noopener noreferrer nofollow"](https://down.52pojie.cn/JfCdrX.7z:href="https) | PassWord：5dPxREzsOa89

jadx打开，关键逻辑如下：

```
if (a.B(obj, StringsKt.trim((CharSequence) editText2.getText().toString()).toString())) {
    Toast.makeText(this$0, "恭喜你，flag正确!", 1).show();
} else {
    Toast.makeText(this$0, "flag错误哦，再想想！", 1).show();
}

// str是UID，str2是flag
public final boolean B(String str, String str2) {
    Intrinsics.checkNotNullParameter(str, "str");
    Intrinsics.checkNotNullParameter(str2, "str2");
    if (!(str.length() == 0 && str2.length() == 0) && StringsKt.startsWith$default(str2, "flag{", false, 2, (Object) null) && StringsKt.endsWith$default(str2, "}", false, 2, (Object) null)) {
        String substring = str2.substring(5, str2.length() - 1);
        Intrinsics.checkNotNullExpressionValue(substring, "this as java.lang.String…ing(startIndex, endIndex)");
        C c = C.INSTANCE;
        MD5Utils mD5Utils = MD5Utils.INSTANCE;
        Base64Utils base64Utils = Base64Utils.INSTANCE;
        String encode = B.encode(str + "Wuaipojie2023");
        Intrinsics.checkNotNullExpressionValue(encode, "encode(str3)");
        byte[] bytes = encode.getBytes(Charsets.UTF_8);
        Intrinsics.checkNotNullExpressionValue(bytes, "this as java.lang.String).getBytes(charset)");
        return Intrinsics.areEqual(substring, c.cipher(mD5Utils.MD5(base64Utils.encodeToString(bytes)), 5));
    }
    return false;
}

public class B {
    public static String encode(String str) {
        int length = str.length();
        char[] cArr = new char[length];
        int i = length - 1;
        while (i >= 0) {
            int i2 = i - 1;
            cArr[i] = (char) (str.charAt(i) ^ '5');
            if (i2 = 0 && Intrinsics.compare((int) str.charAt(i2), 90)  0) {
                sb.append(str.charAt(i2));
            } else {
                sb.append(cipher(str.charAt(i2), i));
            }
        }
        String sb2 = sb.toString();
        Intrinsics.checkNotNullExpressionValue(sb2, "sb.toString()");
        return sb2;
    }
}
```

比较简单的方法应该是在最终比较的时候直接输出计算好的flag

也可以自己看加密逻辑，用脚本重新实现一遍

```
import base64
from hashlib import md5

uid = "445991"
tmp_str = uid + "Wuaipojie2023"

tmp_str_len = len(tmp_str)
tmp_list = [0] * tmp_str_len
i = tmp_str_len - 1
# 简单的异或
while i >= 0:
    i2 = i - 1
    tmp_list[i] = ord(tmp_str[i]) ^ ord("5")
    if i2 = 65 and ord_a = 97 and ord_a = 65 and ord_a
```

## 解题领红包之五 {Windows 中级题}

[//down.52pojie.cn/RRxUhX.7z" target="_blank" rel="noopener noreferrer nofollow"](https://down.52pojie.cn/RRxUhX.7z:href="https) | PassWord：CGSZnTvK0fwB

file type: `PE32+` executable (GUI) x86-64, for MS Windows

加壳了

直接调试，异常退出，所以让程序运行起来再附加

`MessageBoxW`

可以找到dump的地方 `0x140001000`，dump之后慢慢分析

`445991` `0x6ce27`

`1111111122222222333333334444444455555555666666667777777788888888` 可以确定flag长度64，而且全部是16进制字符，没有"flag{..}"这种形式

最终比较结果是 "flag{!!!_HAPPY_NEW_YEAR_2023!!!}"

假的的关键逻辑：

```
__int64 __fastcall real_key_func(__int64 input_key, int uid)
{
  unsigned __int64 v2; // kr00_8
  int k; // [rsp+20h] [rbp-3A8h]
  int j; // [rsp+24h] [rbp-3A4h]
  int v6; // [rsp+28h] [rbp-3A0h]
  int v7; // [rsp+28h] [rbp-3A0h]
  int changed_uid; // [rsp+2Ch] [rbp-39Ch]
  int v9; // [rsp+30h] [rbp-398h]
  int v10; // [rsp+30h] [rbp-398h]
  unsigned int v11; // [rsp+38h] [rbp-390h]
  int v12; // [rsp+3Ch] [rbp-38Ch]
  int v13; // [rsp+40h] [rbp-388h]
  int v14; // [rsp+44h] [rbp-384h]
  unsigned int v15; // [rsp+48h] [rbp-380h]
  int v16[6]; // [rsp+58h] [rbp-370h] BYREF
  int the_content[104]; // [rsp+70h] [rbp-358h] BYREF
  int expect_result[104]; // [rsp+210h] [rbp-1B8h] BYREF

  v12 = end_point((_WORD *)input_key);
  the_copy((__int64)expect_result, 56ui64, (__int64)dword_1400168F0, 56ui64);
  *(_WORD *)(input_key + 2i64 * (unsigned int)(v12 - 1)) ^= '}';// 去掉结尾的"}"
  v6 = 0xFFFFFFFF;
  for ( changed_uid = uid - 1; changed_uid >= 0; changed_uid = 2 * changed_uid + 9 )
    ;
  for ( j = 0; j > 4)) ^ (v6 + v5) ^ (*a2 + 32 * v5);
    v5 += (a2[3] + (v4 >> 4)) ^ (v6 + v4) ^ (a2[2] + 32 * v4);
  }
  *a1 = v4;
  a1[1] = v5;
  return v6;
}
```

`dword_1400168F0`经过简单解密是: "!!!!_HAPPY_NEW_YEAR_2023!!!!"

真的关键逻辑：

```
__int64 __fastcall decrypt_happy_new_year(__int64 key_hex_num, int uid, unsigned int a3)
{
  int i; // [rsp+20h] [rbp-188h]
  int j; // [rsp+20h] [rbp-188h]
  int k; // [rsp+20h] [rbp-188h]
  int l; // [rsp+20h] [rbp-188h]
  int changed_uid; // [rsp+24h] [rbp-184h]
  int v9; // [rsp+28h] [rbp-180h]
  int v10; // [rsp+28h] [rbp-180h]
  int v11; // [rsp+2Ch] [rbp-17Ch]
  int v12; // [rsp+30h] [rbp-178h]
  unsigned int v13; // [rsp+34h] [rbp-174h]
  int v14; // [rsp+38h] [rbp-170h]
  int v15; // [rsp+3Ch] [rbp-16Ch]
  int expect_result; // [rsp+40h] [rbp-168h]
  char v17; // [rsp+44h] [rbp-164h]
  char v18; // [rsp+5Fh] [rbp-149h]
  char v19; // [rsp+60h] [rbp-148h]
  int v20[52]; // [rsp+B0h] [rbp-F8h]
  int with_uid_list[4]; // [rsp+180h] [rbp-28h] BYREF
  if ( !key_hex_num )
    return 0i64;
  v11 = 0x11111111;
  for ( i = 0; i = 0; changed_uid = 2 * changed_uid + 9 )
    ;
  for ( k = 0; k > 1;
  else
    v13 = 3;
  return v13;
}

__int64 __fastcall sub_1400026E0(unsigned int *a1, _DWORD *with_uid_list, int changed_uid, unsigned int a4)
{
  unsigned int v5; // [rsp+0h] [rbp-28h]
  unsigned int v6; // [rsp+4h] [rbp-24h]
  unsigned int i; // [rsp+8h] [rbp-20h]
  v5 = *a1;
  v6 = a1[1];
}
```
