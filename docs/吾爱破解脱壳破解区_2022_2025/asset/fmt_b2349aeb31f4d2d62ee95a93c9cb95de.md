
# [原创] PE文件整体结构介绍

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1765357
- 抓取时间: 2026-11-01T22:57:53

逆向是一个长期的过程，不积硅步，无以至千里；不积小流，无以成江海。在学逆向的前期，初学者都有一个绕不开的东西，那就是PE文件结构，这东西不难，但是内容有点多，这一篇文章面向新手，就先讲讲PE文件的整体结构。我文章中如有错误，请大佬指正！

先说明一下：
1、本文章使用的将PE文件编译为十六进制字节码格式用的是`C32Asm`，但大家实际操作还是推荐使用`WinHex`，工具均取自于爱盘。
2、这篇文章主要是讲PE文件的整体结构，还有数据目录表没有具体讲解，数据目录之后会以单独的形式一个个讲解。

PE文件是可移植的可执行的文件，什么是可执行文件？可执行文件是为进程创建所服务的，进程在运行之前，需要将该进程所需要运行的代码、该进程支持的相关数据等一个进程创建所必须的信息以某种格式存储在磁盘中，而这种格式就是可执行文件格式。简单来说就是在进程创建之前会将该进程创建所需要的信息以可执行文件格式存储在磁盘中，同样也可以说进程创建所需要的信息会在可执行文件中详细记录。有了该进程的详细记录，那么操作系统在创建进程的时候就可以根据该进程的详细记录来创建进程，也就是说可执行文件是进程创建的前身。但可执行文件和进程不能完全划等号，因为进程是动态的，是会根据你的操作变化而变化的；而可执行文件是静态的，是没有执行权、是死板的、是不会根据你的变化而变化的。而可移植的是因为微软在早期的一个战略目标，他想要达到只要是PC端就会有他们的影子，可以见到当时微软的野心是十分的大的，所以想要在任何平台上都可以运行那可移植性就十分重要了！

