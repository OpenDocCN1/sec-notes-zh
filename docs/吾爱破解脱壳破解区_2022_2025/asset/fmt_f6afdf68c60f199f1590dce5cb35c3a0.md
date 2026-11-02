
```
    switch ( (unsigned __int16)a3 )
    {
      case 1u:
        v10 = sub_140001A20(a1);
        if ( v10 )
        {
          if ( (int)sub_140001FC0(a1, v18) > 0 )
          {
            v12 = sub_140002110(v18, v10);
            sub_140002060(a1, 0x111i64, 0x300i64, v12);
            result = 1i64;
          }
          else
          {
            result = 0i64;
          }
        }
   //...
   }
```

`sub_140002060` 中的 `0x111` 和 `0x300` 很像进入判断分支的信号量，其中的 `a4` 由 `v12` 传入，`v12` 由 `sub_140002110` 的返回值决定，因此，我们在 `x64dbg` 中定位到 `0x140002110`，修改代码让它直接返回 4 就能实现爆破了：

按 `Ctrl + G`，粘贴函数地址跳转

![](https://attach.52pojie.cn/forum/202302/06/171314ileyemmbybylssaz.png)

052d4fab2e458009e6c6c4c64a2acacf_1c975811c0bb495fbc625da956fec24b.png (13.18 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjU4ODc0OXxkNjBiMmU0Y3wxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc0OXxkNjBiMmU0Y3wxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:13 上传

![](https://attach.52pojie.cn/forum/202302/06/171429m2dzz0dbhzkdip9d.png)

019d852bb97dcd0ce93b4ca73031a431_12ee75cad1eb45e98240a811156a754b.png (74.15 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc1MHxmOTE2MzMyOXwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1MHxmOTE2MzMyOXwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:14 上传

按下空格照下入修改函数头，直接返回 4

![](https://attach.52pojie.cn/forum/202302/06/171451tkfuf2m132uuu3ko.png)

0c5d99250b2e15ed906cdc2e0b610a4d_3cc7243c4a8d4b8b9406bc52e373a1c0.png (43.46 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc1MXwzOTNhMmRiMnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1MXwzOTNhMmRiMnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:14 上传

再次提交 `flag`，提示 `Success`

![](https://attach.52pojie.cn/forum/202302/06/171506j0kqekqr5fej50f4.png)

27675067d2f26af66f366da50c3234ac_90920ac4390f445e9fff842bd1dc7a0e.png (23.78 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjU4ODc1MnxlN2E3MDRmYnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1MnxlN2E3MDRmYnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:15 上传

## 3> 坑

那么接下来事情【似乎】就很简单了，我们分析 `sub_140002110` 的逻辑，并找出让它返回 4 的条件就行了。如果你这么想，恭喜你掉坑里了！我们不妨先进来看看这个函数，调试结果表明第一个参数是 `flag`，第二个是转成整数的 `uid`，为了方便阅读这里对部分变量进行了重命名，在 `IDA` 中选择对应的变量按 `N` 即可实现：

```
__int64 __fastcall sub_140002110(const wchar_t *flag, int uid)
{
  unsigned __int64 v2; // kr00_8
  int k; // [rsp+20h] [rbp-3A8h]
  int j; // [rsp+24h] [rbp-3A4h]
  int v6; // [rsp+28h] [rbp-3A0h]
  int v7; // [rsp+28h] [rbp-3A0h]
  int map_uid; // [rsp+2Ch] [rbp-39Ch]
  int v9; // [rsp+30h] [rbp-398h]
  int v10; // [rsp+30h] [rbp-398h]
  unsigned int v11; // [rsp+38h] [rbp-390h]
  int len_flag; // [rsp+3Ch] [rbp-38Ch]
  int v13; // [rsp+40h] [rbp-388h]
  int v14; // [rsp+44h] [rbp-384h]
  unsigned int v15; // [rsp+48h] [rbp-380h]
  int map_uid_arr[6]; // [rsp+58h] [rbp-370h] BYREF
  int v17[104]; // [rsp+70h] [rbp-358h] BYREF
  int Destination[104]; // [rsp+210h] [rbp-1B8h] BYREF

  len_flag = wcslen(flag);
  memcpy_s(Destination, 0x38ui64, qword_1400168F0, 0x38ui64);
  flag[len_flag - 1] ^= 0x7Du;
  v6 = -1;
  for ( map_uid = uid - 1; map_uid >= 0; map_uid = 2 * map_uid + 9 )
    ;
  for ( j = 0; j 
```

大家看看这里有几个变量是跟 `flag` 有关的，除了一开始煞有介事地取了 `flag` 的长度，还做了字符串截取外，后续的 `sub_140001D70(&v17[k], map_uid_arr, (unsigned int)map_uid);` 似乎也与 `flag` 有关，看上去似乎关键点就是在这里了！注意到最终返回的是 `v15`，而 `v15` 要么为 0，要么为 `v11`，`v11` 是 `sub_140001D70` 决定的，只要确保最后一次调用返回的是 4 就行了。`sub_140001D70` 的伪码如下，看起来已经【非常像】解密函数了：

```
__int64 __fastcall sub_140001D70(unsigned int *a1, _DWORD *a2, int a3)
{
  unsigned int v4; // [rsp+0h] [rbp-28h]
  unsigned int v5; // [rsp+4h] [rbp-24h]
  unsigned int v6; // [rsp+8h] [rbp-20h]
  unsigned int i; // [rsp+Ch] [rbp-1Ch]

  v4 = *a1;
  v5 = a1[1];
  v6 = 0;
  for ( i = 0; i > 4)) ^ (v6 + v5) ^ (*a2 + 32 * v5);
    v5 += (a2[3] + (v4 >> 4)) ^ (v6 + v4) ^ (a2[2] + 32 * v4);
  }
  *a1 = v4;
  a1[1] = v5;
  return v6;
}
```

首先还是尝试爆破让这个函数直接返回 4，不出意料程序提示 `Success`。

但是仔细看下，这里的返回值完全是由 `a3(map_uid)` 决定的，与 `flag` 无关！我这边一开始还以为是伪码出了问题，后面经调试和阅读反汇编，发现伪码是正确的，返回值的确与 `flag` 无关，排这个坑用了一天多的时间，这里也大概写下过程吧。

首先，怎样让它返回 4 呢？让 `v6` 在 `0x20`（32）次循环累加后值等于 4 即可！但在整数运算中 `4/32 = 0`，这明显是不可能的，不过考虑到无符号整数的溢出问题也不是完全没戏。但这些运算看着就头疼，因此我写了代码想从中找到可以直接通过验证的 `uid`，程序跑了几分钟也没给我找出来，跑的范围已经远远超过论坛 `uid` 的实际范围了。再加上已经有人成功提交，我这才完全明确程序走的不是这个分支。

一天过去。

## 3> 再次分析

后面的分析就基本不需要用 `x64dbg` 了，因为显然用 `IDA` 调试伪码更方便。

同样要靠附加运行，不能直接调试。将程序拖入 `IDA`，同时运行，并在 `IDA` 中选择本地 Windows 调试器：

![](https://attach.52pojie.cn/forum/202302/06/171535gh9why9r3shjwtiy.png)

b46f70489ed1dd73f6a8765558220dbe_9dc3b5db2fd445399d18f9da584511b5.png (34.61 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjU4ODc1M3xhODQ1M2M2YXwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1M3xhODQ1M2M2YXwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:15 上传

然后就可以附加了：

![](https://attach.52pojie.cn/forum/202302/06/171547h1rrmdwdxcs0cn1o.png)

14eb30613acfb18c2de17299ee0cdfd8_17e18feae75541afbb3d3b80683b6feb.png (92.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc1NHw3ODUzNjZiYnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1NHw3ODUzNjZiYnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:15 上传

重新定位到 `sub_1400011D0` 函数，这次我们直接拿伪码下断点：

![](https://attach.52pojie.cn/forum/202302/06/171557n5tux3x3z154a4y3.png)

0c447042d245d2b7d479a973d13db066_7548dee7677a40b299cff0466b5c8816.png (75.41 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc1NXw2ZTIxYmIyY3wxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1NXw2ZTIxYmIyY3wxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:15 上传

准备好后，按下 `F9` 让程序运行。

可以看到 `a4` 是一个非常奇怪的数，而且前面的分析也表明了它不可能是 4，同样也不可能是 1, 2, 3 中的任何一个，所以程序只能走 `default` 分支

![](https://attach.52pojie.cn/forum/202302/06/171609gukqee6ertr6fezk.png)

72ee6d8aef3065581b08692b5568b56d_1f2c706593ba4817b49ecd6fe241cac1.png (46.27 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc1Nnw0MWI5ZDk0NHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1Nnw0MWI5ZDk0NHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:16 上传

接下来再看看 `v6`：

![](https://attach.52pojie.cn/forum/202302/06/171620xqb6en288h8dtahb.png)

506f834d901bc476323e40ea08daa5ec_04ffcf82a02e40a2a643cc4d3e529905.png (19.68 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjU4ODc1N3xkNTBiNTVmYnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1N3xkNTBiNTVmYnwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:16 上传

`v6` 为空？！这和之前假设的不一样！

接着，程序走到了这里，然后又是熟悉的 `2060`, `0x111`, `0x300`：

![](https://attach.52pojie.cn/forum/202302/06/171631r3ykbtq36dd9blul.png)

4fdf17155f04c2d926b2326e859a7fcf_4b67d4f7b3784a67863a751226999b5a.png (24.84 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU4ODc1OHw2ZTVhMWIxOHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4ODc1OHw2ZTVhMWIxOHwxNzkzNTc0MDY3fDE2ODA4NnwxNzQyMzM0&nothumb=yes)

2023-2-6 17:16 上传

用 `x64dbg` 定位到 `sub_140002A50`，令它直接返回 4，同样弹框提示 `Success`！

另外，经多次调试，我发现程序第一次经过 `v6` 时，`v6` 必然为空，而 `sub_140002A50` 的逻辑同时与 `flag` 和 `uid` 有关，也就是说 `sub_140002A50` 函数才是真正决定你能否通过验证的函数！

现在让我们重新认识一下上两张图的这段代码，注释内容均为调试得出，这里的 `map` 表示映射，`uid_map` 就是说这个数值是只和 `uid` 有关的，与 `flag` 无关：

```
        if ( v6 )

        {

                        // 程序会经过两次，第一次 v6 一定是空的

          v13 = Block + 75;

          wsprintfW(v20, Block, String1);

          ((void (__fastcall *)(__int64, WCHAR *, LPCWSTR, __int64))qword_140017C90[5])(hwind, v20, v13, 16 L);

          result = 1 L;

        }

        else

        {

                        // 所以第一次会跑这里再检查一遍, uid_map 应该是与uid有关的值映射，可能与flag无关
```
