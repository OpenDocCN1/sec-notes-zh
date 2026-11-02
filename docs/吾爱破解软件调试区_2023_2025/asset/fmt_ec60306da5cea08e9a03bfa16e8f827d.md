
# [漏洞分析] 【CVE-2023-2008】Linux kernel [dma-buf子系统 udmabuf]页面越界读写漏洞分析与利用

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1996104
- 抓取时间: 2026-11-01T21:52:37

## 0x00. 一切开始之前

[//nvd.nist.gov/vuln/detail/CVE-2023-2008" target="_blank" rel="noopener noreferrer nofollow"](https://nvd.nist.gov/vuln/detail/CVE-2023-2008:rel="nofollow noopener" href="https) 是一个发生在 Linux 内核的 [//docs.kernel.org/driver-api/dma-buf.html" target="_blank" rel="noopener noreferrer nofollow"](https://docs.kernel.org/driver-api/dma-buf.html:rel="nofollow noopener" href="https) 的 `/dev/udmabuf` 设备驱动中的一个漏洞，由于缺乏对范围扩张后的 `udmabuf::pages` 的边界检查，导致攻击者可以完成越界页面指针映射，从而完成越权文件写入攻击以进行提权；该漏洞影响版本为 4.20 ~ 5.4.201 、5.5 ~ 5.10.126 、5.11~5.15.50、5.16 ~ 5.18.7 ，本文我们选用 5.11 版本的内核源码进行分析
原本打算在 2025 年到来之前写完发出去的，结果今晚被导师叫出去吃大餐，所以最后拖到了 2025 年的第一天凌晨才慢慢写完：）
在开始之前，我们先补充一些 `dma-buf` 与 `udmabuf` 的基础知识

### 基本原理

[//docs.kernel.org/driver-api/dma-buf.html" target="_blank" rel="noopener noreferrer nofollow"](https://docs.kernel.org/driver-api/dma-buf.html:rel="nofollow noopener" href="https) 为不同设备驱动与其他子系统提供了一个方便统一的 内存共享框架 ，常用于 GPU 及显示驱动等，其主要有三个用于交互的部分：
- `dma-buf` ：表示一个 `sg_table` ，并作为文件描述符导出到用户空间以允许在进程、子系统、设备间传递（注：`UNIX domain socket`）
- `dma-fence` ：提供了对异步硬件操作是否完成的检测机制
- `dma-resv` ：为一个特定的 `dma-buf` 管理一组 `dma-fence` ，通过隐式的（内核排序的）同步工作以保持表面上的一致访问

在 `dma-buf` 框架中主要有两个对象，分别是：
- `exporter` ：共享内存的生产者（内核驱动）
  - 负责为内存实现 `struct dma_buf_ops` 操作函数表
  - 允许通过 `dma_buf` API 共享内存
  - 通过 `struct dma_buf` 管理内存分配的细节
  - 决定实际的分配，并负责所有共享用户的 [//www.wowotech.net/memory_management/scatterlist.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.wowotech.net/memory_management/scatterlist.html:rel="nofollow noopener" href="https) （`page` 为单位的物理连续内存块）迁移
- `importer`/`user` ：共享内存的消费者（可以是用户态程序也可以是内核态驱动）
  - 不关心内存来源
  - 通过 `struct dma_buf_attachment` 接口访问共享内存（一般通过共享的文件描述符获取）

![](https://s2.loli.net/2024/12/20/RUPS1INVcml4kiL.png)

> 

#### 扩展阅读：sg_table 与 scatterlist - 离散连续内存

简而言之，单个 `scatterlist` 用来表示一块 物理连续 的 以页为单位 的内存块，`sg_table` 则表示一个离散 `scatterlist` 数组：
```
struct scatterlist {
        unsigned long        page_link;
        unsigned int        offset;
        unsigned int        length;
        dma_addr_t        dma_address;
#ifdef CONFIG_NEED_SG_DMA_LENGTH
        unsigned int        dma_length;
#endif
#ifdef CONFIG_NEED_SG_DMA_FLAGS
        unsigned int    dma_flags;
#endif
};

struct sg_table {
        struct scatterlist *sgl;        /* the list */
        unsigned int nents;                /* number of mapped entries */
        unsigned int orig_nents;        /* original size of list */
};
```
其通常结构如下图所示：
![](https://s2.loli.net/2024/09/20/Rr5akvXyLQCbMUd.png)

对于分配的 `scatterlist` 数组长度大于一张内存页的情况，其会被分割成多块，其中单张 `scatterlist` 数组页面上的最后一个成员的 `page_link` 字段会用于标识是否存在下一份页面以及下一份 `scatterlist` 页面所在内存页：
![](https://s2.loli.net/2024/09/20/V8w2i9QyE7Ndrg5.png)

具体而言，`dma-buf` 的通常使用流程如下：
- `exporter` 侧用 `DEFINE_DMA_BUF_EXPORT_INFO()` 创建一个临时 `dma_buf_export_info` 结构体，并定义导出信息，这通常包括：
  - `::ops`：自定义的 `struct dma_buf_ops`
  - `::priv`：自定义私有数据的指针
  - `::size` ：共享内存大小
  - `::flags` ：内存读写权限
- `exporter` 侧调用 `dma_buf_export()` 创建 `dma_buf` 对象
- `exporter` 侧调用 `dma_buf_fd()` 将 `dma_buf` 对象关联到一个新的文件描述符 `fd` 中（注：使用 `fd` 进行传递是比较常见的方式，但不唯一）
- `exporter` 侧将 `fd` 传给 `importer`
- `importer` 调用 `dma_buf_get(fd)` 获取 `dma_buf` 对象
- `importer` 调用 `dma_buf_attach()` 和 `dma_buf_map_attachment()` 获取共享缓存的信息
- `importer` 使用 `sg_table` 的内容
- `importer` 调用 `dma_buf_detach()` 和 `dma_buf_unmap_attachment()` 释放对 `dma_buf` 的引用
- `importer` 调用 `dma_buf_put(fd)` 释放文件描述符

![](https://s2.loli.net/2024/10/08/li3mnaFSkyrcpOW.png)

对于涉及用户态的 DMA-BUF 使用而言，将 `dma_buf` 封装到 `fd` 当中通常是必须的：`exporter` 获取共享内存对应的 `dma-buf` 的文件描述符 `fd` ，通过 `ioctl` 或是 `UNIX domain socket` 等方式发送给 `importer`，`importer` 再 `mmap(fd)` 以访问共享内存：
![](https://s2.loli.net/2024/09/20/5kEc7D8T3gaHw4l.png)

> 

需要注意的是 `exporter` 与 `importer` 并不总是分离的，也可以有同时存在于同一驱动的情况（例如 `DRM`）

### `udmabuf`：简易封装接口

得益于 `dma_buf` 的好用，Linux kernel [//patchwork.kernel.org/project/qemu-devel/patch/20180313154826.20436-1-kraxel@redhat.com/" target="_blank" rel="noopener noreferrer nofollow"](https://patchwork.kernel.org/project/qemu-devel/patch/20180313154826.20436-1-kraxel@redhat.com/:rel="nofollow noopener" href="https)引入了一个封装好的 `dma_buf` 设备驱动——`udmabuf` ，其在内核中实现了一个封装好的 `exporter`，并支持通过 `ioctl()` 创建 `dma_buf` 以在用户态程序之间进行内存共享

`udmabuf` 的基本使用方式非常简单：
- 使用 `memfd_create()` 创建一个文件描述符
- 封装 `udmabuf_create` 请求，通过 `ioctl("/dev/udmabuf")` 获取 `dma_buf` 的 `fd`
- 通过 `mmap()` 进行访问

有了 `dma_buf` 的 `fd`，后面的内存访问操作就和之前没什么区别了，在进程间进行共享只需要共享这个文件描述符即可（例如使用 `UNIX domain socket`）

`udmabuf` 的实现非常简单，主要就是注册了一个杂项设备 `/dev/udmabuf` ，并支持两个自定义的 `IOCTL` 命令：
> 

`/drivers/dma-buf/udmabuf.c`
```
static long udmabuf_ioctl(struct file *filp, unsigned int ioctl,
                          unsigned long arg)
{
        long ret;
        switch (ioctl) {
        case UDMABUF_CREATE:
                ret = udmabuf_ioctl_create(filp, arg);
                break;
        case UDMABUF_CREATE_LIST:
                ret = udmabuf_ioctl_create_list(filp, arg);
                break;
        default:
                ret = -ENOTTY;
                break;
        }
        return ret;
}

static const struct file_operations udmabuf_fops = {
        .owner                = THIS_MODULE,
        .unlocked_ioctl = udmabuf_ioctl,
#ifdef CONFIG_COMPAT
        .compat_ioctl   = udmabuf_ioctl,
#endif
};

static struct miscdevice udmabuf_misc = {
        .minor          = MISC_DYNAMIC_MINOR,
        .name           = "udmabuf",
        .fops           = &udmabuf_fops,
};
```
`udmabuf_ioctl_create()` 与 `udmabuf_ioctl_create_list()` 最终都会调用到 `udmabuf_create()` ，其会按照 DMA-BUF 的常规流程创建一个 `dma_buf()` 并分配对应的内存页：
