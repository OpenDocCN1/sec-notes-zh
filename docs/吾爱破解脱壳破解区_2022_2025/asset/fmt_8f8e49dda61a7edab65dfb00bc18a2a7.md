
```
if ((unsigned long) (size) bk)) {
    fwd = bck;
    bck = bck->bk;
    victim->fd_nextsize = fwd->fd;
    victim->bk_nextsize = fwd->fd->bk_nextsize;
    fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
```

## 调试

![](https://attach.52pojie.cn/forum/202310/16/183649rb3q4c2c6izt9tzq.png)

image-20231015202953631.png (75.47 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYwNnwxZjNiM2FmM3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYwNnwxZjNiM2FmM3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

39

2023-10-16 18:36 上传

布置堆结构如上，图中从上到下`chunk`分别为 `p1`, `g1`, `p2`, `g2`。

![](https://attach.52pojie.cn/forum/202310/16/183706vvpbmm7vbp44q7ib.png)

image-20231015203107725.png (80.51 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0OTYwN3w2M2VlNzIyNnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYwN3w2M2VlNzIyNnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

40

2023-10-16 18:37 上传

![](https://attach.52pojie.cn/forum/202310/16/183722hlquc7a747a4nciu.png)

image-20231015203416086.png (104.79 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYwOHw5ZjNjZWU5NXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYwOHw5ZjNjZWU5NXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

41

2023-10-16 18:37 上传

将 `chunk_p1` 放进 `largebin`，将 `chunk_p2` 放进 `unsorted_bin`，(`largebin`)`p1_size` > (`unsorted`)`p2_size`。

![](https://attach.52pojie.cn/forum/202310/16/183737jkoddvzrvvrvcsov.png)

image-20231015203720711.png (97.55 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYwOXw3N2Q3N2U3MnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYwOXw3N2Q3N2U3MnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

42

2023-10-16 18:37 上传

![](https://attach.52pojie.cn/forum/202310/16/183754vpyidgow2nkxcng2.png)

image-20231015203842452.png (49.66 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0OTYxMHxlNmYwOTlhMHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYxMHxlNmYwOTlhMHwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

43

2023-10-16 18:37 上传

修改`p1_bk_nextsize` = `target`-`0x20`，也就是`fake_chunk_fd_nextsize`。

![](https://attach.52pojie.cn/forum/202310/16/183812xk1n85y5770u1x9r.png)

image-20231015204330313.png (173.17 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYxMXxmMWVhODI5ZXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYxMXxmMWVhODI5ZXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

44

2023-10-16 18:38 上传

然后申请 `0x438` 大小的 `chunk`，触发整理机制将 `chunk_p2` 链接进 `largebin`，因为 `p2_size` ，会触发如下代码。

```
// victim:p2, fwd:largebin表头, bck:largebin_least_chunk(p1)

if ((unsigned long) (size) bk)) {

    fwd = bck;

    bck = bck->bk; 

    victim->fd_nextsize = fwd->fd;

    victim->bk_nextsize = fwd->fd->bk_nextsize;

    fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;

}
```

- `p2->fd_nextsize = p1`
- `p2->bk_nextsize = (target-0x20)p1->bk_nextsize`
- `(target-0x20)p1->bk_nextsize = target = p2`

第三步时将 (`target`)`fake_chunk_fd_nextsize` 改为了 `p2_prev`。

![](image-20231015204706099.png)

最后目标地址被成功修改为一个大数。写大数的行为，可以用来修改`global_max_fast`。

## decrypt_safe_linking(glibc > 2.31)

本次使用 `ubuntu:22.04` 进行编译。

### 源码

```
#include 
#include 
#include 

long decrypt(long cipher)
{
    puts("The decryption uses the fact that the first 12bit of the plaintext (the fwd pointer) is known,");
    puts("because of the 12bit sliding.");
    puts("And the key, the ASLR value, is the same with the leading bits of the plaintext (the fwd pointer)");
    long key = 0;
    long plain;

    for(int i=1; i> bits) > 12;
        printf("round %d:\n", i);
        printf("key:    %#016lx\n", key);
        printf("plain:  %#016lx\n", plain);
        printf("cipher: %#016lx\n\n", cipher);
    }
    return plain;
}

int main()
{
    /*
     * This technique demonstrates how to recover the original content from a poisoned
     * value because of the safe-linking mechanism.
     * The attack uses the fact that the first 12 bit of the plaintext (pointer) is known
     * and the key (ASLR slide) is the same to the pointer's leading bits.
     * As a result, as long as the chunk where the pointer is stored is at the same page
     * of the pointer itself, the value of the pointer can be fully recovered.
     * Otherwise, we can also recover the pointer with the page-offset between the storer
     * and the pointer. What we demonstrate here is a special case whose page-offset is 0.
     * For demonstrations of other more general cases, plz refer to
     * https://github.com/n132/Dec-Safe-Linking
     */

    setbuf(stdin, NULL);
    setbuf(stdout, NULL);

    // step 1: allocate chunks
    long *a = malloc(0x20);
    long *b = malloc(0x20);
    printf("First, we create chunk a @ %p and chunk b @ %p\n", a, b);
    malloc(0x10);
    puts("And then create a padding chunk to prevent consolidation.");

    // step 2: free chunks
    puts("Now free chunk a and then free chunk b.");
    free(a);
    free(b);
    printf("Now the freelist is: [%p -> %p]\n", b, a);
    printf("Due to safe-linking, the value actually stored at b[0] is: %#lx\n", b[0]);

    // step 3: recover the values
    puts("Now decrypt the poisoned value");
    long plaintext = decrypt(b[0]);

    printf("value: %p\n", a);
    printf("recovered value: %#lx\n", plaintext);
    assert(plaintext == (long)a);
}
```

### 基础知识

对 `tcache_next(fd)` 新增检查:

```
// 加密

#define PROTECT_PTR(pos, ptr) \
  ((__typeof (ptr)) ((((size_t) pos) >> 12) ^ ((size_t) ptr)))

// 解密

#define REVEAL_PTR(ptr)  PROTECT_PTR (&ptr, ptr)

/*

* 原理: A:fd;B:(pos>>12);C:(ptr); A=B^C; C=A^B;

*/
```

![](https://attach.52pojie.cn/forum/202310/16/183835xiojjjpqqz2pa9o5.png)

eg.png (144.29 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0OTYxMnw2NzU1ZDFjN3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYxMnw2NzU1ZDFjN3wxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

45

2023-10-16 18:38 上传

`P` 表示将保存在空闲块的 `fd` 字段中的指针值。`L` 表示 `fd` 字段本身的地址。`L>>12`是 `L` 的右移值，用于对 `P` 进行异或运算，从而产生一个编码指针`P'`。Safe Linking 将这个`P'`值存储在 `fd` 字段中。

![](https://attach.52pojie.cn/forum/202310/16/183852m3jq4vewno8nmjpr.png)

image-20231016094308974.png (25.98 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYxM3w2Y2Y0ZDk1MnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYxM3w2Y2Y0ZDk1MnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

46

2023-10-16 18:38 上传

![](https://attach.52pojie.cn/forum/202310/16/183908ywryd1d1rgy7w16r.png)

image-20231016094240180.png (47.03 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYxNHw0N2UyZjdmOXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYxNHw0N2UyZjdmOXwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

47

2023-10-16 18:39 上传

![](https://attach.52pojie.cn/forum/202310/16/183923rfs2hu2hnhfdbnhf.png)

image-20231016094355521.png (15.82 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0OTYxNXw4NzQ3OWU0YnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0OTYxNXw4NzQ3OWU0YnwxNzkzNTczMDY5fDE2ODA4NnwxODQ1MDky&nothumb=yes)

48

2023-10-16 18:39 上传

bypass safe-linking机制需要用到 `uaf`或者 `double free` 之类的漏洞, 同时释放 `tcache`到一个空闲 `tacahe bin`中, 此时由于`tcache bin` 中没有空闲`chunk`, `tcache->entry[tc_idx]=0`，若存在 `uaf` 或者 `double free`，可以泄露出 `leak_addr= (&tcache_chunk->fd)>>12` 位置, 则 `heap_base=leak_addr`。`double free` 需要将 `tcache_chunk_bk` 改为 `0`，绕过检查。对于 `2.32`及以后的 `glibc` 版本的 `tcache_poisoning` 需要将 `target` 地址进行加密。

### 调试

![](https://attach.52pojie.cn/forum/202310/16/183944dsvr4mrrm1ser46o.png)
