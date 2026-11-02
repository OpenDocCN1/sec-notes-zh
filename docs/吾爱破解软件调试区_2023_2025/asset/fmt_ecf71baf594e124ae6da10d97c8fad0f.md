
# [调试逆向] `NtCurrentPeb()` 的反调试

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2016256
- 抓取时间: 2026-11-01T21:52:23

## `NtCurrentPeb()` 的反调试

在 BUU 的 crackMe 中出现

```
if ( (NtCurrentPeb()->ProcessHeap & 0x70) != 0 )
      v12 = v10 + v11;
if ( (NtCurrentPeb()->NtGlobalFlag & 0x70) != 0 )
      v12 = v10 + v11;
if ( *&NtCurrentPeb()->BeingDebugged )
    {
      v10 = -83;
      v11 = 43;
    }
```

上述三段均为反调试指令

### `NtCurrentPeb`

根据官方文档 [`NtCurrentTeb()`](https://learn.microsoft.com/zh-cn/windows/win32/api/winnt/nf-winnt-ntcurrentteb)：

> `NtCurrentTeb()` 例程返回指向当前线程的线程环境块（[`TEB`](https://learn.microsoft.com/zh-cn/windows/desktop/api/winternl/ns-winternl-teb)）的指针。

语法

C++

```
_TEB * NtCurrentTeb();
```

返回值

指向当前线程的线程环境块的指针。

### `ProcessHeap`

官方文档 [`GetProcessHeaps()`](https://learn.microsoft.com/zh-cn/windows/win32/api/heapapi/nf-heapapi-getprocessheaps)：

> 注解

`GetProcessHeaps()` 函数获取调用进程的默认堆的句柄，以及通过在进程中的任何线程上调用 [`HeapCreate()`](https://learn.microsoft.com/zh-cn/windows/desktop/api/heapapi/nf-heapapi-heapcreate) 函数创建的任何其他专用堆的句柄。

`GetProcessHeaps()` 函数主要用于调试，因为函数检索到的某些专用堆可能是由进程中运行的其他代码创建的，并且可能会在 `GetProcessHeaps()` 返回后被销毁。销毁堆会使堆的句柄失效，而继续使用此类句柄可能会导致应用程序中出现未定义的行为。只应在调用进程的默认堆和进程创建和管理的专用堆上调用堆函数。

若要获取调用进程的进程堆的句柄，请使用 [`GetProcessHeap()`](https://learn.microsoft.com/zh-cn/windows/win32/api/heapapi/nf-heapapi-getprocessheap) 函数。

### 为什么能反调试

#### `BeingDebugged`

这个很好理解，从名字直接可以看出，官方文档是这么介绍的 [`PEB`](https://learn.microsoft.com/zh-cn/windows/win32/api/winternl/ns-winternl-peb)：

> `BeingDebugged`
>
> 指示当前是否正在调试指定的进程。但是，`PEB` 结构是一种内部操作系统结构，其布局将来可能会更改。最好改用 [`CheckRemoteDebuggerPresent()`](https://learn.microsoft.com/zh-cn/windows/desktop/api/debugapi/nf-debugapi-checkremotedebuggerpresent) 函数。

#### `ProcessHeap` && `NtGlobalFlag`

deepseek是这么说的：

> 调试器是否会影响程序的活动堆（如进程的默认堆或动态创建的私有堆），取决于调试环境是否启用了特定的堆调试机制。以下是关键分析：
>
> ---
>
> ### 1. 调试机制对堆的影响
>
> #### (1) 调试标志与堆行为
>
> - `NtGlobalFlag` 调试标志
>
>   若进程由调试器启动（而非附加），系统可能在进程环境块（`PEB`）中设置 `NtGlobalFlag` 的调试标志，例如：
>
>   - `FLG_HEAP_ENABLE_TAIL_CHECK` (`0x10`)：在堆块末尾添加保护字段（如 `0xABABABAB`），检测缓冲区溢出。
>   - `FLG_HEAP_ENABLE_FREE_CHECK` (`0x20`)：释放堆块时填充特定值（如 `0xFEEEFEEE`），检测释放后篡改。
>   - `FLG_HEAP_VALIDATE_PARAMETERS` (`0x40`)：严格校验堆操作的参数，触发异常。
>
>     这些标志会直接修改堆管理器的行为，导致堆块大小、布局和元数据发生变化。
>
> - 页堆（Page Heap）
>
>   若启用页堆（通过 `gflags /i program.exe +hpa` 或注册表），系统会为每个堆分配附加保护页（Guard Page），并隔离堆块：
>
>   - 完整页堆（Full Page Heap）：每个堆块独占一页内存，末尾附加不可访问的保护页，溢出会立即触发访问异常（`STATUS_ACCESS_VIOLATION`）。
>   - 结构变化：堆块的元数据和用户内存分离，调试时可精确定位越界写问题。
>
> #### (2) 调试填充模式
>
> - 调试模式下，堆管理器会填充特定模式以辅助诊断：
>   - 未初始化内存：填充 `0xBAADF00D`（标记未初始化的堆内存）。
>   - 已释放内存：填充 `0xFEEEFEEE`（标记释放后的堆块）。
>
>   这些模式会改变堆内存的实际内容，可能影响依赖未初始化数据的程序逻辑。
>
> ---
>
> ### 2. 调试器启动方式的影响
>
> #### (1) 调试器启动进程
>
> - 系统会在进程初始化阶段设置调试标志（如 `NtGlobalFlag`），直接影响默认堆（`GetProcessHeap()`）的创建。
> - 示例：若启用页堆，`ProcessHeap` 的结构会完全不同于非调试模式。
>
> #### (2) 调试器附加到运行中的进程
>
> - 附加操作本身通常不会修改已有堆的配置，但可通过调试器命令（如 `!heap`）或调用 `HeapSetInformation()` 动态启用调试功能。
> - 例外：若进程已启用调试堆（如通过 Application Verifier），附加调试器会观察到调试堆的行为。
>
> ---
>
> ### 3. 实际影响场景
>
> #### (1) 堆块大小与布局变化
>
> - 调试堆分配更大内存：
>   例如，请求 100 字节的堆块，调试模式下可能实际分配 132 字节（含调试字段和保护字节）。
> - 内存地址差异：
>   启用页堆时，堆块可能位于独立的虚拟内存页，导致指针地址与正常模式不同。
>
> #### (2) 异常与稳定性
>
> - 越界写触发崩溃：
>   调试堆的保护机制（如尾检查）会主动拦截缓冲区溢出，导致程序在调试时崩溃，而正常模式下可能静默失败。
> - 释放后使用（Use-After-Free）检测：
>   释放的堆块被填充 `0xFEEEFEEE`，若程序尝试读写，调试器可能捕获到异常。
>
> #### (3) 性能下降
>
> - 调试堆的额外检查和填充操作会显著降低内存分配/释放速度，尤其在频繁操作堆的程序中。
>
> ---
>
> 调试器可能影响活动堆，但需满足以下条件：
>
> 1. 调试器以启动（非附加）方式运行进程，触发 `NtGlobalFlag` 调试标志。
> 2. 显式启用堆调试功能（如页堆或 `HeapSetInformation()`）。
>
> 若未启用堆调试机制，仅附加调试器通常不会改变堆行为。开发者可通过检查 `NtGlobalFlag` 或观察内存模式确认调试堆是否生效。

可以看出，`ProcessHeap` 因为调试出现的变化就是因为

> 进程初始化阶段设置调试标志（如 `NtGlobalFlag`），直接影响默认堆（`GetProcessHeap()`）的创建。

而这个检测是否被触发和启动进程还是附加进程有关

那我就可以尝试对比一下 IDA 的附加调试是否会触发这两个判断

[![image.png](https://attach.52pojie.cn/forum/202503/18/221427xmaa70ya0t6hl85u.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MzExN3wwN2M1Y2NjZHwxNzkzNTY5OTQxfDE2ODA4NnwyMDE2MjU2&nothumb=yes)

image.png (37.11 KB, 下载次数: 2)

2025-3-18 22:14 上传

运行题目程序，打开 IDA，选择 Attach to process，选择程序的进程，打好断点

[![image 1.png](https://attach.52pojie.cn/forum/202503/18/221416rupabaaivudwvb00.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MzExMnwyMTBkMjUyOXwxNzkzNTY5OTQxfDE2ODA4NnwyMDE2MjU2&nothumb=yes)

image 1.png (134.38 KB, 下载次数: 1)

2025-3-18 22:14 上传

[![image 2.png](https://attach.52pojie.cn/forum/202503/18/221418akjjb8qjjsyiq8z8.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MzExM3wxNWNjMTc0M3wxNzkzNTY5OTQxfDE2ODA4NnwyMDE2MjU2&nothumb=yes)

image 2.png (95.68 KB, 下载次数: 2)

2025-3-18 22:14 上传

可以看到寄存器 `eax` 的值为 `0`，将会直接运行 63 行

（汇编知识：`TEST` 指令在两个操作数的对应位之间进行 `AND` 操作，当然 IDA 会直接闪烁显示跳转的箭头）

而如果我们直接运行程序的话则会执行 62 行

[![image 3.png](https://attach.52pojie.cn/forum/202503/18/221420ekg1ub1i68u1b2b6.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MzExNHwyYzFhMmE1ZnwxNzkzNTY5OTQxfDE2ODA4NnwyMDE2MjU2&nothumb=yes)

image 3.png (64.38 KB, 下载次数: 2)

2025-3-18 22:14 上传

`ProcessHeap` 也是同样操作，同样结果，大家可以试一试

[![image 4.png](https://attach.52pojie.cn/forum/202503/18/221423zv7uqh9h1u7c9hcm.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MzExNXw1MDUwNGU3ZHwxNzkzNTY5OTQxfDE2ODA4NnwyMDE2MjU2&nothumb=yes)

image 4.png (23.23 KB, 下载次数: 2)

2025-3-18 22:14 上传

当然，这个方法没法过掉 `BeingDebugged` 的检测

[![image 5.png](https://attach.52pojie.cn/forum/202503/18/221425c1v0s4ftnj595f9y.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MzExNnw2MzExMDQwM3wxNzkzNTY5OTQxfDE2ODA4NnwyMDE2MjU2&nothumb=yes)

image 5.png (54.54 KB, 下载次数: 1)

2025-3-18 22:14 上传
