
```c
if (IS_ENABLED(CONFIG_SLUB_TINY) || kmem_cache_debug(s)) {  //这里判断编译参数，以及对于kmem_cache的设置
    free_to_partial_list(s, slab, head, tail, cnt, addr);
    return;
}

do {
    if (unlikely(n)) {
        spin_unlock_irqrestore(&n->list_lock, flags);
        n = NULL;
    }
    prior = slab->freelist;
    counters = slab->counters;
    set_freepointer(s, tail, prior);    //将tail头插在slab->freelist上
    new.counters = counters;
    was_frozen = new.frozen;
    new.inuse -= cnt;
    if ((!new.inuse || !prior) && !was_frozen) {    //如果说（new中即将没有了使用的object或原slab的空闲块本身就为空）并且（该new没有被冻结）
        if (kmem_cache_has_cpu_partial(s) && !prior) {
            /*
             * Slab 之前没有出现在列表中，并且将部分为空。我们可以推迟列表移动而是冻结它。也就是链入cpu_slab
             */
            new.frozen = 1;
        } else { /* 需要从list中删除 */
            n = get_node(s, slab_nid(slab));
            /*
             * 推测性地获取list_lock。
             * 如果 cmpxchg 不成功，那么我们可以删除 list_lock 而不进行任何处理.
             *
             * 否则list_lock将与更新slab列表的其他处理器同步。
             */
            spin_lock_irqsave(&n->list_lock, flags);
        }
    }
} while (!cmpxchg_double_slab(s, slab,
    prior, counters,
    head, new.counters,
    "__slab_free"));        //这里的判断条件是只要满足slab->freelist == prior 并且满足slab->counters == counters ，然后将其分别赋值为head和new.counters,然后就继续运行

if (likely(!n)) {           //这里分空表示上面的do-while中没有运行到从list删除的部分
    if (likely(was_frozen)) {   //如果曾经被冻结
        /*
         * 未采取列表锁定，因此不需要列表活动。
         */
        stat(s, FREE_FROZEN);
    } else if (new.frozen) {
        /*
         * 如果我们刚刚冻结了该slab，我们就将他放入cpu的部分列表
         */
        put_cpu_partial(s, slab, 1);
        stat(s, CPU_PARTIAL_FREE);
    }
    return;
}

/*
 * 如果slab上的对象都是空闲的并且slab_alloc_node的部分列表数量大于最小的限制
 */
if (unlikely(!new.inuse && n->nr_partial >= s->min_partial))
    goto slab_empty;

/*
 * 留在slab中的objects。 如果之前不在partial list中，则添加它。
 */
if (!kmem_cache_has_cpu_partial(s) && unlikely(!prior)) {
    remove_full(s, n, slab);
    add_partial(n, slab, DEACTIVATE_TO_TAIL);
    stat(s, FREE_ADD_PARTIAL);
}

spin_unlock_irqrestore(&n->list_lock, flags);
return;

slab_empty:
if (prior) {    //prior不为空说明slab本来应该在partial链表上
    /*
     * Slab on the partial list.
     */
    remove_partial(n, slab);
    stat(s, FREE_REMOVE_PARTIAL);
} else {    //为空则说明在full链表
    /* Slab must be on the full list */
    remove_full(s, n, slab);
}

spin_unlock_irqrestore(&n->list_lock, flags);
stat(s, FREE_SLAB);
discard_slab(s, slab);  //释放这一张slab
}
```

## 5. `kfree` 相关

```c
/**
 * kfree - free previously allocated memory
 * @object: pointer returned by kmalloc() or kmem_cache_alloc()
 *
 * If object is NULL, no operation is performed.
 */
void kfree(const void *object)
{
    struct folio *folio;
    struct slab *slab;
    struct kmem_cache *s;

    trace_kfree(_RET_IP_, object);

    if (unlikely(ZERO_OR_NULL_PTR(object)))     //对于传入指针的检测
        return;

    folio = virt_to_folio(object);
    if (unlikely(!folio_test_slab(folio))) {
        free_large_kmalloc(folio, (void *)object);
        return;
    }

    slab = folio_slab(folio);
    s = slab->slab_cache;
    __kmem_cache_free(s, (void *)object, _RET_IP_);
}
```

