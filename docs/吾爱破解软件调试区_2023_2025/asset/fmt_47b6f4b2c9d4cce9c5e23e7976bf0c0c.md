
# struct page 关键字段解析

## 结构体定义

```c
union {     /* This union is 4 bytes in size. */

        /*

         * If the page can be mapped to userspace, encodes the number

         * of times this page is referenced by a page table.

         */

        atomic_t _mapcount;

        /*

         * If the page is neither PageSlab nor mappable to userspace,

         * the value stored here may help determine what this page

         * is used for.  See page-flags.h for a list of page types

         * which are currently stored here.

         */

        unsigned int page_type;

        unsigned int active;        /* SLAB */

        int units;          /* SLOB */

    };

    /* Usage count. *DO NOT USE DIRECTLY*. See page_ref.h */

    atomic_t _refcount;

#ifdef CONFIG_MEMCG

    unsigned long memcg_data;

#endif

    /*

     * On machines where all RAM is mapped into kernel address space,

     * we can simply calculate the virtual address. On machines with

     * highmem some memory is mapped into kernel virtual memory

     * dynamically, so we need a place to store that address.

     * Note that this field could be 16 bits on x86 ... ;)

     *

     * Architectures with slow multiplication can define

     * WANT_PAGE_VIRTUAL in asm/page.h

     */

#if defined(WANT_PAGE_VIRTUAL)

    void *virtual;          /* Kernel virtual address (NULL if

                       not kmapped, ie. highmem) */

#endif /* WANT_PAGE_VIRTUAL */

#ifdef LAST_CPUPID_NOT_IN_PAGE_FLAGS

    int _last_cpupid;

#endif

} _struct_page_alignment;
```

老样子，先解释关键字段

## 关键字段说明

### lru
`lru`：最近未使用页这个概念在计算机组成原理或者说操作系统课程里面都会讲解，这里也就不过多描述，在`linux`内核当中，`page`通过该字段来组织成链表

### slab 相关
`slab`相关：用来存放`slab`相关成员

```c
struct {    /* slab, slob and slub */
            union {
                struct list_head slab_list;
                struct {    /* Partial pages */
                    struct page *next;
#ifdef CONFIG_64BIT
                    int pages;  /* Nr of pages left */
                    int pobjects;   /* Approximate count */
#else
                    short int pages;    
                    short int pobjects;
#endif
                };
            };
            struct kmem_cache *slab_cache; /* not slob */
            /* Double-word boundary */
            void *freelist;     /* first free object */
            union {
                void *s_mem;    /* slab: first object */
                unsigned long counters;     /* SLUB */
                struct {            /* SLUB */
                    unsigned inuse:16;
                    unsigned objects:15;
                    unsigned frozen:1;
                };
            };
        };
```

下面给出又一张十分详细的图，是由简·李奥师傅所作

