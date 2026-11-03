
# [原创] PE 文件结构剖析：手工压缩与注入实战

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990080
- 抓取时间: 2026-11-01T22:18:19

本文由笔者首发于先知社区的技术文章板块：[https://xz.aliyun.com/t/16622](https://xz.aliyun.com/t/16622)

## PE文件结构分析

PE 全称是 Portable Executable，即可移植的可执行文件，是 Windows 操作系统下可执行文件的总称，是用于存储可执行文件 (`exe`, `scr`)、动态链接库 (`dll`, `oxc`, `cpl`) 和驱动程序 (`sys`, `vxd`) 的标准文件格式。

PE 文件结构复杂而丰富，它包含了可执行文件的所有必要信息，以便操作系统正确加载和执行程序。了解 PE 文件结构对于逆向工程、软件分析和安全研究非常重要，因为它提供了深入了解可执行文件内部工作原理的窗口，下面是对其文件结构的详细分析

### DOS头 (DOS Header + DOS Stub)

PE 文件的开头通常包含一个 DOS 头，用于向后兼容早期的 MS-DOS 操作系统，使得 DOS 识别出这是有效的执行体，然后运行紧随之后的是 DOS Stub

DOS Header 由一个 0x40 大小的 `IMAGE_DOS_HEADER` 结构体组成

```
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE 文件头结构体
    WORD   e_magic;                     // 标识符，用于确认这是MZ格式的文件，值为0x5A4D
    WORD   e_cblp;                      // 文件中最后一个扇区的字节数
    WORD   e_cp;                        // 文件中的扇区总数
    WORD   e_crlc;                      // 重定位表中的条目数
    WORD   e_cparhdr;                   // 文件头的大小，以16字节为单位
    WORD   e_minalloc;                  // 程序加载时所需的最小额外内存段落数
    WORD   e_maxalloc;                  // 程序加载时所需的最大额外内存段落数
    WORD   e_ss;                        // 初始堆栈段选择子（段地址）
    WORD   e_sp;                        // 初始堆栈指针值
    WORD   e_csum;                      // 校验和，用于检验文件的完整性
    WORD   e_ip;                        // 初始指令指针（IP值）
    WORD   e_cs;                        // 初始代码段选择子（段地址）
    WORD   e_lfarlc;                    // 文件中重定位表的偏移量
    WORD   e_ovno;                      // 覆盖号，用于实现覆盖功能
    WORD   e_res[4];                    // 保留字段，供未来使用
    WORD   e_oemid;                     // OEM标识符，用于特定于OEM的扩展
    WORD   e_oeminfo;                   // OEM信息，供OEM使用
    WORD   e_res2[10];                  // 保留字段，供未来扩展使用
    LONG   e_lfanew;                    // 指向新EXE（PE）头的偏移量，从文件开始处计算
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

其中主要关注 `e_magic` 和 `e_lfanew` 这俩成员变量即可。`e_magic` 位于文件首，其值对应的 ASCII 为 `MZ`，标识该文件为可执行文件；`e_lfanew` 的值表示 PE 头的偏移地址

![](https://xzfile.aliyuncs.com/media/upload/picture/20241208020053-3382188a-b4c5-1.png)

DOS Stub 在多数情况下由汇编器/编译器自动生成，由代码和数据混合而成，大小不固定，在不支持 PE 文件格式的操作系统中，它将简单显示一个错误提示。不需要过多关注，在 Windows OS 下不会运行这部分代码，但在DOS环境中可以运行。

### NT头

NT 头是 PE 文件的核心部分，也是 PE 头的一部分，包含了有关可执行文件的重要信息。PE 头的开始位置由 DOS 头中的 `e_lfanew` 字段指定。在 32 位下这个结构体由一个 0xf8 大小的 `IMAGE_NT_HEADERS` 结构体组成，该结构中包含了 PE 文件被载入内存时需要用到的重要域，该结构体的大小为0xf8字节，如下

```
typedef struct _IMAGE_NT_HEADERS {

    DWORD Signature;  // PE签名，0x4字节

    IMAGE_FILE_HEADER FileHeader;  // PE头，0x14字节

    IMAGE_OPTIONAL_HEADER32 OptionalHeader;  // PE可选头

} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

其中的 PE 头结构体 `IMAGE_FILE_HEADER` 成员如下，它们各自的作用详见注释

```
typedef struct _IMAGE_FILE_HEADER {
    WORD Machine;  // 目标机器的类型码，如x86或ARM
    WORD NumberOfSections;  // 文件中节(section)的数量
    DWORD TimeDateStamp;  // 文件创建或最后修改的时间戳
    DWORD PointerToSymbolTable;  // 指向文件中符号表的偏移地址
    DWORD NumberOfSymbols;  // 符号表中符号条目的数量
    WORD SizeOfOptionalHeader;  // 可选头的大小，用于存储扩展的文件信息
    WORD Characteristics;  // 文件特性标志，指DLL、应用程序、可执行文件等
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

其中的 PE 可选头结构体 `IMAGE_OPTIONAL_HEADER32` 成员如下，它们各自的作用详见注释

```
#define IMAGE_NUMBEROF_DIRECTORY_ENTRIES 16  // 数据目录项数，固定为16

typedef struct _IMAGE_OPTIONAL_HEADER {

    // 标准域

    WORD Magic;  // 可选头类型，0x10表示32位，0x20表示64位

    BYTE MajorLinkerVersion;  // 主链接器的版本号，以字节为单位

    BYTE MinorLinkerVersion;  // 副链接器的版本号，以字节为单位

    DWORD SizeOfCode;  // 代码段大小，以字节为单位

    DWORD SizeOfInitializedData;  // 初始化数据段大小，以字节为单位

    DWORD SizeOfUninitializedData;  // 未初始化数据段大小，以字节为单位

    DWORD AddressOfEntryPoint;  // 程序入口点地址，相对于ImageBase

    DWORD BaseOfCode;  // 代码段起始基址RVA

    DWORD BaseOfData;  // 数据段起始基址RVA

    // NT附加域

    DWORD ImageBase;  // 镜像基址，即加载到内存的起始地址

    DWORD SectionAlignment;  // 节在内存中的对齐大小，以字节为单位

    DWORD FileAlignment;  // 节在文件中的对齐大小，以字节为单位

    WORD MajorOperatingSystemVersion;  // 主操作系统版本号

    WORD MinorOperatingSystemVersion;  // 副操作系统版本号

    WORD MajorImageVersion;  // 主镜像版本号

    WORD MinorImageVersion;  // 副镜像版本号

    WORD MajorSubsystemVersion;  // 主子系统版本号

    WORD MinorSubsystemVersion;  // 副子系统版本号

    DWORD Win32VersionValue;  // Win32版本值，通常为0

    DWORD SizeOfImage;  // 镜像在内存中的大小，以字节为单位

    DWORD SizeOfHeaders;  // PE头物理大小，以字节为单位

    DWORD CheckSum;  // 校验和，用于验证镜像的完整性

    WORD Subsystem;   // 子系统类型

    WORD DllCharacteristics;  // DLL特性标志，指示文件是DLL、应用程序等

    DWORD SizeOfStackReserve;  // 运行时为每个线程栈保留的内存大小

    DWORD SizeOfStackCommit;  // 运行时每个线程栈初始占用的内存大小

    DWORD SizeOfHeapReserve;  // 运行时为进程堆保留的内存大小

    DWORD SizeOfHeapCommit;  // 运行时进程堆初始占用的内存大小

    DWORD LoaderFlags;  // 载入器标志，通常为0

    DWORD NumberOfRvaAndSizes;  // 数据目录的项数，固定为IMAGE_NUMBEROF_DIRECTORY_ENTRIES的值

    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];  // 数据目录数组

} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

