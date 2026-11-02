
`0x20` 大小的 `fastbins` 链上形成了 double free。

再次 `malloc` 两次后，设断点在 `0x40089F`。

![958172_B4MCXD5ETFVSG4E.png](https://attach.52pojie.cn/forum/202310/14/114253fz5qzblvesflflyl.png)

`958172_B4MCXD5ETFVSG4E.png`（45.94 KB，2023-10-14 11:42 上传，下载次数：1）

再次查看 `bins`，因为申请两次后，`fastbins` 中剩下 `f1(0x60300)`，而 `0x60300` 指向 `0x603020` 没有改变，`0x603020` 指向 `0x60300` 也没变，并且 `fastbins` 中的 chunk 标记为 `prev_inuse` 一直为 1，所以 `fastbins` 中依然保留这个 ABA 结构。

![958172_7QJAPUB4DDD2PHB.png](958172_7QJAPUB4DDD2PHB.png)

接下来，查看汇编代码，`StackVar` 值改为 `0x20`，为了放入 `0x20` 大小的 `fastbins`，接下来把 `f1` 指向了 `StackVar` 以上 `0x8` 处，也就是 `prev_size` 的位置。将 `StackVar` 放入了 `0x20` 的 `fastbins` 中。在 `0x40092C` 处下断点。

![958172_7QJAPUB4DDD2PHB.png](https://attach.52pojie.cn/forum/202310/14/114320l6ey9ep66t8obt1r.png)

`958172_7QJAPUB4DDD2PHB.png`（467.06 KB，2023-10-14 11:42 上传，下载次数：0）

查看堆信息。

![958172_TMDXE86NU7SRWS3.png](https://attach.52pojie.cn/forum/202310/14/114344xnos057ib6si30on.png)

`958172_TMDXE86NU7SRWS3.png`（577.32 KB，2023-10-14 11:43 上传，下载次数：0）

这时候再申请两次便可申请到栈上。

![958172_WBDHQYTZ4AXN4ZB.png](https://attach.52pojie.cn/forum/202310/14/114414kz0r00ktsm03gt5v.png)

`958172_WBDHQYTZ4AXN4ZB.png`（47.41 KB，2023-10-14 11:43 上传，下载次数：0）

在 `0x40095c` 下断点。

![958172_D7WDB4DZ4DJS79F.png](https://attach.52pojie.cn/forum/202310/14/114516szjkqwrm7w9q8wim.png)

`958172_D7WDB4DZ4DJS79F.png`（807.51 KB，2023-10-14 11:44 上传，下载次数：0）

可以看到，已经申请到了栈上的值。

## `unsorted_bin_attack`

`glibc`

### 源码

```c
#include 
#include 

int main(){
    fprintf(stderr, "This file demonstrates unsorted bin attack by write a large unsigned long value into stack\n");
    fprintf(stderr, "In practice, unsorted bin attack is generally prepared for further attacks, such as rewriting the "
           "global variable global_max_fast in libc for further fastbin attack\n\n");

    unsigned long stack_var=0;
    fprintf(stderr, "Let's first look at the target we want to rewrite on stack:\n");
    fprintf(stderr, "%p: %ld\n\n", &stack_var, stack_var);

    unsigned long *p=malloc(400);
    fprintf(stderr, "Now, we allocate first normal chunk on the heap at: %p\n",p);
    fprintf(stderr, "And allocate another normal chunk in order to avoid consolidating the top chunk with"
           "the first one during the free()\n\n");
    malloc(500);

    free(p);
    fprintf(stderr, "We free the first chunk now and it will be inserted in the unsorted bin with its bk pointer "
           "point to %p\n",(void*)p[1]);

    //------------VULNERABILITY-----------

    p[1]=(unsigned long)(&stack_var-2);
    fprintf(stderr, "Now emulating a vulnerability that can overwrite the victim->bk pointer\n");
    fprintf(stderr, "And we write it with the target address-16 (in 32-bits machine, it should be target address-8):%p\n\n",(void*)p[1]);

    //------------------------------------

    malloc(400);
    fprintf(stderr, "Let's malloc again to get the chunk we just free. During this time, the target should have already been "
           "rewritten:\n");
    fprintf(stderr, "%p: %p\n", &stack_var, (void*)stack_var);
}
```

使用 `ubuntu:16.04` 进行编译，然后使用 `pwncli` 改写 `rpath`。

### 调试

首先申请了两个堆块，第一个堆块不属于 `fastbin` 大小，先进入 `unsortedbin` 中，第二个堆块为了防止第一块堆块与 `topchunk` 合并。在 `free` 第一个堆块前设置断点。

![958172_CHJZNE3YDAFDZMS.png](https://attach.52pojie.cn/forum/202310/14/114639ii8svud0li28j9ib.png)

`958172_CHJZNE3YDAFDZMS.png`（76.81 KB，2023-10-14 11:46 上传，下载次数：1）

查看 `bins` 和 `heap` 信息。

![958172_CKURHWFQTKVYSZ4.png](https://attach.52pojie.cn/forum/202310/14/114705oee7wt9eeueqm3qx.png)

`958172_CKURHWFQTKVYSZ4.png`（367.54 KB，2023-10-14 11:47 上传，下载次数：0）

`free` 第一个 chunk 以后，`bins` 和 `heap` 信息，`unsortedbin` 里的第一个 chunk 的 `fd` 和 `bk` 指向 `main_arena+0x58` 的位置。

![958172_E396T5K2YSZR35Q.png](https://attach.52pojie.cn/forum/202310/14/114727qdmm6seebl4g6pbl.png)

`958172_E396T5K2YSZR35Q.png`（530.82 KB，2023-10-14 11:47 上传，下载次数：1）

接下来利用 `uaf` 将 `unsortedbin` 中的第一个 chunk 的 `bk` 指针（`rax` 存储的指针指向 `fd`，`rax+8` 指向 `bk`，`bk` 指向后加入的 chunk）指向 `StackVar` 的 `prev_size` 位置。

![958172_UC93S5J8Q2CNP72.png](https://attach.52pojie.cn/forum/202310/14/114748t9c5oo55dcu229f5.png)

`958172_UC93S5J8Q2CNP72.png`（27.19 KB，2023-10-14 11:47 上传，下载次数：2）

在 `0x4007D9` 处下断点，查看 `heap` 和 `bins` 信息。可以看到，`0x602000` 处的 chunk 的 `bk` 指针被改为了一个栈值，`fd` 指向 `main_arena+0x58` 的位置。

![image-20230922101848230.png](https://attach.52pojie.cn/forum/202310/14/114805qvrarrgyjj8kvmhh.png)

`image-20230922101848230.png`（124.67 KB，2023-10-14 11:47 上传，下载次数：0）

再次将 `unsortedbin` 中第一个 chunk 给 `malloc` 出来以后，`unsortedbin` 中仅剩 `StackVar-0x10`。

![958172_ZJY9Q5VVBT68CZ3.png](https://attach.52pojie.cn/forum/202310/14/114822mkun3zekr4gs5gss.png)

`958172_ZJY9Q5VVBT68CZ3.png`（17.51 KB，2023-10-14 11:48 上传，下载次数：1）

在 `0x400828` 下断点。查看 `heap` 和 `bins` 信息。

![958172_Z4EUWZVH32R7NCC.png](https://attach.52pojie.cn/forum/202310/14/114841mzkmmfwvkvjxazhs.png)

`958172_Z4EUWZVH32R7NCC.png`（162.11 KB，2023-10-14 11:48 上传，下载次数：0）

可以看到，`StackVar` 的 `fd` 指针即用户区域起始处已被修改为 `main_arena+0x58` 的值。

## `unsorted_bin_into_stack`

`glibc`

### 源码
