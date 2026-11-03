
# ASP2.12 壳分析

## 入口点跳转代码

```
0040539A            | B8 10190000        | mov eax,1910                        | 原PE文件的入口点
0040539F            | 50                 | push eax                            |
004053A0            | 0385 22040000      | add eax,dword ptr ss:[ebp+422]      | 换成VA
004053A6            | 59                 | pop ecx                             | ecx:EntryPoint
004053A7            | 0BC9               | or ecx,ecx                          | ecx:EntryPoint
004053A9            | 8985 A8030000      | mov dword ptr ss:[ebp+3A8],eax      |把入口点存入[ebp+3A8]里，在查找以下这个地址就是下面push 0中0的地址
004053AF            | 61                 | popad                               | 返回壳之前的状态
004053B0            | 75 08              | jne capturescreen.4053BA            |
004053B2            | B8 01000000        | mov eax,1                           |
004053B7            | C2 0C00            | ret C                               |
004053BA            | 68 10194000        | push capturescreen.401910           |这里本来是push 0，然后通过地址0x4053A9动态修改，从而实现跳转到真正的入口点
004053BF            | C3                 | ret                                 | 跳转到真正的入口点
```

通过动态修改地址内容，跳转到真正的入口点

至此`asp2.12`壳浅层分析完毕，这里还没有去分析解密函数内部的内容。

在参考文章中，都是把解密函数用`IDA`把解密函数给抠出来，编译为`obj`使用的

## 代码分析

接下来就是代码的分析，我把我认为的困难的部分给拎出来单独说一下

首先就是

```
if(pTemp.VirtualAddress AddressOfEntryPoint
        && (pTemp.VirtualAddress + pTemp.Misc.VirtualSize) > newOptHead->AddressOfEntryPoint)
{
        packaddr = (LPBYTE)(pImageBase + pTemp.PointerToRawData + 0x57c);//0x57c是加密数据的偏移

        //修正解压函数地址
        g_esi = (DWORD)(pNewPe + pTemp.VirtualAddress + 0x70E - 0x44403E);
        //修正OEP
        DWORD dwOEP = *(DWORD*)(pNewPe + pTemp.VirtualAddress + 0x39b);
        newOptHead->AddressOfEntryPoint = dwOEP;
        //修正导入表数据目录
        DWORD dwIAT = *(DWORD*)(pNewPe + pTemp.VirtualAddress + 0x279);
        newOptHead->DataDirectory[1].VirtualAddress = dwIAT;
        //取出地址表异或值
        bXorVal = *(BYTE*)(pNewPe + pTemp.VirtualAddress + 0x148);
        //地址表是否参与异或运算
        isXor = *(BYTE*)(pNewPe + pTemp.VirtualAddress + 0x145) > 0 ? 0 : 1;
}
```

### packaddr 解析

首先看`packaddr`，这个指针是加密区段的内存偏移，这里是通过文件偏移来读取区段的偏移地址。去下图，`0x1A00` + `0x57C` = `0x1F7C`，`0x1A00`是`.aspack`区段的文件偏移地址。

