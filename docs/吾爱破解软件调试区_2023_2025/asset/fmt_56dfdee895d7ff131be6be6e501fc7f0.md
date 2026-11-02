
![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2016-5195/CVE-2016-5195.4.excalidraw.png)

在返回之后重新 `retry`，进入 `follow_page_mask` 函数，但是漏洞已经出现了：

```
long __get_user_pages(struct task_struct *tsk, struct mm_struct *mm,
                unsigned long start, unsigned long nr_pages,
                unsigned int gup_flags, struct page **pages,
                struct vm_area_struct **vmas, int *nonblocking)
{
        ... ...
        ... ...
retry:
                ... ...
                cond_resched();
                page = follow_page_mask(vma, start, foll_flags, &page_mask);
                if (!page) {
                        int ret;
                        ret = faultin_page(tsk, vma, start, &foll_flags,
                                        nonblocking);
                        switch (ret) {
                        case 0:
                                goto retry;
                ... ...
                ... ...
}
```

进入 `retry` 流程后，`cond_resched` 函数会主动放权，导致 `madviseThread` 线程有一次出手机会。

### 页表释放

`madviseThread` 解除刚刚分配的页，这会导致刚刚分配的页失效，且由于刚刚清除了 `FOLL_WRITE` 标记位，接下来就是触发漏洞的时刻！

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2016-5195/CVE-2016-5195.5.excalidraw.png)

### 第三次页错误

调度返回 `__get_user_pages` 函数，重新进入 `follow_page_mask`，由于页表刚刚被释放，因此会触发第三次缺页错误，进入 `faultin_page` 函数中。

```
                cond_resched();
                page = follow_page_mask(vma, start, foll_flags, &page_mask);
                if (!page) {
                        int ret;
                        ret = faultin_page(tsk, vma, start, &foll_flags,
                                        nonblocking);
```

### 二次分配新页

在这次分配中，由于没有了写标记，因此 `fault_flags` 不会添加 `FAULT_FLAG_WRITE` 标记。此时的调用链为：

```
faultin_page ->
  handle_mm_fault ->
    __handle_mm_fault ->
      handle_pte_fault ->
        do_fault
```

在 `do_fault` 函数中，因为 `flags` 不带有 `FAULT_FLAG_WRITE`，因此最终会调用 `do_read_fault` 而不是 `do_cow_fault` 直接返回 page cache，因为内核会觉得你希望读而不是写，无所谓。

```
static int do_fault(struct mm_struct *mm, struct vm_area_struct *vma,
                unsigned long address, pte_t *page_table, pmd_t *pmd,
                unsigned int flags, pte_t orig_pte)
{
        ... ...
        ... ...
        if (!(flags & FAULT_FLAG_WRITE))
                return do_read_fault(mm, vma, address, pmd, pgoff, flags,
                                orig_pte);
        if (!(vma->vm_flags & VM_SHARED))
                return do_cow_fault(mm, vma, address, pmd, pgoff, flags,
                                orig_pte);
        return do_shared_fault(mm, vma, address, pmd, pgoff, flags, orig_pte);
}
```

这样，我们又双叒叕一次返回到了 `__get_user_pages` 函数中，再次调用 `follow_page_mask` 函数。不过这一次写标记已经去掉了，因此会正常给你返回 `page`，且该 `page` 是 page cache。

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2016-5195/CVE-2016-5195.6.excalidraw.png)

### 写入数据

终于，在经过长长的页分配之后，我们可以返回了，这次直接返回到 `__access_remote_vm` 函数中：

```
static int __access_remote_vm(struct task_struct *tsk, struct mm_struct *mm,
                unsigned long addr, void *buf, int len, int write)
{
        struct vm_area_struct *vma;
        void *old_buf = buf;
    ... ...
        while (len) {
                int bytes, ret, offset;
                void *maddr;
                struct page *page = NULL;

                ret = get_user_pages(tsk, mm, addr, 1,
                                write, 1, &page, &vma);
                if (ret mmap_sem);

        return buf - old_buf;
}
```

在返回页表项后，开始写入数据。由于 PoC 是直接写 `/proc/self/mem`，这种写法可以无视页表权限强制写入，这是 `memcpy` 做不到的。在这种写入方法下，会首先 `kmap` 出一块地址，写入并置脏页标记：

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2016-5195/CVE-2016-5195.7.excalidraw.png)

由于 pache cache 关联的页表项是脏页，因此最后利用 page cache 的写回机制复写磁盘上的文件，攻击完成。

![](https://alist.qrz.today/d/public/imghost/blog/2023-cve-2016-5195/CVE-2016-5195.8.excalidraw.png)

### 总结

这个漏洞通过 `write` `mmap` 出的只读私有内存触发页错误，去除写标记，利用条件竞争卸载当前列表，可以让本应报错的写入失效，能够分配出页表；然后利用 `/proc/self/mem` 的 `kmap` 强行写入，最后利用 page cache 的回写机制写回物理文件，达到修改只读文件的效果。

在这个漏洞中，竞争出现在第二次页错误去除写标记后，`cond_resched` 函数给了 `madvise` 插入的机会，让它能够释放页表项，导致漏洞的发生。

## 补丁分析

[git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=19be0eaffa3ac7d8eb6784ad9bdbc7d67ed8e619](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=19be0eaffa3ac7d8eb6784ad9bdbc7d67ed8e619) 添加了一个新的标志位 `FOLL_COW`：

```
diff --git a/include/linux/mm.h b/include/linux/mm.h
index e9caec6a51e97..ed85879f47f5f 100644
--- a/include/linux/mm.h
+++ b/include/linux/mm.h
@@ -2232,6 +2232,7 @@ static inline struct page *follow_page(struct vm_area_struct *vma,
 #define FOLL_TRIED        0x800        /* a retry, previous pass started an IO */
 #define FOLL_MLOCK        0x1000        /* lock present pages */
 #define FOLL_REMOTE        0x2000        /* we are working on non-current tsk/mm */
+#define FOLL_COW        0x4000        /* internal GUP flag */
 typedef int (*pte_fn_t)(pte_t *pte, pgtable_t token, unsigned long addr,
                         void *data);
```

在第二次页错误后，不会去除写标记而是添加 `FOLL_COW` 标记：
