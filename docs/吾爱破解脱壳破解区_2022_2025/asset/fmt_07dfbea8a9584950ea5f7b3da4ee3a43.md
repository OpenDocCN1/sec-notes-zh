
# [原创] 超时代9.25版本有密码提取分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1774776
- 抓取时间: 2026-11-01T22:56:31

## 提要

首先添加一个全部为数据为了0的文件，把文件结尾改成视频格式。然后加密就可以看到他的加密方式了。一共是3种加密，
首先会把前256个字节使用第3种加密方式加密，然后每次读取1024个字节进行加密。
分别为第1种加密16个字节，这个里面分别进行了2次查表异或加密和移位等操作。
第2种加密是240字节，和一个固定的字符串进行异或，类似与9.36的版本的加密。
第3种加密是256字节，从0个字节开始和0-255进行异或，进行3次加密，加在一起就是1024个字节。
结尾的时候会判断最后一次读取的字节如果小于16字节就不会进行加密。如果大于16字节小于240字节就会进行第一种加密，然后剩余的不进行加密，如果大于256个字节就会进行使用第1种和第2种加密方式，剩余的不进行加密。以此类推。

9.25的解密开始地址为 `F17F18`

![](https://attach.52pojie.cn/forum/202304/17/005901zipzqlwdkwipwewz.png)

22222222222222222.png (64.9 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYwODA0MXwwNjcwNzMxZXwxNzkzNTczNzg4fDE2ODA4NnwxNzc0Nzc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwODA0MXwwNjcwNzMxZXwxNzkzNTczNzg4fDE2ODA4NnwxNzc0Nzc2&nothumb=yes)

1

2023-4-17 00:59 上传

由于9.25进行的附加保护，不像9.36可以直接附加。需要对进行先过保护才能附加。方法是论坛大佬提供的。

1. `od`打开加密后的视频。
2. 对`WaitForDebugEvent`进行下段点。运行30-40多次其中运行，如果出现卡顿一下后继续运行3-5后。去到`00401000`。

修改代码如下，`eip`更新为`00401000`，运行完后就可以取消`WaitForDebugEvent`断点，把程序运行起来。在新打开一个OD进行附加`player.dll`

这个时候就可以附加上去了。

```
00401000,"pushad"
00401001,"pushfd"
00401002,"push 0x12FD54"
00401007,"call DebugActiveProcessStop"
0040100c,"popfd"
0040100d,"popad"
0040100e,"jmp WaitForDebugEvent"
```
这个值为程序启动后有一个`player.dll`的程序PID号

把视频播放出来。对`ReadFile`进行下段，单步更下去。回来到`player.dll`领空，这个时候就要注意了。

```
64501B90    55              push ebp
64501B91    8BEC            mov ebp,esp
64501B93    57              push edi
64501B94    8BF8            mov edi,eax
64501B96    837F 0C 00      cmp dword ptr ds:[edi+0xC],0x0
64501B9A    0F84 E6000000   je libwzpla.64501C86
64501BA0    837F 10 00      cmp dword ptr ds:[edi+0x10],0x0
64501BA4    0F84 DC000000   je libwzpla.64501C86
64501BAA    837F 14 00      cmp dword ptr ds:[edi+0x14],0x0
64501BAE    56              push esi
64501BAF    8D77 14         lea esi,dword ptr ds:[edi+0x14]
64501BB2    0F84 CD000000   je libwzpla.64501C85
64501BB8    53              push ebx
64501BB9    BB 04000000     mov ebx,0x4
64501BBE    8BFF            mov edi,edi
64501BC0    8B07            mov eax,dword ptr ds:[edi]
64501BC2    8B0E            mov ecx,dword ptr ds:[esi]
64501BC4    50              push eax
64501BC5    6A 00           push 0x0
64501BC7    51              push ecx
64501BC8    E8 43766B00     call libwzpla.64BB9210
64501BCD    83C4 0C         add esp,0xC
64501BD0    83C6 04         add esi,0x4
64501BD3    4B              dec ebx
64501BD4  ^ 75 EA           jnz short libwzpla.64501BC0
64501BD6    8B17            mov edx,dword ptr ds:[edi]
64501BD8    03D2            add edx,edx
64501BDA    03D2            add edx,edx
64501BDC    33C0            xor eax,eax
64501BDE    85D2            test edx,edx
64501BE0    7E 22           jle short libwzpla.64501C04
64501BE2    8B75 08         mov esi,dword ptr ss:[ebp+0x8]
64501BE5    8A1C30          mov bl,byte ptr ds:[eax+esi]                         ; esi是保存这需要进行解密的16个字节
64501BE8    8BC8            mov ecx,eax
64501BEA    83E1 03         and ecx,0x3
64501BED    8B4C8F 14       mov ecx,dword ptr ds:[edi+ecx*4+0x14]                ; 这个会申请4个4字节地址
64501BF1    8BD0            mov edx,eax
64501BF3    C1FA 02         sar edx,0x2
64501BF6    881C0A          mov byte ptr ds:[edx+ecx],bl                         ; 这个里会对文件进行分开存储16个字
64501BF9    8B17            mov edx,dword ptr ds:[edi]                           ; 第一个地址存储1 5 9 13
64501BFB    03D2            add edx,edx                                          ; 第二个地址存储2 6 10 14
64501BFD    40              inc eax                                              ; 第三个地址存储3 7 11 15
64501BFE    03D2            add edx,edx                                          ; 第四个地址存储4 8 12 16
64501C00    3BC2            cmp eax,edx
64501C02  ^ 7C DE           jl short libwzpla.64501BE2
64501C04    8B4F 08         mov ecx,dword ptr ds:[edi+0x8]
64501C07    8BC7            mov eax,edi                                          ; 这里进行9次循环解密
64501C09    E8 72F4FFFF     call libwzpla.64501080                               ; 表1
64501C0E    8B77 08         mov esi,dword ptr ds:[edi+0x8]
64501C11    4E              dec esi
64501C12    5B              pop ebx
64501C13    83FE 01         cmp esi,0x1
64501C16    7C 22           jl short libwzpla.64501C3A
64501C18    8BCF            mov ecx,edi
64501C1A    E8 D1F5FFFF     call libwzpla.645011F0                               ; 移位
64501C1F    8BC7            mov eax,edi
64501C21    E8 4AF5FFFF     call libwzpla.64501170                               ; 表2
64501C26    8BCE            mov ecx,esi
64501C28    8BC7            mov eax,edi
64501C2A    E8 51F4FFFF     call libwzpla.64501080                               ; 表1
64501C2F    E8 0CF7FFFF     call libwzpla.64501340                               ; 表3
64501C34    4E              dec esi
64501C35    83FE 01         cmp esi,0x1
64501C38  ^ 7D DE           jge short libwzpla.64501C18
64501C3A    8BCF            mov ecx,edi
64501C3C    E8 AFF5FFFF     call libwzpla.645011F0                               ; 移位
64501C41    8BC7            mov eax,edi
64501C43    E8 28F5FFFF     call libwzpla.64501170                               ; 表2
64501C48    33C9            xor ecx,ecx
64501C4A    8BC7            mov eax,edi
64501C4C    E8 2FF4FFFF     call libwzpla.64501080                               ; 表1
64501C51    8B0F            mov ecx,dword ptr ds:[edi]
64501C53    03C9            add ecx,ecx
64501C55    03C9            add ecx,ecx
64501C57    33C0            xor eax,eax
64501C59    85C9            test ecx,ecx
64501C5B    7E 28           jle short libwzpla.64501C85
64501C5D    8D49 00         lea ecx,dword ptr ds:[ecx]
64501C60    8BD0            mov edx,eax
64501C62    83E2 03         and edx,0x3
64501C65    8B5497 14       mov edx,dword ptr ds:[edi+edx*4+0x14]
64501C69    8BC8            mov ecx,eax
64501C6B    C1F9 02         sar ecx,0x2
64501C6E    8A0C0A          mov cl,byte ptr ds:[edx+ecx]
64501C71    8B55 0C         mov edx,dword ptr ss:[ebp+0xC]
64501C74    80F1 0A         xor cl,0xA                                           ; 最后和A进行异或
64501C77    880C10          mov byte ptr ds:[eax+edx],cl
64501C7A    8B0F            mov ecx,dword ptr ds:[edi]                           ; 异或完后还要进行最后排序
64501C7C    03C9            add ecx,ecx
64501C7E    40              inc eax
64501C7F    03C9            add ecx,ecx
64501C81    3BC1            cmp eax,ecx
64501C83  ^ 7C DB           jl short libwzpla.64501C60
64501C85    5E              pop esi
64501C86    5F              pop edi
64501C87    5D              pop ebp
64501C88    C3              retn
```

表一：

```
64501080    8B50 10         mov edx,dword ptr ds:[eax+0x10]          ; eax地址保存这个程序的特定值生成的解密表  
                                                                        每个生成的视频都是随机的。无密码提取就需要追到这个表是怎么生成的。
64501083    56              push esi                                 ; eax保存的表首，一共是有11组 从下往上读取
64501084    8B70 14         mov esi,dword ptr ds:[eax+0x14]
64501087    03C9            add ecx,ecx
64501089    03C9            add ecx,ecx
6450108B    0FB6148A        movzx edx,byte ptr ds:[edx+ecx*4]
6450108F    3016            xor byte ptr ds:[esi],dl                 ; dl是解密表的最后一行的第一个值，esi是解密字节的第一个值
64501091    8B50 10         mov edx,dword ptr ds:[eax+0x10]
64501094    0FB6548A 04     movzx edx,byte ptr ds:[edx+ecx*4+0x4]
64501099    8B70 14         mov esi,dword ptr ds:[eax+0x14]
6450109C    3056 01         xor byte ptr ds:[esi+0x1],dl             ; dl是解密表的最后一行的第二个值，esi是解密字节的第二个值
6450109F    8B50 10         mov edx,dword ptr ds:[eax+0x10]
645010A2    0FB6548A 08     movzx edx,byte ptr ds:[edx+ecx*4+0x8]
645010A7    8B70 14         mov esi,dword ptr ds:[eax+0x14]
645010AA    3056 02         xor byte ptr ds:[esi+0x2],dl             ; dl是解密表的最后一行的第三个值，esi是解密字节的第三个值
645010AD    8B50 10         mov edx,dword ptr ds:[eax+0x10]
```
