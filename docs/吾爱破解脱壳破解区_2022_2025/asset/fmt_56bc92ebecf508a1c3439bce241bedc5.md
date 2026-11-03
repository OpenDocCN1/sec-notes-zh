
## 反汇编代码

```
__int64 __fastcall sub_14001C2C0(__int64 a1)
{
  unsigned __int64 v1; // rax
  __int64 v2; // r8
  __int64 v3; // r8
  __int64 v4; // r8
  __int64 v5; // r8
  __int64 v6; // r8
  __int64 v7; // r8
  int i; // [rsp+24h] [rbp+4h]
  char v10[48]; // [rsp+1D0h] [rbp+1B0h] BYREF
  char v11[48]; // [rsp+200h] [rbp+1E0h] BYREF
  char v12[48]; // [rsp+230h] [rbp+210h] BYREF
  char v13[48]; // [rsp+260h] [rbp+240h] BYREF
  char v14[48]; // [rsp+290h] [rbp+270h] BYREF
  char v15[48]; // [rsp+2C0h] [rbp+2A0h] BYREF
  char v16[48]; // [rsp+2F0h] [rbp+2D0h] BYREF
  char v17[48]; // [rsp+320h] [rbp+300h] BYREF
  char v18[48]; // [rsp+350h] [rbp+330h] BYREF
  char v19[48]; // [rsp+380h] [rbp+360h] BYREF
  char v20[48]; // [rsp+3B0h] [rbp+390h] BYREF
  char v21[24]; // [rsp+3E0h] [rbp+3C0h] BYREF
  unsigned __int64 v22; // [rsp+3F8h] [rbp+3D8h]

  sub_140012CF8(&unk_14004A0FF);
  for ( i = 0; ; ++i )
  {
    v22 = i;
    v1 = sub_140012AEB(a1 + 4096);
    if ( v22 >= v1 )
      break;
    if ( *sub_14001215E(a1 + 4096, i) == 'R' )
    {
      if ( *sub_14001215E(a1 + 4096, i + 1) == 39 )
      {
        ++i;
        qmemcpy(v10, sub_1400121C7(&unk_140043010, 3i64), 0xCui64);
        sub_14001204B(a1, v10, 0i64);
      }
      else
      {
        qmemcpy(v11, sub_1400121C7(&unk_140043010, 3i64), 0xCui64);
        LOBYTE(v2) = 1;
        sub_14001204B(a1, v11, v2);
      }
    }
    else if ( *sub_14001215E(a1 + 4096, i) == 'U' )
    {
      if ( *sub_14001215E(a1 + 4096, i + 1) == 39 )
      {
        qmemcpy(v12, sub_1400121C7(&unk_140043010, 4i64), 0xCui64);
        sub_14001204B(a1, v12, 0i64);
        ++i;
      }
      else
      {
        qmemcpy(v13, sub_1400121C7(&unk_140043010, 4i64), 0xCui64);
        LOBYTE(v3) = 1;
        sub_14001204B(a1, v13, v3);
      }
    }
    else if ( *sub_14001215E(a1 + 4096, i) == 'F' )
    {
      if ( *sub_14001215E(a1 + 4096, i + 1) == 39 )
      {
        qmemcpy(v14, sub_1400121C7(&unk_140043010, 0i64), 0xCui64);
        sub_14001204B(a1, v14, 0i64);
        ++i;
      }
      else
      {
        qmemcpy(v15, sub_1400121C7(&unk_140043010, 0i64), 0xCui64);
        LOBYTE(v4) = 1;
        sub_14001204B(a1, v15, v4);
      }
    }
    else if ( *sub_14001215E(a1 + 4096, i) == 'L' )
    {
      if ( *sub_14001215E(a1 + 4096, i + 1) == 39 )
      {
        qmemcpy(v16, sub_1400121C7(&unk_140043010, 2i64), 0xCui64);
        LOBYTE(v5) = 1;
        sub_14001204B(a1, v16, v5);
        ++i;
      }
      else
      {
        qmemcpy(v17, sub_1400121C7(&unk_140043010, 2i64), 0xCui64);
        sub_14001204B(a1, v17, 0i64);
      }
    }
    else if ( *sub_14001215E(a1 + 4096, i) == 'D' )
    {
      if ( *sub_14001215E(a1 + 4096, i + 1) == 39 )
      {
        qmemcpy(v18, sub_1400121C7(&unk_140043010, 5i64), 0xCui64);
        sub_14001204B(a1, v18, 0i64);
        ++i;
      }
      else
      {
        qmemcpy(v19, sub_1400121C7(&unk_140043010, 5i64), 0xCui64);
        LOBYTE(v6) = 1;
        sub_14001204B(a1, v19, v6);
      }
    }
    else if ( *sub_14001215E(a1 + 4096, i) == 'B' )
    {
      if ( *sub_14001215E(a1 + 4096, i + 1) == 39 )
      {
        qmemcpy(v20, sub_1400121C7(&unk_140043010, 1i64), 0xCui64);
        sub_14001204B(a1, v20, 0i64);
        ++i;
      }
      else
      {
        qmemcpy(v21, sub_1400121C7(&unk_140043010, 1i64), 0xCui64);
        LOBYTE(v7) = 1;
        sub_14001204B(a1, v21, v7);
      }
    }
  }
  return 0i64;
}
```

## 魔方图

有魔方必有魔方图，花了大量时间来找魔方图发现

![](https://attach.52pojie.cn/forum/202412/09/151026nm6i679m4izwy6pq.png)

`image.png` (39.73 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ5M3wxNjA5Y2RjM3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:10 上传

![](https://attach.52pojie.cn/forum/202412/09/151038gud3gggz1crd4ghb.png)

`image.png` (43.42 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ5NHw3Y2RkZmRjZHwxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:10 上传

他这个图其实是由27×36的列表组成的，动态调试一下发现有个（11，1）与（13，31）这么个玩意儿，于是`dump`下来

![](https://attach.52pojie.cn/forum/202412/09/151056d58lezrrjld44885.png)

`image.png` (37.86 KB, 下载次数: 2)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjQ5NXwyZjM5MmYxY3wxNzkzNTcxNTAwfDE2ODA4NnwxOTg5NjU3&nothumb=yes)

2024-12-9 15:10 上传

## `dump` 的 python 脚本：

```
import struct

def read_signed_dwords_from_addr(addr, length):

    print("")

    for i in range(0, length, 4):

        dword_value = get_wide_dword(addr + i)

        signed_value = struct.unpack('i', struct.pack('I', dword_value))[0]

        print(f"{signed_value},", end="")

read_signed_dwords_from_addr(0x14C8B0, 0xF30)
```

和矩阵排列