![](https://attach.52pojie.cn/forum/202303/27/110026nggxopaghgoor2gi.png)

image-20230130151243152.png (252.47 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwMjA0MXw1MmRiMWE2ZHwxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjA0MXw1MmRiMWE2ZHwxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes)

2023-3-27 11:00 上传

我们来看一下PE的整体结构，具体的内容下面会逐一的讲解，特别提醒：该图的PE结构是从下往上看。这里还讲一下其他的东西：附加数据；附加数据不属于PE文件结构中，但有的PE文件中会有存在，附加数据是干什么的呢？主要是为了省事，当PE文件执行某件事的时候需要用到一些相对重要的数据，但又不想搞两个文件，就将那些相对重要的数据写在了PE文件的附加部分(PE结构的后面)，当需要使用附加数据的时候就跳转到那PE结构的后面执行附件数据相关的代码，执行完成后再跳转回去。

## 一、DOS头讲解

DOS头由DOS-MZ文件头和DOS块组成，我们一个一个来讲解。我们先来看一下DOS-MZ文件头的结构：

```
typedef struct _IMAGE_DOS_HEADER {　// DOS-MZ文件头
WORD　e_magic;　　　　　　// DOS 可执行文件的标识符
WORD　e_cblp;　　　　　　// Bytes on last page of file
WORD　e_cp;　　　　　　// Pages in file
WORD　e_crlc;　　　　　　// Relocations
WORD　e_cparhdr;　　　　　// Size of header in paragraphs
WORD　e_minalloc;　　　　　// Minimum extra paragraphs needed
WORD　e_maxalloc;　　　　　// Maximum extra paragraphs needed
WORD　e_ss;　　　　　　// Initial (relative) SS value
WORD　e_sp;　　　　　　// Initial SP value
WORD　e_csum;　　　　　　// Checksum
WORD　e_ip;　　　　　　// Initial IP value
WORD　e_cs;　　　　　　// Initial (relative) CS value
WORD　e_lfarlc;　　　　　// File address of relocation table
WORD　e_ovno;　　　　　　// Overlay number
WORD　e_res[4];　　　　　// Reserved words
WORD　e_oemid;　　　　　　// OEM identifier (for e_oeminfo)
WORD　e_oeminfo;　　　　　// OEM information; e_oemid specific
WORD　e_res2[10];　　　　　// Reserved words
LONG　e_lfanew;　　　　　// PE 签名的文件偏移量
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

这上面的结构初学者可能看不懂，这里简单的解释一下结构体，结构体可以通过基础类型来构造，常见的基础类型有整形、浮点型、字符型等，我们可以通过这些基础类型作为基石来搭建一个新的结构体，这个新的结构体也可以作为一个新的类型来使用；我们再来讲讲什么是DOS文件头；DOS文件头是为了兼容DOS系统所遗留下来的产物，如果使用的不是DOS系统，其实DOS头只有两个数据是比较重要的。一个是`e_magic`，大小为`WORD`(两个字节)，作用是表示DOS 可执行文件的标识符，占用 2 字节，该位置保存着的字符是“MZ”。如果新手不是很理解也没有关系，你可以想象，当你出门在外的时候，这时你看到附近停了一辆车，你瞄了一眼看了下车标，说了句：“哦，原来是这个牌子的车啊！”，而这个“MZ”字符就类似于车标的作用，是用来标识这个文件的。另外一个是`e_lfanew`，大小为`LONG`(四个字节)，这里存有PE 签名的文件偏移量，可以指出PE文件头的位置，至于PE文件头等下就讲。DOS-MZ文件头简单的介绍完了，这里也顺带说一下DOS块，DOS块中存储的是链接器填充的信息，不重要，大小也不确定。讲实话如果不是使用DOS系统的话那么DOS块对文件没有什么影响的，可以不用管这一块大小不确定的DOS块，也可以用来填充你的代码或者数据。我们来看一下十六进制模式下的PE文件，这里使用的工具是`C32Asm_v2.0.1.0`，爱盘中可以直接下载压缩包。先把PE文件丢进`C32Asm`，DOS-MZ文件头大小为64字节，`WORD`为2字节，`DWORD`为4字节，`LONG`为4字节，也可以算算上面的`_IMAGE_DOS_HEADER`结构体大小是不是64字节。

我们看下图：

![](https://attach.52pojie.cn/forum/202303/27/110443k44qotnze77tz05q.png)

image-20230114193425079.png (26.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwMjA1MHxlNjQ5ZmE4M3wxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjA1MHxlNjQ5ZmE4M3wxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes)

2023-3-27 11:04 上传

我所划出来的部分正是DOS-MZ文件头，一行16字节，一共64字节，正好4行；我们先来看一下DOS 可执行文件的标识符，它为`_IMAGE_DOS_HEADER`结构体的第一个变量`e_magic`，长度为`WORD`，那么我们往左看去可以看到位于第一行的前两个字节4D 5A，然后再看向右边的窗口可以看到第一行数据的前两个字符为“MZ”，之前讲过`e_magic`当中存放着DOS 可执行文件的标识符，占用 2 字节，该位置存储的字符就是“MZ”。之前也说过DOS-MZ文件头当中只有两个变量比较重要，讲完了第一个变量`e_magic`，接下来讲第二个重要变量`e_lfanew`，它位于DOS-MZ文件头的最后四个字节，也就是位置0x3c，可以看上图所选定的DOS-MZ文件头的最后四位字节为00 00 00 B0，你问我为什么前面的DOS 可执行文件的标识符是正着读，而这里是倒着读，我只能告诉你这是因为内存地址在内存中存放的时候是一个反序或者说是一个倒序，貌似这些DOS 可执行文件的标识符、PE签名什么的都是正着读，或许只是正着读比较顺口，数据还是以一个倒序的方式存储的。我对此不是很清楚，评论区有没有大佬出来讲解一下呢？作为新手就先当在十六进制模式下看变量中存储的数据是倒着看的吧！我们省去前面一大串0后值为0xB0，根据这个相对地址去找到对应的位置，我们直接看0xB0位置的值为50 45 00 00，这四个字节为PE签名，用于将该文件标识为 PE 格式图像文件。总之`e_lfanew`变量中存储的这四个字节为 PE 签名的文件偏移量，此信息使Windows能够正确执行映像文件。此签名是“PE\0\0” (字母“P”和“E”，后跟两个 null 字节)。

## 二、PE文件头

PE文件头由三部分组成，分别是PE签名、COFF 文件头(又称‘标准PE头’)、可选标头(又称‘扩展PE头’)。而且PE文件头自身也是一个结构体，我们来看一下(下方展示的为32位对应的PE文件结构体)：

```
typedef struct _IMAGE_NT_HEADERS {
DWORD Signature; //该结构体中的Signature就是PE签名，标识该文件是否是PE文件。该部分占4字节，即“50 45 0000”。
IMAGE_FILE_HEADER FileHeader;
IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

我们可以看到上面这个结构体由三部分组成，第一部分PE签名在讲DOS头的时候讲过，是用于将该文件标识为 PE 格式图像文件。我们可以从上面清楚的看到PE签名类型为`DWORD`，大小为四字节，存储在`Signature`变量当中。第二个变量`FileHeader`是标准PE头，类型为结构体`IMAGE_FILE_HEADER`，听到这里新手可能会很诧异，为什么结构体也可以用来充当类型定义变量？这里你可以想一下，变量为一种容器，类型可以用来定义这个容器的结构，但基础的结构有限，当处理某些比较麻烦的数据时可能会出现基础的结构不好用了，用起来比较麻烦了，那这时候就出现了用多个基础的类型结构组成的结构体类型，这样变量存储数据时能定义的类型就变得灵活多变了。

### 2.1、标准PE头

我们来看一下标准PE头的结构：

```
typedef struct _IMAGE_FILE_HEADER {
WORD　Machine; // 标识目标计算机类型的数字。
WORD　NumberOfSections; // 区段数量。
DWORD　TimeDateStamp; // 自1970年1月1日00：00起的秒数的低 32 位 (C 运行时time_t值) ，该值指示文件创建时间。
DWORD　PointerToSymbolTable; // COFF符号表的文件偏移量，如果没有COFF符号表，则为零。映像的此值应为零，因为 COFF 调试信息已弃用。
DWORD　NumberOfSymbols; // 符号表中的条目数。此数据可用于查找紧跟在符号表后面的字符串表。映像的此值应为零，因为 COFF 调试信息已弃用。
WORD　SizeOfOptionalHeader; // 可选标头的大小，这是可执行文件所必需的。
WORD　Characteristics; // 指示文件属性的标志。
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

看完上图有的可能会让新手很懵逼，没事，我们慢慢来讲。首先上方的结构体总大小为20字节，其中只有四个变量值得注意，分别是`Machine`、`NumberOfSections`、`SizeOfOptionalHeader`、`Characteristics`。我们先来讲`Machine`，作用是标识目标计算机类型的数字，什么是标识目标计算机类型的数字呢？其实就是说明PE文件只能在哪些CPU类型或模拟指定计算机的系统上运行。先来看一下官方给出的计算机类型表：

![](https://attach.52pojie.cn/forum/202303/27/110832ze6wmz95n5wtq6wn.png)

image-20230117133654995.png (76.17 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwMjA1MnxkOWNjZmVjMnwxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjA1MnxkOWNjZmVjMnwxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes)

2023-3-27 11:08 上传

上面这张图看不懂没关系，我们找一个PE文件来讲解一下怎么用这张图来看PE文件适用的CPU类型。

![](https://attach.52pojie.cn/forum/202303/27/111046a35tkj5595stc0tt.png)

image-20230117134825932.png (23.65 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwMjA1NXxjNzYwMjFkM3wxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjA1NXxjNzYwMjFkM3wxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes)

2023-3-27 11:10 上传

看上图，所划的区域为标准PE头，你可以看到所划的区域前面为50 45 00 00，这正是PE签名，PE签名后面的二十个字节就是标准PE头。我们看前两个字节就是大小为`WORD`的`Machine`，这里记得要倒着看，即为01 4C，我们拿着014C去上面的计算机类型表去寻找解释，可以在value列看到0x14c以及对应的说明。讲完了`Machine`，我们接着讲`NumberOfSections`，大小为两字节，作用是表示PE文件的区段数量。还是看上图，`Machine`后面的两个字节为00 04，说明此PE文件的区段数量为4，我们打开PE查看工具来看看是不是四个区段：

![](https://attach.52pojie.cn/forum/202303/27/111241s7pc4ti8wdtdbjwd.png)

image-20230117141251292.png (48.72 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwMjA1OXxmNTc5ODhjM3wxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMjA1OXxmNTc5ODhjM3wxNzkzNTczODcwfDE2ODA4NnwxNzY1MzU3&nothumb=yes)

2023-3-27 11:12 上传

使用PE文件查看工具可以看到区段数量为4个，与`NumberOfSections`当中存储的区段数量相同。讲完了`NumberOfSections`，我们接着讲第三个重要变量`SizeOfOptionalHeader`，作用是表示可选标头的大小，这是可执行文件所必需的。看上图所划的区域内第17、18个字节00 E0，我们将E0换算成10进制为224，这里可能有新手要问进制是怎么转换的？其实为了省事一般都会用工具去转换进制，如果在没有工具的情况下需要进制转换，自己虽然算起来比较麻烦，但是也是没有办法的事情，这里只讲一下16进制转换10进制，不做过多的延伸，感兴趣的可以自己去了解一下。这里用的方法说起来也非常的简单，假设从末尾开始为第0位，每往前位置就加一，那么就从末尾开始用每一位的数去乘以16的n次方，这个n为这个数的位置。比如E0转化十进制就为E乘以16的1次方+0乘以16的0次方=14乘以16+0乘以1=224。补充一下知识点，十六进制是逢十六进一，十六进制由0、1、2、3、4、5、6、7、8、9、A、B、C、D、E、F组成。`SizeOfOptionalHeader`变量中的数据对于32位PE文件通常为00E0h，对于64位PE文件通常为00F0h。这里的h是用来表示这个数据为16进制。上图我们可以看到`SizeOfOptionalHeader`变量当中的数据为00 E0，扩展PE头大小为224，文件为32位PE文件。最后一个重要变量`Characteristics`，作用为指示文件属性的标志。这是什么意思呢？我们看上图所划区域内最后两个字节01 03，即为103，这里我们需要将这个值转换为二进制，十六进制怎么转换为二进制呢？十六进制数的每一位都可以转换为四位二进制数，比如3可以转换为0011，1可以转换为0001，那么103的二进制就呼之欲出了，那就是0001 0000 0011，这里就不详细讲二进制了，不清楚的可以去网上搜索二进制。将103成功转换成二进制后，我们需要来看下面这张表，它会告诉我们这个PE文件有哪些属性。