### 节表区

节表描述了 PE 文件中各个节的布局和属性，其位于 NT 头之后，也是 PE 头的最后一个部分

节区表记录了 PE 文件中所有节区的相关属性，节区表由一系列的 `IMAGE_SECTION_HEADER` 结构排列而成，每个结构用来描述一个节，结构的排列顺序和它们描述的节在文件中的排列顺序是一致的。全部有效结构的最后以一个空的 `IMAGE_SECTION_HEADER` 结构作为结束，所以节表中 `IMAGE_SECTION_HEADER` 结构数量等于节的数量加一。`IMAGE_SECTION_HEADER` 结构体大小为 0x28 字节

### PE 文件其余特定区域

再继续往下便是真真正正的 `text` 节，`data` 节，`rsrc` 节。以及数据目录表、导入表、导出表、资源表、重定位表、甚至还有其他自定义部分，如 TLS 表（线程局部存储表）、加载配置表 (Load Configuration Table) 等，这些部分包含了各种附加信息和配置…

![](https://xzfile.aliyuncs.com/media/upload/picture/20241208020106-3b62e23c-b4c5-1.png)

然后补充一下 `.relocation` 节，该节的前八字节记录了节的 RVA 和需要进行重定位或修改的条目数量，每一项都记录了哪些 RVA 地址下的硬编码寻址在加载进内存时是需要进行重定位的

### 导入表&导出表

最后着重讲解一下 PE 文件的导入表和导出表，在 Windows 程序逆向中，我们能从这两个表中获取到许多非常重要信息

导入表：每个 `exe` 或者 `dll` 一般都会有它的导入表，记录了其自身会使用到的其他模块导出的函数。即记录调用了哪些模块 (`dll`)，以及调用了它里面的哪些函数

导入表的意义是确定 PE 文件依赖哪个模块的哪个函数，以及确定模块加载进内存后具体函数的地址。一个导入表的大小是 0x14 字节，结构体如下
