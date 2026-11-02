
```
/* 从 per-cpu 链表上取出 page, 调用者必须保护链表 */
static struct page *__rmqueue_pcplist(struct zone *zone, int migratetype,
            unsigned int alloc_flags,
            struct per_cpu_pages *pcp,
            struct list_head *list)
{
    struct page *page;

    do {
        if (list_empty(list)) { // list 是空的
            // 
            pcp->count += rmqueue_bulk(zone, 0,
                    READ_ONCE(pcp->batch), list,
                    migratetype, alloc_flags);
            if (unlikely(list_empty(list)))
                return NULL;
        }

        // 链表脱链
        page = list_first_entry(list, struct page, lru);
        list_del(&page->lru);
        pcp->count--;
    } while (check_new_pcp(page));

    return page;
}
```

这里先判定链表，若为空，则调用`rmqueue_bulk()`函数，从`zone`上拿到`pages`之后再进行unlink，而`rmqueue_bulk()`函数最终会调用`__rmqueue()`

```
/*

 * 为了高效率，从 buddy 分配器获得指定数量的元素, 

 * 所有的单个元素都在持有锁的情况下进行.  将其添加到提供的链表中.

 * 返回放置在 *list 链表上的 pages 数量.

 */

static int rmqueue_bulk(struct zone *zone, unsigned int order,

            unsigned long count, struct list_head *list,

            int migratetype, unsigned int alloc_flags)

{

    int i, alloced = 0;

    spin_lock(&zone->lock);

    for (i = 0; i lru, list);

        alloced++;

        if (is_migrate_cma(get_pcppage_migratetype(page)))

            __mod_zone_page_state(zone, NR_FREE_CMA_PAGES,

                          -(1 lock);

    return alloced;

}
```

### `__rmqueue_smallest()`

该函数就是由`order`对应的`free_area`中类型为migration type的链表上进行分配，如果不够则向高`order`处请求，由于这里都是以`2^order`来进行分配，因此如果说我`order`为1，且这里不够的话，我们就转而`order`为2的链表，将其中的块对半拆下到低`order`中，其中向更高`order`分配是通过循环和脱链完成，而拆高阶的`page`是通过`expand()`函数来进行的

```
/*
 * 对给定的 migrationtype 遍历 free lists 
 * 并从 freelists 上移除最小可用的页面
 */
static __always_inline
struct page *__rmqueue_smallest(struct zone *zone, unsigned int order,
                        int migratetype)
{
    unsigned int current_order;
    struct free_area *area;
    struct page *page;

    /* 在 preferred list 上寻找一个合适 size 的 page */
    for (current_order = order; current_order free_area[current_order]);
        page = get_page_from_free_area(area, migratetype);
        if (!page)
            continue;
        del_page_from_free_list(page, zone, current_order);
        expand(zone, page, order, current_order, migratetype);
        set_pcppage_migratetype(page, migratetype);
        return page;
    }

    return NULL;
}
```

而拆分函数`expand()`也比较简单

```
/*

 * 此处再分割的顺序对 IO subsystem 而言是十分重要的.

 * 请不要在有好的理由及回归测试前改变这个顺序。

 * 特别地，当大块的内存被分割，更小块（内存）被传递的顺序

 * 则由他们在该函数中被分割的顺序决定。

 * 根据实际测试，这是影响传递给IO子系统的 pages 顺序的主要因素，

 * 考虑到包含一个内存大块（由一系列小的分配作用）的 buddy system 的行为，

 * 这也是合理的。这种行为是 sglist 合并成功的关键因素。

 *

 * -- nyc

 */

static inline void expand(struct zone *zone, struct page *page,

    int low, int high, int migratetype)

{

    unsigned long size = 1  low) {

        high--;

        size >>= 1;

        VM_BUG_ON_PAGE(bad_range(zone, &page[size]), &page[size]);

        /*

         * 标记为 guard pages (或 page), 这将允许在 buddy 将被

         * 释放时合并回分配器.对应的页表项不会被创建，

         * pages 在 虚拟地址空间上仍将保持不存在。

         */

        if (set_page_guard(zone, &page[size], high, migratetype))

            continue;

        add_to_free_list(&page[size], zone, high, migratetype);

        set_buddy_order(&page[size], high);

    }

}
```

### `__rmqueue()`

最开始我以为这个才是最终函数，但其实他不是，他反而还会调用`__rmqueue_smallest()`

