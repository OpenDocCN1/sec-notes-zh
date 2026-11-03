
## 语法解混淆脚本

```
#include 

static main()
{
    auto seg, current_ea, ea;
 
    // 遍历所有段
    for (seg = get_first_seg(); seg != BADADDR; seg = get_next_seg(seg))
    {
        auto seg_name = get_segm_name(seg);
 
        // 检查段名是否符合要求
        if (seg_name != ".text" && seg_name != "UPX0")
        {
            //Message("跳过段: %s (0x%X)\n", seg_name, seg);
            continue;
        }
 
        Message("正在处理段: %s (0x%X)\n", seg_name, seg);
 
        // 获取段的起始和结束地址
        auto start_ea = seg;
        auto end_ea = get_segm_end(seg);
 
        current_ea = start_ea;
        auto pattern1 = "0F 84 01 00 00 00";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern1);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern1 at 0x%X\n", ea);
            patch_byte(ea + 6, 0x90);   
            current_ea = ea + 1;
        }
 
        current_ea = start_ea;
        auto pattern2 = "0F 84 02 00 00 00";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern2);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern2 at 0x%X\n", ea);
            patch_word(ea + 6, 0x9090);             
            current_ea = ea + 1;
        }
 
        current_ea = start_ea;
        auto pattern3 = "0F 84 08 00 00 00";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern3);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern3 at 0x%X\n", ea);
            patch_qword(ea + 6, 0x9090909090909090);
            current_ea = ea + 1;
        }
 
        current_ea = start_ea;
        auto pattern4 = "0F 8D 06 00 00 00";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern4);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern4 at 0x%X\n", ea);
            patch_word(ea + 6, 0x0B0F);             
            patch_dword(ea + 8, 0x90909090);
            current_ea = ea + 1;
        }
 
        current_ea = start_ea;
        auto pattern5 = "E9 06 00 00 00 00";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern5);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern5 at 0x%X\n", ea);
            patch_word(ea + 5, 0x0B0F);             
            patch_dword(ea + 7, 0x90909090);
            current_ea = ea + 1;
        }
 
        current_ea = start_ea;
        auto pattern6 = "EB E9";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern6);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern6 at 0x%X\n", ea);
            patch_word(ea, 0x0B0F);                   
            patch_dword(ea + 2, 0x90909090);
            current_ea = ea + 1;
        }
 
        current_ea = start_ea;
        auto pattern7 = "3C E9";
        while (1)
        {
            ea = find_binary(current_ea, SEARCH_DOWN, pattern7);
            if (ea > end_ea || ea == BADADDR)
                break;
 
            Message("Found pattern7 at 0x%X\n", ea);
            patch_word(ea + 2, 0x0B0F);              // ud2
            patch_word(ea + 4, 0x9090);              // nop
            current_ea = ea + 1;
        }
 
    }
 
    patch_word(0x413824, 0x0B0F);
    patch_word(0x41669E, 0x0B0F);
    patch_word(0x41878D, 0x0B0F);
    patch_qword(0x41878F, 0x9090909090909090);
    patch_byte(0x418797, 0x90);   
 
    Message("Finished.\n");
}
```

运行脚本后，将`patch`后文件保存为`re-new1.exe`，`IDA`打开此文件后，发现大部分函数已经被识别出来：

![](https://attach.52pojie.cn/forum/202502/21/144806tr86gslz0jv1n68a.png)

10.png (546.18 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUwOHwzYjBjYTU2NHwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUwOHwzYjBjYTU2NHwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

手工把剩下没有自动识别成函数的位置按`P`键生成函数，至此我们完全实现了语法解混淆：

![](https://attach.52pojie.cn/forum/202502/21/144812tudugepsssd0df0k.png)

11.png (1.09 MB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUxMHwzZGEwMzk3MXwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUxMHwzZGEwMzk3MXwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

## 三、 去除语义混淆

从`start`函数中，不难发现`sub_4098F0`为`main`函数，`F5`反编译代码如下：

![](https://attach.52pojie.cn/forum/202502/21/144816fb8aoz2px8r9y86k.png)

12.png (1.06 MB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUxMXw2MjdmODcxMHwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUxMXw2MjdmODcxMHwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

可以发现程序还是非常难以阅读，不过此时笔者发现下面区域的字节都是固定常数`0-9`：

![](https://attach.52pojie.cn/forum/202502/21/144817jtocl1icigvg2bgz.png)

13.png (739.67 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUxMnxkN2U5Y2E3Y3wxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUxMnxkN2U5Y2E3Y3wxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

交叉引用发现它们只被读而没被写，因此可以分别重命名为`byte0-9`：

![](https://attach.52pojie.cn/forum/202502/21/144819ca1peeppa1nwsep6.png)

14.png (569.06 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUxNHw1ZjY1NGY4Y3wxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUxNHw1ZjY1NGY4Y3wxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

重新分析反编译代码，发现它们被用来实现语法混淆，例如`155`行的条件分支永远为真，`v147`永远为`0`等。因此我们可以考虑把对这些常量的引用的指令变成对常数引用的指令，利用`IDA`的优化来去除这些永真永假跳转。以`byte0`为例，其指令形式为`movsx ecx/eax/edx, cs:byte0`，占7个字节，我们可以将其改为`mov ecx/eax/edx, 0`（占5个字节），再填充2字节的`nop`指令，对`byte0-byte9`使用`idc`脚本去除混淆，代码如下：[C] 纯文本查看 复制代码

```
#include 

static main()

{

    auto seg, current_ea, mnemonic, op1, op2;

    // 遍历所有段
    for (seg = get_first_seg(); seg != BADADDR; seg = get_next_seg(seg))
    {

        auto seg_name = get_segm_name(seg);

        // 检查段名是否符合要求
        if (seg_name != ".text" && seg_name != "UPX0")
        {

            //Message("跳过段: %s (0x%X)\n", seg_name, seg);
            continue;
        }

        Message("正在处理段: %s (0x%X)\n", seg_name, seg);

        // 获取段的起始和结束地址
        auto start_ea = seg;
        auto end_ea = get_segm_end(seg);

        // 遍历段中的每一条指令
        current_ea = start_ea;
        while (current_ea 
```

![](https://attach.52pojie.cn/forum/202502/21/144823mnngln8860zxx0mz.png)

15.png (477 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUxNXwzNWRlODM0MnwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUxNXwzNWRlODM0MnwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

此时，笔者发现三个使用频率最高的函数也使用了语义混淆，例如`sub_41307D`（重命名为`return_arg1`）返回第一个参数的值：

![](https://attach.52pojie.cn/forum/202502/21/144826hcpmanad1ttzp5bt.png)

16.png (100.5 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1NjUxNnxhMDczY2ZlZHwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NjUxNnxhMDczY2ZlZHwxNzkzNTcxMjcyfDE2ODA4NnwyMDA4MzA5&nothumb=yes)

2025-2-21 14:48 上传

类似地，可以发现`sub_413162`（重命名为`return_num1`）返回固定值`1`，`sub_4131D6`（重命名为`return_arg2`）返回第二个参数的值，我们可以继续编写`idc`脚本，把对这`3`个函数的调用（`call`指令）换成对应的对`rax/eax`操作的指令，代码如下：[C] 纯文本查看 复制代码