### `virt_to_folio`

通过我们的虚拟地址来获得我们相应物理地址所对应的`page`结构体。

```c
static inline struct folio *virt_to_folio(const void *x)
{
    struct page *page = virt_to_page(x);
    return page_folio(page);
}
```

```c
/**
 * page_folio - Converts from page to folio.
 * @p: The page.
 *
 * 每个page都是folio的一部分。 不能在 NULL 指针上调用此函数。
 *
 * 上下文：@page 上不需要引用，也不需要锁定。 
 * 如果调用者不持有引用，则此调用可能会与作品集拆分竞争，
 * 因此在获得作品集上的引用后，应重新检查作品集是否仍包含此页面。
 * Return: 包含该page的folio结构体
 */
#define page_folio(p)       (_Generic((p),              \
    const struct page *:    (const struct folio *)_compound_head(p), \
    struct page *:      (struct folio *)_compound_head(p)))
```

下面是一段`folio`的介绍。

> Linux 中的每个物理地址都有一个 `struct page`。 结构页是一种相当弱的数据类型； 当页面实际上是尾页并且没有映射时，很容易查看（例如）页面->映射。 Folios 是分离 `struct page` 的某些角色的开始。 从概念上讲，folios 获取 `struct page` 的内容（尾页部分除外）并将它们移动到 `struct folio` 中。 这并不是补丁集实际要做的事情，因为我还不够受虐狂，无法一次性完成所有这些更改。
>
> 我们可以（而且应该）走得更远。 我们需要了解内存（当前）是如何分配和使用的。 映射到用户空间的任何内存都需要有脏位和锁定位，有引用计数和映射计数。
>
> 引用的 kernelnewbie，不知道他在说什么，初步判定是用来优化以往的`page`结构提出的新特性，这里是返回了`folio`结构体供我们`kfree`使用。

### `free_large_kmalloc`

大页面则使用伙伴系统的接口，将其直接释放至伙伴系统之中。

```c
void free_large_kmalloc(struct folio *folio, void *object)
{
    unsigned int order = folio_order(folio);
    if (WARN_ON_ONCE(order == 0))
        pr_warn_once("object pointer: 0x%p\n", object);
    kmemleak_free(object);
    kasan_kfree_large(object);
    kmsan_kfree_large(object);
    mod_lruvec_page_state(folio_page(folio, 0), NR_SLAB_UNRECLAIMABLE_B,
                  -(PAGE_SIZE
```

### `__kmem_cache_free`

调用了另一个函数。

```c
void __kmem_cache_free(struct kmem_cache *s, void *x, unsigned long caller)
{
    slab_free(s, virt_to_slab(x), x, NULL, &x, 1, caller);
}
```

### `slab_free`

这里也是直接调用了我们之前所讲述的较为关键的释放函数。

```c
static __fastpath_inline void slab_free(struct kmem_cache *s, struct slab *slab,
                      void *head, void *tail, void **p, int cnt,
                      unsigned long addr)
{
    memcg_slab_free_hook(s, slab, p, cnt);
    /*
     * With KASAN enabled slab_free_freelist_hook modifies the freelist
     * to remove objects, whose reuse must be delayed.
     */
    if (slab_free_freelist_hook(s, &head, &tail, &cnt))
        do_slab_free(s, slab, head, tail, cnt, addr);
}
```

### -1. 参考

- [arttnba3.cn](http://arttnba3.cn/)
- [www.kernel.org](https://www.kernel.org/)
- [kernelnewbies.org](https://kernelnewbies.org/)
- [lwn.net](https://lwn.net/)