![](http://imgsrc.baidu.com/forum/pic/item/4ec2d5628535e5dd881a441633c6a7efcf1b62c1.jpg)

### flags
`flags`：表示该页所处在的状态，定义于`include/linux/page-flags.h`当中，他是一个枚举类型，如下：

```c
enum pageflags {

    PG_locked,        /* Page is locked. Don't touch. */

    PG_referenced,

    PG_uptodate,

    PG_dirty,

    PG_lru,

    PG_active,

    PG_workingset,

    PG_waiters,        /* Page has waiters, check its waitqueue. Must be bit #7 and in the same byte as "PG_locked" */

    PG_error,

    PG_slab,

    PG_owner_priv_1,    /* Owner use. If pagecache, fs may use*/

    PG_arch_1,

    PG_reserved,

    PG_private,        /* If pagecache, has fs-private data */

    PG_private_2,        /* If pagecache, has fs aux data */

    PG_writeback,        /* Page is under writeback */

    PG_head,        /* A head page */

    PG_mappedtodisk,    /* Has blocks allocated on-disk */

    PG_reclaim,        /* To be reclaimed asap */

    PG_swapbacked,        /* Page is backed by RAM/swap */

    PG_unevictable,        /* Page is "unevictable"  */

#ifdef CONFIG_MMU

    PG_mlocked,        /* Page is vma mlocked */

#endif

#ifdef CONFIG_ARCH_USES_PG_UNCACHED

    PG_uncached,        /* Page has been mapped as uncached */

#endif

#ifdef CONFIG_MEMORY_FAILURE

    PG_hwpoison,        /* hardware poisoned page. Don't touch */

#endif

#if defined(CONFIG_IDLE_PAGE_TRACKING) && defined(CONFIG_64BIT)

    PG_young,

    PG_idle,

#endif

#ifdef CONFIG_64BIT

    PG_arch_2,

#endif

    __NR_PAGEFLAGS,

    /* Filesystems */

    PG_checked = PG_owner_priv_1,

    /* SwapBacked */

    PG_swapcache = PG_owner_priv_1,    /* Swap page: swp_entry_t in private */

    /* Two page bits are conscripted by FS-Cache to maintain local caching

     * state.  These bits are set on pages belonging to the netfs's inodes

     * when those inodes are being locally cached.

     */

    PG_fscache = PG_private_2,    /* page backed by cache */

    /* XEN */

    /* Pinned in Xen as a read-only pagetable page. */

    PG_pinned = PG_owner_priv_1,

    /* Pinned as part of domain save (see xen_mm_pin_all()). */

    PG_savepinned = PG_dirty,

    /* Has a grant mapping of another (foreign) domain's page. */

    PG_foreign = PG_owner_priv_1,

    /* Remapped by swiotlb-xen. */

    PG_xen_remapped = PG_owner_priv_1,

    /* SLOB */

    PG_slob_free = PG_private,

    /* Compound pages. Stored in first tail page's flags */

    PG_double_map = PG_workingset,

    /* non-lru isolated movable page */

    PG_isolated = PG_reclaim,

    /* Only valid for buddy pages. Used to track pages that are reported */

    PG_reported = PG_uptodate,

};
```

这里采用的复用的手法，也就是说`flags`字段还容纳了其他元素，如下，结构划分位于`/include/linux/page-flags-layout.h`当中

```c
/*
 * page->flags layout:
 *
 * There are five possibilities for how page->flags get laid out.  The first
 * pair is for the normal case without sparsemem. The second pair is for
 * sparsemem when there is plenty of space for node and section information.
 * The last is when there is insufficient space in page->flags and a separate
 * lookup is necessary.
 *
 * No sparsemem or sparsemem vmemmap: |       NODE     | ZONE |             ... | FLAGS |
 *      " plus space for last_cpupid: |       NODE     | ZONE | LAST_CPUPID ... | FLAGS |
 * classic sparse with space for node:| SECTION | NODE | ZONE |             ... | FLAGS |
 *      " plus space for last_cpupid: | SECTION | NODE | ZONE | LAST_CPUPID ... | FLAGS |
 * classic sparse no space for node:  | SECTION |     ZONE    | ... | FLAGS |
 */
```

可以看到在不同布局下他其实是可以用作指向归属的`zone`和`node`的

### _mapcount
`_mapcount`：记录该页被页表映射的次数，初始值为-1，他是一个根据不同情况所采用的联合结构体，如果说他是被用户空间所映射，那么他会记录被映射的次数，但若是他没被映射到用户空间，页不是`PageSlab`,那么他为`page_type`字段，它定义于`/include/linux/page-flags.h`字段当中，如下：

```c
/*
 * For pages that are never mapped to userspace (and aren't PageSlab),
 * page_type may be used.  Because it is initialised to -1, we invert the
 * sense of the bit, so __SetPageFoo *clears* the bit used for PageFoo, and
 * __ClearPageFoo *sets* the bit used for PageFoo.  We reserve a few high and
 * low bits so that an underflow or overflow of page_mapcount() won't be
 * mistaken for a page type value.
 */

#define PAGE_TYPE_BASE    0xf0000000

/* Reserve        0x0000007f to catch underflows of page_mapcount */

#define PAGE_MAPCOUNT_RESERVE    -128

#define PG_buddy    0x00000080

#define PG_offline    0x00000100

#define PG_table    0x00000200

#define PG_guard    0x00000400
```

### _refcount
`_refcount`：用作该页在内核中的引用次数，初值为0，若大于0表示正在被使用，等于0表示空闲或将要被释放，内核函数`get_page()`和`put_page()`函数会来进行引用计数的增减，后者若引用计数器为1则会调用`__put_single_page()`释放该页面

### virtual
`vitrual`：指向物理页框对应虚拟地址（这里有点疑问那就是他被多个页表映射咋办捏，还是说每次切换进程的时候会刷新一下这里呢？）

## 内存模型
说完数据结构，还记得上面`flags`不同布局下对应的结构吗，`linux`一般提供了三种内存模型，定义在`/include/asm-generic/memory_model.h`

![](http://imgsrc.baidu.com/forum/pic/item/2fdda3cc7cd98d106098edf8643fb80e7aec90af.jpg)

常用模型是`sparsemem`,所以我们只了解他，中文翻译过来就是离散内存模型。在这个模型下，内存中会存在一个`mem_section`类型的指针数组，而其中元素指向的`mem_section`结构体中的`section_mem_map`成员会指向一个`struct page`类型的数组，它对应于一个连续的物理地址空间，如下图所示

![](http://imgsrc.baidu.com/forum/pic/item/962bd40735fae6cdb11e4aa84ab30f2443a70f55.jpg)

其中`mem_section`结构体的定义在`/include/linux/mmzone.h`当中，如下：
