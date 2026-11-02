
### 节点状态枚举

```
    N_MEMORY,        /* 节点有着内存(普通，高端，可移动) */
    N_CPU,        /* 节点有着一个或多个 cpu */
    N_GENERIC_INITIATOR,    /* 节点有一个或多个 Generic Initiators */
    NR_NODE_STATES
};
```

说完`node`，我来绘个图吧，这里老抄作业好像体现不出自己真正学到了东西

![](http://imgsrc.baidu.com/forum/pic/item/4d086e061d950a7b0985aa0d4fd162d9f3d3c943.jpg)

我们将在之后一步一步慢慢完善这个图片

## zone区域

同样的，先说其数据结构`struct zone`，他位于`/include/linux/mmzone.h`

```
struct zone {
    /* Read-mostly fields */

    /* zone watermarks, access with *_wmark_pages(zone) macros */
    unsigned long _watermark[NR_WMARK];
    unsigned long watermark_boost;

    unsigned long nr_reserved_highatomic;

    /*
     * We don't know if the memory that we're going to allocate will be
     * freeable or/and it will be released eventually, so to avoid totally
     * wasting several GB of ram we must reserve some of the lower zone
     * memory (otherwise we risk to run OOM on the lower zones despite
     * there being tons of freeable ram on the higher zones).  This array is
     * recalculated at runtime if the sysctl_lowmem_reserve_ratio sysctl
     * changes.
     */
    long lowmem_reserve[MAX_NR_ZONES];

#ifdef CONFIG_NUMA
    int node;
#endif
    struct pglist_data  *zone_pgdat;
    struct per_cpu_pageset __percpu *pageset;
    /*
     * the high and batch values are copied to individual pagesets for
     * faster access
     */
    int pageset_high;
    int pageset_batch;

#ifndef CONFIG_SPARSEMEM
    /*
     * Flags for a pageblock_nr_pages block. See pageblock-flags.h.
     * In SPARSEMEM, this map is stored in struct mem_section
     */
    unsigned long       *pageblock_flags;
#endif /* CONFIG_SPARSEMEM */

    /* zone_start_pfn == zone_start_paddr >> PAGE_SHIFT */
    unsigned long       zone_start_pfn;

    /*
     * spanned_pages is the total pages spanned by the zone, including
     * holes, which is calculated as:
     *  spanned_pages = zone_end_pfn - zone_start_pfn;
     *
     * present_pages is physical pages existing within the zone, which
     * is calculated as:
     *  present_pages = spanned_pages - absent_pages(pages in holes);
     *
     * managed_pages is present pages managed by the buddy system, which
     * is calculated as (reserved_pages includes pages allocated by the
     * bootmem allocator):
     *  managed_pages = present_pages - reserved_pages;
     *
     * So present_pages may be used by memory hotplug or memory power
     * management logic to figure out unmanaged pages by checking
     * (present_pages - managed_pages). And managed_pages should be used
     * by page allocator and vm scanner to calculate all kinds of watermarks
     * and thresholds.
     *
     * Locking rules:
     *
     * zone_start_pfn and spanned_pages are protected by span_seqlock.
     * It is a seqlock because it has to be read outside of zone->lock,
     * and it is done in the main allocator path.  But, it is written
     * quite infrequently.
     *
     * The span_seq lock is declared along with zone->lock because it is
     * frequently read in proximity to zone->lock.  It's good to
     * give them a chance of being in the same cacheline.
     *
     * Write access to present_pages at runtime should be protected by
     * mem_hotplug_begin/end(). Any reader who can't tolerant drift of
     * present_pages should get_online_mems() to get a stable value.
     */
    atomic_long_t       managed_pages;
    unsigned long       spanned_pages;
    unsigned long       present_pages;

    const char      *name;

#ifdef CONFIG_MEMORY_ISOLATION
    /*
     * Number of isolated pageblock. It is used to solve incorrect
     * freepage counting problem due to racy retrieving migratetype
     * of pageblock. Protected by zone->lock.
     */
    unsigned long       nr_isolate_pageblock;
#endif

#ifdef CONFIG_MEMORY_HOTPLUG
    /* see spanned/present_pages for more description */
    seqlock_t       span_seqlock;
#endif

    int initialized;

    /* Write-intensive fields used from the page allocator */
    ZONE_PADDING(_pad1_)

    /* free areas of different sizes */
    struct free_area    free_area[MAX_ORDER];

    /* zone flags, see below */
    unsigned long       flags;

    /* Primarily protects free_area */
    spinlock_t      lock;

    /* Write-intensive fields used by compaction and vmstats. */
    ZONE_PADDING(_pad2_)

    /*
     * When free pages are below this point, additional steps are taken
     * when reading the number of free pages to avoid per-cpu counter
     * drift allowing watermarks to be breached
     */
    unsigned long percpu_drift_mark;

#if defined CONFIG_COMPACTION || defined CONFIG_CMA
    /* pfn where compaction free scanner should start */
    unsigned long       compact_cached_free_pfn;
    /* pfn where compaction migration scanner should start */
    unsigned long       compact_cached_migrate_pfn[ASYNC_AND_SYNC];
    unsigned long       compact_init_migrate_pfn;
    unsigned long       compact_init_free_pfn;
#endif

#ifdef CONFIG_COMPACTION
    /*
     * On compaction failure, 1
```
同样地，我们来了解其中比较重要的字段

- `_watermark` 水位线，一般表示剩余空闲页框，他又三个挡位，分别是`WMARK_MIN`、`WMARK_LOW`、`WMARK_HIGH`，他存放在`_watermark`数组当中，进行内存分配的时候，分配器会根据当前水位来采取不同的措施，下面搞个图：
![](http://imgsrc.baidu.com/forum/pic/item/38dbb6fd5266d016ce8f7ae1d22bd40734fa3561.jpg)

- `lowmem_reserve`：当本`zone`没有空闲块之后，会到别的`zone`中进行分配，避免分配内存全分配在低端`zone`，而我们不能保证这里分配的内存是可释放，或者最终会被释放的，出现低端`zone`区域内存提前耗尽，而高端`zone`区保留大量内存，因此声名该字段来保留一段内存，而这里的`zone`区内存是其他`zone`不能打扰的

- `node`：标识该`zone`所属`node`，当然，这里只在`NUMA`启动，`UMA`中只有一个`node`，不需要这个字段

- `zone_pgdat`：标识所属的`pglist_data`节点，同上面的`node`

- `pageset`：由于目前都是多处理器CPU架构，因此对于临界区的同步互斥访问就是一个严重的问题，而防止出错的办法之一加锁解锁十分浪费资源，因此每个`zone`当中都为每一个CPU准备一个单独的页面仓库，最开始`buddy system`会首先将页面放置在各个CPU独自的页面仓库当中，需要进行分配的时候优先从其中分配，其类型结构体位于`/include/linux/mmzone.h`

```
struct per_cpu_pages {

    int count;      /* number of pages in the list */

    int high;       /* high watermark, emptying needed */

    int batch;      /* chunk size for buddy add/remove */

    /* Lists of pages, one per migrate type stored on the pcp-lists */

    struct list_head lists[MIGRATE_PCPTYPES]; //双链表指针数组，指向空闲页们

};

struct per_cpu_pageset {

    struct per_cpu_pages pcp;

#ifdef CONFIG_NUMA

    s8 expire;

    u16 vm_numa_stat_diff[NR_VM_NUMA_STAT_ITEMS];

#endif

#ifdef CONFIG_SMP

    s8 stat_threshold;

    s8 vm_stat_diff[NR_VM_ZONE_STAT_ITEMS];

#endif

};
```

此结构是一个包括状态，他会被存放在每个CPU独立的.data..percpu段当中，下面再再再次引用`arttnba3`师傅的图，真的态🐂辣
![](http://imgsrc.baidu.com/forum/pic/item/902397dda144ad340c83614e95a20cf431ad853f.jpg)

- `zone_start_pfn`：该`zone`的起始物理地址编号`pfn`（page frame number）

- `spanned_pages`：本`zone`区域中内存的`page`总数

- `present_pages`：本`zone`中实际存在的物理页框数

- `managed_pages`：本`zone`中`buddy system`管理的页面数量

- `free_area`：`buddy_system`按照`order`管理的页面，为一个`free_area`结构体数组，具体定义如下：

```
struct free_area {
    struct list_head    free_list[MIGRATE_TYPES];
    unsigned long        nr_free;
};
```

看图好吧,这个`order`起始就是伙伴系统中的对于不同大小页分配的请求大小
![](http://imgsrc.baidu.com/forum/pic/item/f3d3572c11dfa9ec9b54b0de27d0f703908fc185.jpg)

- `flags`：标识`zone`的状态

- `vm_stat`：统计数据，这里是一个数组，而数组大小取决于定义的枚举类型，如下：

```
enum zone_stat_item {

    /* First 128 byte cacheline (assuming 64 bit words) */

    NR_FREE_PAGES,

    NR_ZONE_LRU_BASE, /* Used only for compaction and reclaim retry */

    NR_ZONE_INACTIVE_ANON = NR_ZONE_LRU_BASE,

    NR_ZONE_ACTIVE_ANON,

    NR_ZONE_INACTIVE_FILE,

    NR_ZONE_ACTIVE_FILE,

    NR_ZONE_UNEVICTABLE,

    NR_ZONE_WRITE_PENDING,  /* Count of dirty, writeback and unstable pages */

    NR_MLOCK,       /* mlock()ed pages found and moved off LRU */

    /* Second 128 byte cacheline */

    NR_BOUNCE,

#if IS_ENABLED(CONFIG_ZSMALLOC)

    NR_ZSPAGES,     /* allocated in zsmalloc */

#endif

    NR_FREE_CMA_PAGES,

    NR_VM_ZONE_STAT_ITEMS };
```

讲完一般结构，这里需要注意，虽说我们的`node`节点中直接就是一个`zone`数组，但他们之间是有区别的，此在`/include/linux/mmzone.h`中有定义：
