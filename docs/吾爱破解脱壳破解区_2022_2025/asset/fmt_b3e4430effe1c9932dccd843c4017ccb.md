
```asm
.text:0804C3B4 C7 04 24 21 00 00 00 mov dword ptr [esp], 21h ;
```

再看 Data Dump 区 `ff9d0d70` 里的数据

![](https://attach.52pojie.cn/forum/202507/23/163517famnoj1maxh75zta.jpg)

[Snap11.jpg (8.78 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMyMnwwNjQ4ODVhNXwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:35 上传

总结下，其实就是 Registration key = `sub_804DC20`（10237010）= 3587C9F4FAF0316EC32488E937AE4D70

这个`sub_804DC20`是个什么功能呢？

IDA 中用用 `FindCrypt3` 看一下

![](https://attach.52pojie.cn/forum/202507/23/164256vwxyfl1wfy255axx.jpg)

[Snap12.jpg (128.69 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMzMHw0MzMzOTA1OHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:42 上传

双击第一行的地址跳转到下面的地址

![](https://attach.52pojie.cn/forum/202507/23/164421pok7ik5oovfk5rlv.jpg)

[Snap13.jpg (233.33 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMzMXxmZGYzODQyMnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:44 上传

`Function Call` 看一下

![](https://attach.52pojie.cn/forum/202507/23/164707utiagtit79xeipx5.jpg)

[Snap14.jpg (27.06 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMzMnw0YjZhM2UwMHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:47 上传

所以`sub_804DC20`其实就是 MD5

验证下

![](https://attach.52pojie.cn/forum/202507/23/164908qhuzyq2hqu2131mj.jpg)

[Snap15.jpg (25.38 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMzNnw1ZGYzODRlZnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:49 上传

不就是 Registration key = `sub_804DC20`（10237010）= 3587C9F4FAF0316EC32488E937AE4D70

下面再看下这个 10237010 怎么来的。

`edb` 重 run，在 0804C348 下断点

```asm
.text:0804C348 89 44 24 04 mov [esp+4], eax ; filename
.text:0804C34C E8 9F C9 FF FF call ___xstat
.text:0804C351 8B 85 74 FF FF FF mov eax, [ebp+var_8C]
```

断下后看 eax，是软件的目录

![](https://attach.52pojie.cn/forum/202507/23/165348scnrrs1irnzkwhqh.jpg)

[Snap16.jpg (3.02 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTM0MXw0MjQyMzZlZnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:53 上传

两次 F8，run 过 0804C34C 停在 0804C351

```asm
.text:0804C34C E8 9F C9 FF FF call ___xstat
.text:0804C351 8B 85 74 FF FF FF mov eax, [ebp+var_8C]
```

![](https://attach.52pojie.cn/forum/202507/23/170411uygz7wggcggsuu51.jpg)

[Snap17.jpg (95.64 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTM0OHw0Mjk2OGI0ZHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 17:04 上传

看红色框里的内容 1009D81，这个数又是怎么来的呢

偶然间发现，原来是这样

```text
stat /tmp/re-1.3-eval/bin
File: 'tmp/re-1.3-eval/bin' Size: 126 Blocks: 0 IO Block: 4096 directory
Device: 802h/2050d Inode: 16817537
```

![](https://attach.52pojie.cn/forum/202507/23/170839kwdhdsoqjolelzil.jpg)

[Snap18.jpg (23.24 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTM1M3xmZmFhMjRmOXwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 17:08 上传

回到 IDA View-A，按 F5 看下伪代码

```c
__xstat(3, filename, &s);
v1 = 0;
if ( s.st_ino )
{
    v2 = __ROL4__(s.st_ino, 5);
    v3 = (2 * (v2 & 0x55555555)) | ((v2 & 0xAAAAAAAA) >> 1);
    if ( (v2 & 0xFFFF0000) == 0 )
        v3 |= v2 16817537 转化为十六进制1009D81，
```

1009D81 左移 5 位（`__ROL4__(s.st_ino, 5)`），

再经过 `(2 * (v2 & 0x55555555)) | ((v2 & 0xAAAAAAAA) >> 1)` 变成 10237010，

然后 `MD5(10237010)` = 3587C9F4FAF0316EC32488E937AE4D70 最终得到 Registration key。

这个 `stat` 的 Inode 值，一旦文件夹变动，即使是从备份中恢复到原来的目录，Inode 的值也会改变，也就导致 Registration key 随之变动，自然与原来的 license 不匹配，导致 License CRC error，这么设计 Registration key，真是让人无语啊，可能是因为是 TEMP license，正式版应该不会这么处理吧。

Regstration key 部分的分析到此结束，至于 license 部分，关键算法是 Blowfish 算法，相对独立，既可以作为这款软件破解的一部分，也可以作为像我这样的算法小白初学 Blowfish 算法参考案例，详细分析起来也挺耗时耗篇幅的，有时间放在“用 EDB（Linux 下的 [www.52pojie.cn/thread-350397-1-1.html](https://www.52pojie.cn/thread-350397-1-1.html)）分析破解某加密软件 （二）”里再分享吧！
