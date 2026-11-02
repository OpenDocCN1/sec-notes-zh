
# 内存分配代码解析

## `zone` 水位检查与回收

```c
         * (spread_dirty_pages unset) before going into reclaim,

         * which is important when on a NUMA setup the allowed

         * nodes are together not big enough to reach the

         * global limit.  The proper fix for these situations

         * will require awareness of nodes in the

         * dirty-throttling and the flusher threads.

         */

        if (ac->spread_dirty_pages) {

            if (last_pgdat_dirty_limit == zone->zone_pgdat)

                continue;

            if (!node_dirty_ok(zone->zone_pgdat)) {

                last_pgdat_dirty_limit = zone->zone_pgdat;

                continue;

            }

        }

        if (no_fallback && nr_online_nodes > 1 &&

            zone != ac->preferred_zoneref->zone) {

            int local_nid;

            /*

             * If moving to a remote node, retry but allow

             * fragmenting fallbacks. Locality is more important

             * than fragmentation avoidance.

             */

            local_nid = zone_to_nid(ac->preferred_zoneref->zone);

            if (zone_to_nid(zone) != local_nid) {

                alloc_flags &= ~ALLOC_NOFRAGMENT;

                goto retry;

            }

        }

        mark = wmark_pages(zone, alloc_flags & ALLOC_WMARK_MASK);

        if (!zone_watermark_fast(zone, order, mark,

                       ac->highest_zoneidx, alloc_flags,

                       gfp_mask)) {

            int ret;

#ifdef CONFIG_DEFERRED_STRUCT_PAGE_INIT

            /*

             * Watermark failed for this zone, but see if we can

             * grow this zone if it contains deferred pages.

             */

            if (static_branch_unlikely(&deferred_pages)) {

                if (_deferred_grow_zone(zone, order))

                    goto try_this_zone;

            }

#endif

            /* Checked here to keep the fast path fast */

            BUILD_BUG_ON(ALLOC_NO_WATERMARKS preferred_zoneref->zone, zone))

                continue;

            ret = node_reclaim(zone->zone_pgdat, gfp_mask, order);

            switch (ret) {

            case NODE_RECLAIM_NOSCAN:

                /* did not scan */

                continue;

            case NODE_RECLAIM_FULL:

                /* scanned but unreclaimable */

                continue;

            default:

                /* did we reclaim enough */

                if (zone_watermark_ok(zone, order, mark,

                    ac->highest_zoneidx, alloc_flags))

                    goto try_this_zone;

                continue;

            }

        }

try_this_zone:      //本zone正常水位

        page = rmqueue(ac->preferred_zoneref->zone, zone, order,

                gfp_mask, alloc_flags, ac->migratetype);

        if (page) {

            prep_new_page(page, order, gfp_mask, alloc_flags);

            /*

             * If this is a high-order atomic allocation then check

             * if the pageblock should be reserved for the future

             */

            if (unlikely(order && (alloc_flags & ALLOC_HARDER)))

                reserve_highatomic_pageblock(page, zone, order);

            return page;

        } else {

#ifdef CONFIG_DEFERRED_STRUCT_PAGE_INIT

            /* Try again if zone has deferred pages */

            if (static_branch_unlikely(&deferred_pages)) {

                if (_deferred_grow_zone(zone, order))

                    goto try_this_zone;

            }

#endif

        }

    }

    /*

     * It's possible on a UMA machine to get through all zones that are

     * fragmented. If avoiding fragmentation, reset and try again.

     */

    if (no_fallback) {

        alloc_flags &= ~ALLOC_NOFRAGMENT;

        goto retry;

    }

    return NULL;

}
```

其功能就是首先遍历当前的 `zone`，判断当前 `zone` 是否满足 low water mark 水位；若不满足则进行一次快速回收操作，再次检测水位情况；若还是不能满足，则遍历下一个 `zone`，然后采取同样的步骤，最后进入 `rmqueue()` 函数。这就是 buddy system 的核心，过程可以简化看下图：

