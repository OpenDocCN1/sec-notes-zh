
# [CTF] 【2022春节红包】全部题目WriteUp

原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1588907

抓取时间: 2026-11-01T23:36:03

## Windows初级题

### 前期准备

先运行一下随便输入一点东西，看看错误提示

![](https://attach.52pojie.cn/forum/202202/16/201456ljttnqo0po8xuouu.png)

`2_WrongInput.png` (25.59 KB, 下载次数: 248)

[href="forum.php?mod=attachment&aid=MjQ5NTM0Mnw4NGQ1MDJhZHwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0Mnw4NGQ1MDJhZHwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:14 上传

输出是 Error, please try again

用 DetectItEasy 查了一下，是32位程序无壳的，可以直接开始逆。

![](https://attach.52pojie.cn/forum/202202/16/201518sx4d0zt1cbedzx04.png)

`2_DIE.png` (97.22 KB, 下载次数: 256)

[href="forum.php?mod=attachment&aid=MjQ5NTM0M3xmODQwMWRkOXwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0M3xmODQwMWRkOXwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:15 上传

### 静态分析

拖到IDA里分析一下，主要逻辑都在`_main`函数里

![](https://attach.52pojie.cn/forum/202202/16/201619nf6psu6sl4vk4f5v.png)

`2_IDA_1.png` (147.62 KB, 下载次数: 241)

[href="forum.php?mod=attachment&aid=MjQ5NTM0NXwwNDY3ZGI0NnwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0NXwwNDY3ZGI0NnwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:16 上传

从能看到有个循环，循环条件还很怪，地址小于字符串HappyNewYear的地址，跟过去看一下

![](https://attach.52pojie.cn/forum/202202/16/201648vtsqtleiq97f9q46.png)

`2_IDA_2.png` (186.91 KB, 下载次数: 244)

[href="forum.php?mod=attachment&aid=MjQ5NTM0NnwxNTE4NGM2M3wxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0NnwxNTE4NGM2M3wxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:16 上传

给了一个提示，`flag`的长度是00010111，转成10进制就是23

在前面也能看到`if`语句判断了`v21 != 23`

运行程序输入一串长度为23的字符，可以看到输出文字有变化，更加确定了这一点

再来看一眼这个奇怪的循环，`v3`是一个指针，最开始指向`unk_4140B0`

![](https://attach.52pojie.cn/forum/202202/16/201720m1msasm1rxv1jitj.png)

`2_IDA_3.png` (163.11 KB, 下载次数: 261)

[href="forum.php?mod=attachment&aid=MjQ5NTM0N3w1ODVjNWZiYnwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0N3w1ODVjNWZiYnwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:17 上传

跟过去发现，这个东西在字符串HappyNewYear的前面，而且四个字节一组的样子，数数总共是23个，可以猜测这边是经过加密的`flag`

由于解密函数不那么简单，也就不再继续静态分析了，上调试器跟着看看

### 动态分析

我用的是 x96dbg (x32dbg)，用 Ollydbg 或者 IDA 直接调试也一样吧，没啥区别

用x32dbg要注意一点，它停下来的EntryPoint并不是`_main`函数，而是`_start`函数

![](https://attach.52pojie.cn/forum/202202/16/201747l4ts8dt66459szj5.png)

`2_dbg_1.png` (349.82 KB, 下载次数: 251)

[href="forum.php?mod=attachment&aid=MjQ5NTM0OHw1ZmZiYWIzM3wxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0OHw1ZmZiYWIzM3wxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:17 上传

要在这个地方下个断点，再F7单步跟过去

到了入口点，看一下`0x004012F6`到`0x00401317`这一段便是那个循环了，直接在之后下个断点让他跑完

![](https://attach.52pojie.cn/forum/202202/16/201809wg4s29gigio1ca9k.png)

`2_dbg_2.png` (350.56 KB, 下载次数: 254)

[href="forum.php?mod=attachment&aid=MjQ5NTM0OXxiNGVhNjFiOXwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM0OXxiNGVhNjFiOXwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:18 上传

果然就出来一个字符串2022HappyNewYear52PoJie，长度刚好是23

用程序验证一下

![](https://attach.52pojie.cn/forum/202202/16/201828b8pbm364duqbx6qu.png)

`2_Success.png` (26.96 KB, 下载次数: 257)

[href="forum.php?mod=attachment&aid=MjQ5NTM1MHw2OWJmNjA0MXwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM1MHw2OWJmNjA0MXwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:18 上传

成功~这就完事了

## Windows中级题

### 剧透一下

这道题和密码学有关哦~

仿射密码，还有计算模反元素

在CTF中做了不少密码题，在这里看到我好激动啊啊啊啊！

### 前期准备

还是运行一下看看，这个是要输入`UID`和`flag`的，所以每个人`flag`还会不一样哦，我就不截图了

一看这个文件大小就不太对劲，根初级题比有点小了，查个壳看看

![](https://attach.52pojie.cn/forum/202202/16/201854pl3990i3cll93rfi.png)

`3_DIE.png` (97.27 KB, 下载次数: 261)

[href="forum.php?mod=attachment&aid=MjQ5NTM1MXxmYWM3MjRlYnwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM1MXxmYWM3MjRlYnwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:18 上传

果不其然，是被UPX压缩过了，这个我不知道能不能用工具直接脱

看着是UPX [modified]大概不行吧，我就用ESP定律了

### ESP定律脱壳

入口点经典`pushad`单步一下看到只有`ESP`寄存器是红色的

![](https://attach.52pojie.cn/forum/202202/16/201932kkk2294om99kj369.png)

`3_ESP_1.png` (558.8 KB, 下载次数: 248)

[href="forum.php?mod=attachment&aid=MjQ5NTM1MnxhODEwZjZiMHwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM1MnxhODEwZjZiMHwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:19 上传

对`ESP`寄存器右键-在内存窗口中转到，接着在内存窗口中对这个地址右键-断点-硬件，访问几字节都无所谓

接着F9运行，等着`popad`触发硬件断点

![](https://attach.52pojie.cn/forum/202202/16/201957oqw1x5qiy0f11lqz.png)

`3_ESP_2.png` (506.09 KB, 下载次数: 260)

[href="forum.php?mod=attachment&aid=MjQ5NTM1M3w4ZjY3Yjg0M3wxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM1M3w4ZjY3Yjg0M3wxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:19 上传

把硬件断点取消，然后在下面的`jmp`大跳转上下个断点

断下来之后F7单步，来到OEP

接下来就是要用Scylla插件了，从顶上的插件菜单打开Scylla

![](https://attach.52pojie.cn/forum/202202/16/203654ampjv8jtqsgtvm8v.png)

`3_ESP_3_.png` (570.44 KB, 下载次数: 247)

[href="forum.php?mod=attachment&aid=MjQ5NTM2OXxmYTdjZTM4NHwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTM2OXxmYTdjZTM4NHwxNzkzNTc2MTYwfDE2ODA4NnwxNTg4OTA3&nothumb=yes)

2022-2-16 20:36 上传

不解释了，按照图里说的做吧

最后会获得一个`xxxxx_dump_SCY.exe`这个就是脱壳之后的程序了

### 静态分析

#### 关于C++的`string`

这道题使用C++写的

不得不说，我好讨厌C++，源代码看着都费劲，逆向分析更恶心

C++的源代码中经常会用`std::string`这个类型，但是逆向的时候好像IDA分析不出来

我没找到`std::string`的正确结构，在这道题里，我根据代码反推了一个它的结构，也不一定对[捂脸]，如果有大佬知道请一定告诉我
