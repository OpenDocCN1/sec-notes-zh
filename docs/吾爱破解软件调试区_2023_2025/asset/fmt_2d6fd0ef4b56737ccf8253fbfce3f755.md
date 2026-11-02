
# [系统底层] Win10 x64 APC的分析与玩法

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1743643
- 抓取时间：2026-11-01T21:57:45

## 引

这段时间没啥事干，就断断续把 Windows 的 APC机制给分析了一下，发现许多地方还是比较有趣的，例如：用户特殊APC一开始是插入到内核APC链表中的，然后再通过它的 `kernelroutine` 将APC插回用户APC链表。

还发现了我调试的这个win10版本在用户特殊APC执行的时候的存在的BUG（[BUG](#bug)）。

APC的作用我这里就略了，可以百度一下。

Windows 10版本（同以前文章的版本）：`10.0.17763` 版本 17763

废话少说，正文开始：

## 快速导航

（网络文档无法使用页内跳转，只能往下划来看了）

- [APC结构篇](#APC结构篇)
- [APC插入篇](#APC插入篇)
- [APC插入篇总结](#APC插入篇总结)
- [APC执行篇](#APC执行篇)
- [APC执行篇总结](#APC执行篇总结)
- [APC玩法篇](#APC玩法篇)
  1. [第一个玩法](#第一个玩法)
  2. [第二个玩法](#第二个玩法)

文档pdf版本和相关资料下载：

> 如果网络文档内的“跳转”无法使用，推荐阅读PDF版本

链接：[https://pan.baidu.com/s/1owtKjfL80f1WbQj4blIoKQ](https://pan.baidu.com/s/1owtKjfL80f1WbQj4blIoKQ)

提取码：`ICEY`

## APC结构

### `_KAPC_STATE`

```
ntdll!_KAPC_STATE

   +0x000 ApcListHead      : [2] _LIST_ENTRY

   +0x020 Process          : Ptr64 _KPROCESS

   +0x028 InProgressFlags  : UChar

   +0x028 KernelApcInProgress : Pos 0, 1 Bit

   +0x028 SpecialApcInProgress : Pos 1, 1 Bit

   +0x029 KernelApcPending : UChar

   +0x02a UserApcPendingAll : UChar

   +0x02a SpecialUserApcPending : Pos 0, 1 Bit

   +0x02a UserApcPending   : Pos 1, 1 Bit
```

`kthread.ApcState` 指向 `_KAPC_STATE`

`ApcListHead`：

`ApcListHead[0]` 指向 内核APC链表

`ApcListHead[1]` 指向 用户APC链表

当 `ApcListHead.Flink == ApcListHead.Blink == &ApcListHead.Flink` 时，APC链表为空

否则 `ApcListHead.Flink = & _KAPC.ApcListEntry`，`ApcListHead.Blink = & _KAPC.ApcListEntry`

`Process`：

线程所属或者所挂靠的进程

`InProgressFlags`：

是否有APC正在执行 :

第0位置1：内核APC正在执行

第1位置1：特殊APC正在执行

`KernelApcPending`：

是否有内核APC正在等待

`UserApcPendingAll`：

用户APC正在等待

第0位置1：用户特殊APC正在等待

第1位置1：用户普通APC正在等待

### `_KAPC`

```
ntdll!_KAPC
   +0x000 Type             : UChar
   +0x001 SpareByte0       : UChar
   +0x002 Size             : UChar
   +0x003 SpareByte1       : UChar
   +0x004 SpareLong0       : Uint4B
   +0x008 Thread           : Ptr64 _KTHREAD
   +0x010 ApcListEntry     : _LIST_ENTRY
   +0x020 KernelRoutine    : Ptr64     void 
   +0x028 RundownRoutine   : Ptr64     void 
   +0x030 NormalRoutine    : Ptr64     void 
   +0x020 Reserved         : [3] Ptr64 Void
   +0x038 NormalContext    : Ptr64 Void
   +0x040 SystemArgument1  : Ptr64 Void
   +0x048 SystemArgument2  : Ptr64 Void
   +0x050 ApcStateIndex    : Char
   +0x051 ApcMode          : Char
   +0x052 Inserted         : UChar
```

- `Type` 和 `Size`：内核层对象必须存在的常量值。
- `Thread`：目标线程内核对象。
- `ApcListEntry`：APC链表
- `KernelRoutine`：无论那种APC，都会先执行这个函数（以内核的身份）
- `RundownRoutine`：如果插入APC失败，则会调用这个函数（详情查看 APC插入 篇）
- `NormalRoutine`：我们想让线程执行的函数（内核函数 或 用户函数）
- `NormalContext`：第0个参数。
- `SystemArgument1`：第1个参数
- `SystemArgument2`：第2个参数
- `ApcMode`：APC的类型：0内核APC 1用户APC
- `Inserted`：一个布尔标志，指示 APC 是否已插入。

## APC插入

### R3

我们就不讨论`QueueUserAPC`这个函数了，因为他最后是调用`NtQueueApcThread`进入R0的。

`QueueUserAPC`调用栈如下：

![](https://attach.52pojie.cn/forum/202302/08/232959xav8jqf22dd82kqk.png)

QueueUserAPC调用栈.png (23.59 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4OTQ3Mnw1NmI4NDRiOHwxNzkzNTcwMjYzfDE2ODA4NnwxNzQzNjQz&nothumb=yes)

2023-2-8 23:29 上传

所以呀，我们直接干R0的`NtQueueApcThread`就行啦！！

### R0

```
NTSTATUS __stdcall NtQueueApcThread(
        HANDLE ThreadHandle,
        PKNORMAL_ROUTINE ApcRoutine,
        PVOID NormalContext,
        PVOID SystemArgument1,
        PVOID SystemArgument2)
{
  return NtQueueApcThreadEx(ThreadHandle, 0i64, ApcRoutine, NormalContext, SystemArgument1, SystemArgument2);//通过NtQueueApcThread申请的APC全是用户普通APC
}
```

我们可以发现！`NtQueueApcThread`里面只是调用了`NtQueueApcThreadEx`。

说明`NtQueueApcThread`就是一个被阉割功能的函数了hhhh。

注意1：当用户APC触发时，返回的一定是R3的`ntdll!KiUserApcDispatcher`，然后`ntdll!KiUserApcDispatcher`再调用`ApcRoutine`。

注意2：`QueueUserApc`在插入APC调用`NtQueueApcThread`时，参数`ApcRoutine`是`ntdll!RtlDispatchAPC`，并不是我们提供给`QueueUserApc`的函数指针。我们的函数指针需要由`ntdll!RtlDispatchAPC`再次分发执行。

及 `NtQueueApcThread`的第二个参数为`ntdll!RtlDispatchAPC`，第三个参数才是我们我们提供给`QueueUserApc`的函数指针。

`QueueUserApc`简直就是`NtQueueApcThreadEx`的二次阉割函数。（墙裂谴责）

我们继续分析`NtQueueApcThreadEx`：

#### `NtQueueApcThreadEx`