![](http://imgsrc.baidu.com/forum/pic/item/11385343fbf2b211d43468878f8065380dd78ee5.jpg)

相比于代码，下图更加直观。之后我们来查看关键函数 `rmqueue()`，它位于 `/mm/page_alloc.c`。

## 关键函数 `rmqueue()`

```c
/*
 * 从所给zone中获取页. 当order为0的时候，使用pcplists.
 */
static inline
struct page *rmqueue(struct zone *preferred_zone,
            struct zone *zone, unsigned int order,
            gfp_t gfp_flags, unsigned int alloc_flags,
            int migratetype)
{
    unsigned long flags;
    struct page *page;

    if (likely(order == 0)) {
        /*
         * 若没有开启`CMA`|设置`ALLOC_CMA`|迁移类型为MIGRATE_MOVABLE，则先从pcplist上分配
         */
        if (!IS_ENABLED(CONFIG_CMA) || alloc_flags & ALLOC_CMA ||
                migratetype != MIGRATE_MOVABLE) {
            page = rmqueue_pcplist(preferred_zone, zone, gfp_flags,
                    migratetype, alloc_flags);
            goto out;
        }
    }

    /*
     * We most definitely don't want callers attempting to
     * allocate greater than order-1 page units with __GFP_NOFAIL.
     */
    WARN_ON_ONCE((gfp_flags & __GFP_NOFAIL) && (order > 1));
    spin_lock_irqsave(&zone->lock, flags);

    do {
        page = NULL;
        /*
         * order-0 request can reach here when the pcplist is skipped
         * due to non-CMA allocation context. HIGHATOMIC area is
         * reserved for high-order atomic allocation, so order-0
         * request should skip it.
         */
        if (order > 0 && alloc_flags & ALLOC_HARDER) { //order大于0且带有ALLOC_HARDER，使用__rmqueue_smallest分配
            page = __rmqueue_smallest(zone, order, MIGRATE_HIGHATOMIC);
            if (page)
                trace_mm_page_alloc_zone_locked(page, order, migratetype);
        }
        /*
         * 执行到这里说明order>0,我们采用__rmqueue函数，这是真正的兄弟系统核心分配函数
         */
        if (!page)
            page = __rmqueue(zone, order, migratetype, alloc_flags);
    } while (page && check_new_pages(page, order));
    spin_unlock(&zone->lock);
    if (!page)
        goto failed;
    __mod_zone_freepage_state(zone, -(1 flags)) {
        clear_bit(ZONE_BOOSTED_WATERMARK, &zone->flags);
        wakeup_kswapd(zone, 0, 0, zone_idx(zone));
    }

    VM_BUG_ON_PAGE(page && bad_range(zone, page), page);
    return page;

failed:
    local_irq_restore(flags);
    return NULL;
}
```

有部分注释，我在上面中西合璧标注了一下。接下来先提醒大家伙，之前咱们讲解 `zone` 上的一个字段 `per-cpu pageset`，他是为了放置条件竞争的问题，为每个 cpu 单独设置一个仓库用来为 buddy system 进行迅速的分配。这里就是给出了 buddy system 先从他里面调用的函数代码，总结为一下流程：

### `rmqueue()` 分配流程

- 若 `order` 为 0，若没有开启 `CMA` | 设置 `ALLOC_CMA` | 迁移类型为 `MIGRATE_MOVABLE`，则先从 `per-cpu pageset` 中分配并且返回
- `order > 0` 调用 `__rmqueue_smallest()` 分配
- 若未分配成功，这里不管 `order` 是否为 0，调用 `__rmqueue()` 分配
- 结果检查，调用 `check_new_pages()`，未通过则循环跳到第二步

### `rmqueue_pcplist()`

我们一个一个关键函数来查看，首先是分配 `per_cpu_pageset`，也就是如下函数 `rmqueue_pcplist()`：

```c
/* Lock and remove page from the per-cpu list */
static struct page *rmqueue_pcplist(struct zone *preferred_zone,

            struct zone *zone, gfp_t gfp_flags,

            int migratetype, unsigned int alloc_flags)

{

    struct per_cpu_pages *pcp;

    struct list_head *list;

    struct page *page;

    unsigned long flags;

    local_irq_save(flags); // 关中断

    pcp = &this_cpu_ptr(zone->pageset)->pcp;

    list = &pcp->lists[migratetype]; // 获取迁移类型链表

    page = __rmqueue_pcplist(zone,  migratetype, alloc_flags, pcp, list); // 分配

    if (page) {

        __count_zid_vm_events(PGALLOC, page_zonenum(page), 1);

        zone_statistics(preferred_zone, zone);

    }

    local_irq_restore(flags); // 开中断

    return page;

}
```

主要是进行了一些同步互斥操作（开关中断），然后调用函数 `__rmqueue_pcplist()`。
