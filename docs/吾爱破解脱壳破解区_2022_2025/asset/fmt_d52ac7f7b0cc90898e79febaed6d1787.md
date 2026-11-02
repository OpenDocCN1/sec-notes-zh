
# [CTF] how2heap全集及例题详细解析(5~6部分)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1845092
- 抓取时间: 2026-11-01T22:44:31

## 第五部分前言

第五部分开始使用 `ubuntu:18.04` 编译。Tcache 基础请看 [//bbs.kanxue.com/thread-278105.htm" target="_blank" rel="noopener noreferrer nofollow"](https://bbs.kanxue.com/thread-278105.htm:rel="nofollow noopener" href="https)。

### fastbin_reverse_into_tcache

#### 源码

```
#include 
#include 
#include 
#include 

const size_t allocsize = 0x40;

int main(){
  setbuf(stdout, NULL);

  printf(
    "\n"
    "This attack is intended to have a similar effect to the unsorted_bin_attack,\n"
    "except it works with a small allocation size (allocsize 
```

#### 调试

![](https://attach.52pojie.cn/forum/202310/16/182534ww03zfv728k0pc4v.png)

image-20231014193723730.png (64.92 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTU2N3wyNGJkNGQzOHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU2N3wyNGJkNGQzOHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

1

2023-10-16 18:25 上传

![](https://attach.52pojie.cn/forum/202310/16/182550hccydcgvicquiiri.png)

image-20231014193755564.png (22.29 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTU2OHxiMGU4ZTNiNHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU2OHxiMGU4ZTNiNHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

2

2023-10-16 18:25 上传

![](https://attach.52pojie.cn/forum/202310/16/182609j30r5n41ppjr8rw0.png)

image-20231014193900312.png (46.13 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTU2OXw2NWIzOTAyM3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU2OXw2NWIzOTAyM3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

3

2023-10-16 18:26 上传

![](https://attach.52pojie.cn/forum/202310/16/182626hce4383l3wzlttlk.png)

image-20231014193928870.png (24.14 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTU3MHwwNmRhM2E3M3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3MHwwNmRhM2E3M3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

4

2023-10-16 18:26 上传

首先申请 14 个 `chunk` ，先后将 `tcache` 和 `fastbinY[4]` 填满。其中 `victim` 指向第 8 个 `chunk` 也就是 `fastbinY[4]` 的最后一个 `chunk_ptrs[7]`。

![](https://attach.52pojie.cn/forum/202310/16/182644eyouww3w626ui4z3.png)

image-20231014194618108.png (25 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTU3MXxmOWRkMzQzY3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3MXxmOWRkMzQzY3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

5

2023-10-16 18:26 上传

![](https://attach.52pojie.cn/forum/202310/16/182700brir077es77hjfwy.png)

image-20231014194639500.png (26.88 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0OTU3MnxhMDdhZjI5YnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3MnxhMDdhZjI5YnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

6

2023-10-16 18:27 上传

将 `victim(ptrs[7]_fd)` 指向 `stack_var[0]` 的位置，然后将 `tcache` 清空。

![](https://attach.52pojie.cn/forum/202310/16/182716g9r9irllvdpz39zi.png)

image-20231014194826822.png (45.74 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0OTU3M3w1ZjI0MTE2Y3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3M3w1ZjI0MTE2Y3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

7

2023-10-16 18:27 上传

![](https://attach.52pojie.cn/forum/202310/16/182733aw7ks6zjm66l7iej.png)

image-20231014195140863.png (57.55 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0OTU3NHw5MjAzNjA0OHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3NHw5MjAzNjA0OHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

8

2023-10-16 18:27 上传

这次 `malloc` 将会先从 `fastbin` 头部取出一个 `chunk`，然后把 `fastbin` 清空，放入 `tcache` 中，因为 `fastbin` 取出时从头开始，`tcache` 又是 FIFO 结构， 所以放入 `tcache` 是倒序的，把 `stack_var` 也算做了一个 `chunk`，所以是满 7 个。

![](https://attach.52pojie.cn/forum/202310/16/182749k6bzc0xlld4xx9yc.png)

image-20231014195254059.png (57.87 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTU3NXw0MzU0MTU2Y3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3NXw0MzU0MTU2Y3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

9

2023-10-16 18:27 上传

![](https://attach.52pojie.cn/forum/202310/16/182802f0fanafjjm06fsf6.png)

image-20231014195309680.png (19.48 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0OTU3Nnw5OTM1ZGRmZHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3Nnw5OTM1ZGRmZHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

10

2023-10-16 18:28 上传

此时再去申请一个 `0x50` 大小的 `chunk` 将会把 `stack_var` 取出来，此时 `q == stack_var[2]` 。

### house_of_botcake

`libc-2.29` 新增加 `double free` 检查，方法是在 `tcache_entry` 结构体中新增加标志位 `key` 来检查 `chunk` 是否在 `tcache` bin 中。当 `free` 掉一个堆块进入 `tcache` 时，假如堆块的 `bk` 位存放的 `key == tcache_key`， 就会遍历这个大小的 `Tcache` ，假如发现同地址的堆块，则触发 `double Free` 报错。因为 `chunk` 的 `key` 保存在 `bk` 位置，只需将其修改即可绕过 `double free` 检查。而 `house_of_botcake` 是另一种方法。

#### 源码

```
#include 
#include 
#include 
#include 

int main()
{
    /*
     * This attack should bypass the restriction introduced in
     * https://sourceware.org/git/?p=glibc.git;a=commit;h=bcdaad21d4635931d1bd3b54a7894276925d081d
     * If the libc does not include the restriction, you can simply double free the victim and do a
     * simple tcache poisoning
     * And thanks to @anton00b and @subwire for the weird name of this technique */
    // disable buffering so _IO_FILE does not interfere with our heap
    setbuf(stdin, NULL);
    setbuf(stdout, NULL);
    // introduction
    puts("This file demonstrates a powerful tcache poisoning attack by tricking malloc into");
    puts("returning a pointer to an arbitrary location (in this demo, the stack).");
    puts("This attack only relies on double free.\n");
    // prepare the target
    intptr_t stack_var[4];
    puts("The address we want malloc() to return, namely,");
    printf("the target address is %p.\n\n", stack_var);
    // prepare heap layout
    puts("Preparing heap layout");
    puts("Allocating 7 chunks(malloc(0x100)) for us to fill up tcache list later.");
    intptr_t *x[7];
    for(int i=0; i
```

#### 调试

![](https://attach.52pojie.cn/forum/202310/16/182824cgy6q66y44mugo66.png)

image-20231015110448627.png (84.08 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTU3N3w2NWVkMzAxZXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3N3w2NWVkMzAxZXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

11

2023-10-16 18:28 上传

![](https://attach.52pojie.cn/forum/202310/16/182844ejm3mjmppjvljky1.png)

image-20231015110515624.png (71.29 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTU3OHxlNWE0YjExZXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3OHxlNWE0YjExZXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

12

2023-10-16 18:28 上传

![](https://attach.52pojie.cn/forum/202310/16/182900avk6w6vddww23wdw.png)

image-20231015110726890.png (46.51 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTU3OXw2NjkyNjdhMHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTU3OXw2NjkyNjdhMHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

13

2023-10-16 18:29 上传

首先申请 9 个 `non-fast_chunk` 和一个 `obstruct-chunk` ，将 `tcache` 填满，剩余两个放入 `unsorted_bin`，因为 `a` 与 `prev` 相邻，所以会被整合在一起。