![](https://attach.52pojie.cn/forum/202505/24/173701bdn5osnz58l25npm.png)

image-20250522215641433.png (14.52 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA1NHxhMzUzZWVjNnwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA1NHxhMzUzZWVjNnwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:37 上传

那`0x57C`又是怎么来的呢？这里先回到调试器中，看到`0x40509E`地址，这里就把各区段的`RVA`和`Size`存在`esi`寄存器中，把`[ebp + 569]`地址转到内存中，如下图的内存1这里的偏移的地址就是`0x57C`

![](https://attach.52pojie.cn/forum/202505/24/173727r9pvreolutppicn0.png)

image-20250522220115611.png (125.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA1NXxlNjcyZWQyM3wxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA1NXxlNjcyZWQyM3wxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:37 上传

### g_esi 解析

接下来就是`g_esi`，这个是解密函数里需要用到的，在调试器中如下图

![](https://attach.52pojie.cn/forum/202505/24/173747k8ppmyge8p6b5p5u.png)

image-20250524001504251.png (10.97 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA1NnwyMTRhM2RkZXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA1NnwyMTRhM2RkZXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:37 上传

![](https://attach.52pojie.cn/forum/202505/24/173803wj0m3wmttggqklgm.png)

image-20250522223554308.png (97.02 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=Mjc3OTA1N3w4Mjg3ZWU0N3wxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA1N3w4Mjg3ZWU0N3wxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:38 上传

在解密函数中都会有调用`0x405C96`，而`0x405C96`这个函数每次都会返回给`esi`这个寄存器固定的值，这里的常量与代码中的不一样，但经过调试，得出的结果是一样的，我这里是`0x405C9B` - `0x4445CB` = `0xFFFC16D0`，得到`exi`后进行下一个代码，从`0x40570E`开始以`eax`为偏移获取后续的数据，由于基址为`0x40000`，所以偏移量为`0x570E`，因此代码中是求`esi`，通过恒等式移项就可以，即 `esi` + `0x44403E` = 基址 + `0x570E` ==> `esi` = 基址 + `0x570E` - `0x44403E`

![](https://attach.52pojie.cn/forum/202505/24/173827v8mc8v033fe8v0v8.png)

image-20250524002346214.png (36.8 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA1OHw1MGRkZWQ5ZXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA1OHw1MGRkZWQ5ZXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:38 上传

### OEP 解析

接下来是`OEP`，只需要注意代码中的数据，如图

![](https://attach.52pojie.cn/forum/202505/24/173843w0wpnjsz96adio0s.png)

image-20250524004139212.png (107.45 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3OTA1OXw3ODk4NzY4NXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA1OXw3ODk4NzY4NXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:38 上传

标灰的地方就是`OEP`的`RVA`了，所以直接读取该数据所在的`RVA`的地址了

下面的导入表的修正也是同样的道理，直接贴图就不细说了

![](https://attach.52pojie.cn/forum/202505/24/173907s7pcol27obo95p52.png)

image-20250524004427530.png (142.13 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA2MHwxYzdjZTBiNnwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA2MHwxYzdjZTBiNnwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:39 上传

### jmp/CALL 还原条件

下面是`jmp`/`CALL`指令还原的条件判断，但是文章说

`jmp`/`CALL`后面的第一个字节为还原条件，针对不同的程序，判断条件不同

因此可能有`dll`之类不同的分类，总之在当前情况下就是还原条件，后面会详说

### 跳转指令修复

接下来就是还原跳转指令了，如图中的`0x405133`地址的代码，解密函数中的参数就有缓存地址，因此执行解密函数后，缓存地址就是需要还原的数据，代码中`fixCode`函数中的 `char *addrbuff` 参数好像没有用处。下面的代码就是修复的操作，根据汇编语言来写成高级语言

![](https://attach.52pojie.cn/forum/202505/24/173924oojyncl5chslth49.png)

image-20250524111909689.png (89.11 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA2MXw0MTM3Yzk4YXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA2MXw0MTM3Yzk4YXwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:39 上传

然后我认为代码中比较难理解的部分我就分析完了，接下来是提取`ida`中代码编译成`obj`文件的操作说明

![](https://attach.52pojie.cn/forum/202505/24/173942e7cmaa7cjxoaxj99.png)

image-20250524164532888.png (81.82 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3OTA2MnxmNGJkNmE5YnwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA2MnxmNGJkNmE5YnwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx&nothumb=yes)

2025-5-24 17:39 上传

首先载入`IDA`找到解密函数的地址，这里我把这个进行了函数处理，就是把这个当成函数来看待，从而生成函数相关的栈帧、参数、局部变量之类的。

然后把这函数全都复制出来，粘贴到`asm`文件，再用`masm`执行编译为`obj`文件的命令，由于我这边没有`masm`环境，想尝试的可以自行尝试和搜索资料。这里不仅仅只复制解密函数，还需要复制以前的代码，也就是`call`里面的代码。

至此`asp`分析完毕。

## 附件

![](https://static.52pojie.cn/static/image/filetype/rar.gif)

[href="forum.php?mod=attachment&aid=Mjc3OTA2NHw1OGZjNmY5ZHwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA2NHw1OGZjNmY5ZHwxNzkzNTcwOTQ4fDE2ODA4NnwyMDMzODYx)

(16.28 KB, 下载次数: 14)

2025-5-24 17:49 上传

点击文件名下载附件
下载积分: 吾爱币 -1 CB

## 参考资料

[//bbs.kanxue.com/thread-91073.htm" target="_blank" rel="noopener noreferrer nofollow"](https://bbs.kanxue.com/thread-91073.htm:rel="nofollow noopener" href="https)

[[//bbs.kanxue.com/thread-91070.htm" target="_blank" rel="noopener noreferrer nofollow"](https://bbs.kanxue.com/thread-91070.htm:rel="nofollow noopener" href="https)
