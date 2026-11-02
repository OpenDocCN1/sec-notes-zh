
# [分享] PE结构代码

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1644215
- 抓取时间: 2026-11-01T23:22:55

2015年注册的吾爱，这8年一直在做渗透，因为特殊的工作性质，没有在吾爱发过技术贴，对论坛也没有什么贡献，感谢H大提供的平台，我在这里找到了很多技术资料。

感觉渗透做累了～没有一点提高这8年，最近打算转逆向，望各位师兄们能够多多指导我这个菜鸡

最近在打基础，学习了一段时间PE，在这里分享一下我的笔记：

## PE结构图

```
1、DOS头（64byte）
WORD   e_magic; *          //5A4D   //重要，DOS头的开始MZ（MZ是一个人名，DOS的开发）
WORD   e_cblp;             //0090
WORD   e_cp;               //0003
WORD   e_crlc;             //0000
WORD   e_cparhdr;          //0004
WORD   e_minalloc;         //0000
WORD   e_maxalloc;         //FFFF
WORD   e_ss;               //0000
WORD   e_sp;               //00B8
WORD   e_csum;             //0000
WORD   e_ip;              //0000
WORD   e_cs;              //0000
WORD   e_lfarlc;          //0040
WORD   e_ovno;            //0000
WORD   e_res[4];         //00 00 00 00 00 00 00 00
WORD   e_oemid;           //0000
WORD   e_oeminfo;         //0000
WORD   e_res2[10];        //20 0
DWORD   e_lfanew;  *       //00 00 00 D8    //重要，指向PE头开始的偏移位置

2、PE头->>(_IMAGE_FILE_HEADER FileHeader)(_IMAGE_OPTIONAL_HEADER OptionalHeader):
DWORD   Signature; //00 00 45 50

3、_IMAGE_FILE_HEADER FileHeader:   //共20个byte
WORD    Machine;                 //014C              *      //程序运行的CPU型号，1、0x0能在任何处理器运行 2、0x14c 可以在386及后续处理器运行
WORD    NumberOfSections;        //0003              *      //文件中存在的节（除了PE头（可选头和标准头就是NT）以外的节）的总数，如果要新增一个节那么就需要修改这个值
DWORD   TimeDateStamp;          //6264 D913          *
DWORD   PointerToSymbolTable;   //0000 0000
DWORD   NumberOfSymbols;        //0000 0000
WORD    SizeOfOptionalHeader;    //00E0              *        //option_header pe头的大小，如果是32位的话默认是E0h，64位默认的是F0h，大小可以自定义
WORD    Characteristics;         //010F              *        //每个位有不同的含义，可执行文件值为10F，即0 1 2 3 8位置1

4、_IMAGE_OPTIONAL_HEADER OptionalHeader:
WORD    Magic;                 //010B                *         //说明文件类型，如果是32位则是10B，如果是64位则是20B
BYTE    MajorLinkerVersion;     //06
BYTE    MinorLinkerVersion;     //00
DWORD   SizeOfCode;             //0000 4000                    //所有代码节的和，必须是FileAlignment的整数倍 编译器填的 没用（不影响PE文件运行）
DWORD   SizeOfInitializedData;   //0000 3000          *
DWORD   SizeOfUninitializedData;  //0000 0000         *
DWORD   AddressOfEntryPoint;        //0000 1041       *       //重要，程序的入口点（OEP）   比如OD加载程序的时候，停的入口点即是Imagebase+AddressOfEntryPoint 的地址
DWORD   BaseOfCode;                //0000 1000         *          //代码段的基址，没有用
DWORD   BaseOfData;                //0000 0050         *        //数据开始的基址，没有用
DWORD   ImageBase;                 //0040 0000          *      //重要，内存镜像地址
DWORD   SectionAlignment;          //0000 0010          *           //内存对齐
DWORD   FileAlignment;             //0000 0010          *           //文件对齐
WORD    MajorOperatingSystemVersion;       //0040
WORD    MinorOperatingSystemVersion;       //0000
WORD    MajorImageVersion;                  //0000
WORD    MinorImageVersion;                //0000
WORD    MajorSubsystemVersion;            //0004
WORD    MinorSubsystemVersion;            //0000
DWORD   Win32VersionValue;                //0000 0000
DWORD   SizeOfImage;                      //0001 A000       *      //内存中映像（拉伸后（peloader））的大小，可以比实际的值大，但必须是内存对齐的整数倍
DWORD   SizeOfHeaders;                     //0000 1000       *     //所有头+节表对齐后的大小，否则加载会出错
DWORD   CheckSum;                          //0000 0000       *      //效验和（把一个PE文件按照2个字节循环加，加完的值存到checksum里面），判断文件是否被修改
WORD    Subsystem;                          //0003
WORD    DllCharacteristics;                //0000
DWORD   SizeOfStackReserve;                  //0010 0000     *
DWORD   SizeOfStackCommit;                   //0010 0000     *
DWORD   SizeOfHeapReserve;                   //0010 0000     *
DWORD   SizeOfHeapCommit;                    //0010 0000     *
DWORD   LoaderFlags;                         //0000 0000
DWORD   NumberOfRvaAndSizes;                  //0000 0010     //目录项数据，编译器用的
```

### 2、节表

```
BYTE Name[IMAGE_SIZEOF_SHORT_NAME];      // 8个字节的节区名称
union {
DWORD PhysicalAddress;DWORD VirtualSize;}Misc; //节区真实共有多少个字节,VirtualSize 节真正长度即节结束的偏移
DWORD VirtualAddress;            // 从imagebase开始，离imagebase有多远。节的偏移地址，真实节地址为Imagebase+VirtualAddress
DWORD SizeOfRawData;            // 节在文件中对齐后的尺寸（当前节在文件中对齐的大小是132h，如果文件对齐是200，那么就是200h。如果是1000h，那么对齐后的尺寸就是1000h。1000的整数倍）
DWORD PointerToRawData;        // 在文件中的偏移量(在文件中的起始位置)
DWORD PointerToRelocations;
DWORD PointerToLinenumbers;
WORD NumberOfRelocations;
WORD NumberOfLinenumbers;
DWORD Characteristics;       // 节属性如可读，可写，可执行等
```

以上是PE结构参考，下面是用c实现的代码，按照海哥讲的基础分为几个功能模块：读取PE结构，在代码空白区添加节，导入表读取，`filebuffer to imagebuffer`以及`imagebuffer to newbuffer`（部分实现参考网上的师哥，其中导出表部分还未完善。`64.h`是封装的函数体部分。