```
/*
 * 从 buddy allocator 上移除一个元素.
 * 在持有 zone->lock 时调用.
 */
static __always_inline struct page *
__rmqueue(struct zone *zone, unsigned int order, int migratetype,
                        unsigned int alloc_flags)
{
    struct page *page;

    if (IS_ENABLED(CONFIG_CMA)) {
        /*
         * 通过当半数空闲内存在 CMA 区域时从 CMA 中分配
         * 以平衡常规的与CMA区域的可迁移的分配。
         */
        if (alloc_flags & ALLOC_CMA &&
            zone_page_state(zone, NR_FREE_CMA_PAGES) >
            zone_page_state(zone, NR_FREE_PAGES) / 2) {
            page = __rmqueue_cma_fallback(zone, order);
            if (page)
                goto out;
        }
    }
retry:
    page = __rmqueue_smallest(zone, order, migratetype);
    if (unlikely(!page)) {
        if (alloc_flags & ALLOC_CMA)
            page = __rmqueue_cma_fallback(zone, order);

        if (!page && __rmqueue_fallback(zone, order, migratetype,
                                alloc_flags))
            goto retry;
    }
out:
    if (page)
        trace_mm_page_alloc_zone_locked(page, order, migratetype);
    return page;
}
```

整体快速分配可以看下面这张图

![](http://imgsrc.baidu.com/forum/pic/item/a6efce1b9d16fdfadde22b27f18f8c5495ee7b69.jpg)

我们了解完了快速分配，接下来就是慢速分配了，其中他的功能包括了内存碎片化的整理和回收，他的代码太长，我就也只贴一部分，如下：

```
static inline struct page *

__alloc_pages_slowpath(gfp_t gfp_mask, unsigned int order,

      struct alloc_context *ac)

{

  page = __alloc_pages_direct_compact(gfp_mask, order, 

      alloc_flags, ac,

      INIT_COMPACT_PRIORITY,

      &compact_result);

  ......

  page = __alloc_pages_direct_reclaim(gfp_mask, order, alloc_flags, ac, 

       &did_some_progress);

  ......

}
```

其中内存碎片化也即是利用到迁移的知识，这里有两个关键函数，其中之一就是`__alloc_pages_direct_compact`

```
static struct page *
__alloc_pages_direct_compact(gfp_t gfp_mask, unsigned int order,
  unsigned int alloc_flags, const struct alloc_context *ac,
  enum compact_priority prio, enum compact_result *compact_result)
{
 struct page *page;
 unsigned int noreclaim_flag;

 if (!order)
  return NULL;

 noreclaim_flag = memalloc_noreclaim_save();
 *compact_result = try_to_compact_pages(gfp_mask, order, alloc_flags, ac,
         prio);
 memalloc_noreclaim_restore(noreclaim_flag);

 if (*compact_result compact_blockskip_flush = false;
  compaction_defer_reset(zone, order, true);
  count_vm_event(COMPACTSUCCESS);
  return page;
 }

 count_vm_event(COMPACTFAIL);

 cond_resched();

 return NULL;
}
```

这里的函数也是迁移算法memory compaction的代码实现，该算法可以简化为下面的流程

![](http://imgsrc.baidu.com/forum/pic/item/242dd42a2834349b34c1834c8cea15ce37d3be30.jpg)

也就是分为两个链表，一个专门遍历空闲页，一个专门遍历使用页，注意这俩要分别维持链表，然后最后进行交换操作就实现了迁移过程，且记住这个迁移是需要`page`本身是允许的才行,

在完成上述迁移操作后会再次尝试快速分配，这里的碎片化整理还有其他方式，但是我这里暂不区深究，先记录个图等我哪天想起来了再探索

![](http://imgsrc.baidu.com/forum/pic/item/e4dde71190ef76c6cae717e2d816fdfaae5167c6.jpg)

而关于慢速分配还有个函数是`__alloc_pages_direct_reclaim()`，他的作用主要是回收，而不是碎片整理

最后来个整体分配页框的函数流程图

![](http://imgsrc.baidu.com/forum/pic/item/810a19d8bc3eb135a3e35b72e31ea8d3fc1f44d7.jpg)

2> 暂未完工

一天下来怎么硕呢，感觉都是几位师傅的博客一口一口的喂饭，虽说自己理解了大致过程，但是对于源码的解读还是太粗了，这个系列还有释放页框和slub算法的源码实现，slub算法我再上一篇博客中已经讲解了大致原理了哦，这里还差一部分，
