
# 完整代码

```c
//完整代码
#include 
#include 

//0x78 bytes (sizeof)
typedef struct _LDR_DATA_TABLE_ENTRY
{
        LIST_ENTRY InLoadOrderLinks;                                    //0x0
        LIST_ENTRY InMemoryOrderLinks;                                  //0x8
        LIST_ENTRY InInitializationOrderLinks;                          //0x10
        VOID* DllBase;                                                  //0x18
        VOID* EntryPoint;                                               //0x1c
        ULONG SizeOfImage;                                              //0x20
        UNICODE_STRING FullDllName;                                     //0x24
        UNICODE_STRING BaseDllName;                                     //0x2c
        ULONG Flags;                                                    //0x34                              
}LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

//进程回调
VOID CreateProcessCallBack(
        _Inout_ PEPROCESS Process,
        _In_ HANDLE ProcessId,
        _Inout_opt_ PPS_CREATE_NOTIFY_INFO CreateInfo
        )
{
        if (CreateInfo)
        {
                DbgPrintEx(77, 0, "[进程回调]进程创建,进程id为%d,名字为:%wZ\n", ProcessId, CreateInfo->ImageFileName);
        }
        else
        {
                DbgPrintEx(77, 0, "[进程回调]进程销毁,进程id为%d\n", ProcessId);        
        }
}

//线程回调
VOID CreateThreadCallBack(
        _In_ HANDLE ProcessId,
        _In_ HANDLE ThreadId,
        _In_ BOOLEAN Create
        )
{
        if (Create)
        {
                DbgPrintEx(77, 0, "[线程回调]线程创建,进程id为%d,线程id为%d\n", ProcessId, ThreadId);
        }
        else
        {
                DbgPrintEx(77, 0, "[线程回调]线程销毁,进程id为%d,线程id为%d\n", ProcessId,ThreadId);
        }
}
//映像模块回调
VOID LoadImageCallBack(
_In_ PUNICODE_STRING FullImageName,
_In_ HANDLE ProcessId,                // pid into which image is being mapped
_In_ PIMAGE_INFO ImageInfo
)
{
        if (ImageInfo)
        {
                DbgPrintEx(77, 0, "[映像回调]映像加载,进程id为%d,映像类型值为%d,路径为%wZ\n", ProcessId, ImageInfo->SystemModeImage, FullImageName);

        }
        else
        {
                DbgPrintEx(77, 0, "[映像回调]映像销毁,进程id为%d\n", ProcessId);
        }
}

VOID UnloadDriver(PDRIVER_OBJECT pDriver)
{
        //移除进程回调
        PsSetCreateProcessNotifyRoutineEx(CreateProcessCallBack, TRUE);
        //移除线程回调
        PsRemoveCreateThreadNotifyRoutine(CreateThreadCallBack);
        //移除映像回调
        PsRemoveLoadImageNotifyRoutine(LoadImageCallBack);
}

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING pRegPath)
{
        NTSTATUS ntStatu = 0;
        //进程回调
        PLDR_DATA_TABLE_ENTRY pSection = pDriver->DriverSection;
        pSection->Flags |= 0x20;
        ntStatu = PsSetCreateProcessNotifyRoutineEx(CreateProcessCallBack, FALSE);
        DbgPrintEx(77, 0, "[db]进程回调注册状态值:%x\n", ntStatu);
        //线程回调
        ntStatu = PsSetCreateThreadNotifyRoutine(CreateThreadCallBack);
        DbgPrintEx(77, 0, "[db]线程回调注册状态值:%x\n", ntStatu);
        //映像回调
        ntStatu = PsSetLoadImageNotifyRoutine(LoadImageCallBack);
        DbgPrintEx(77, 0, "[db]映像回调注册状态值:%x\n", ntStatu);
        pDriver->DriverUnload = UnloadDriver;
        return STATUS_SUCCESS;
}
```

### 效果图

![](https://attach.52pojie.cn/forum/202304/08/223101hfb744rizubbob6k.png)

image-20230408222017223.png (144.93 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNTg4NHxhOTM3ZTRjOXwxNzkzNTczODI0fDE2ODA4NnwxNzcxNDA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNTg4NHxhOTM3ZTRjOXwxNzkzNTczODI0fDE2ODA4NnwxNzcxNDA0&nothumb=yes)

2023-4-8 22:31 上传

## 0x5: 实现禁止创建指定进程

### 首先要知道 `PsSetCreateProcessNotifyRoutineEx` 函数的触发时间

首先要知道 `PsSetCreateProcessNotifyRoutineEx` 函数的触发时间是在进程完全创建之前。具体来说，当系统正在创建新进程时，`PsSetCreateProcessNotifyRoutineEx` 注册的回调函数会在进程对象创建完成之后、但在用户空间初始化程序执行之前被调用。在这个时刻，进程的一些基本属性已经确定，如进程ID、父进程ID、映像名称等，但是进程的虚拟地址空间还未初始化，也没有执行用户空间的初始化程序。

### 其实我们上面说过回调函数原型解析

其实我们上面说过，回调函数原型中的第三个参数 `_Inout_opt_` `PPS_CREATE_NOTIFY_INFO` `CreateInfo` 中包含了创建进程的信息，其中有一个成员非常重要：`CreationStatus`，这个成员只有为 `STATUS_SUCCESS` 时才能让进程正常加载，所以我们只要让他为 `STATUS_UNSUCCESSFUL` 不就可以让他加载失败了吗？这里 `_Inout` 和 `P` 开头已经告诉了我们这个参数是个指针可以双向传递，所以我们直接 `CreateInfo->CreationStatus = STATUS_UNSUCCESSFUL` 就实现了，这里我举例禁止创建记事本进程为例子，在进程回调函数里过滤出只要创建的进程是记事本进程就不让它加载，核心代码如下：

```
VOID CreateProcessCallBack(

        _Inout_ PEPROCESS Process,

        _In_ HANDLE ProcessId,

        _Inout_opt_ PPS_CREATE_NOTIFY_INFO CreateInfo

        )

{

        if (CreateInfo)

        {

                DbgPrintEx(77, 0, "[进程回调]进程创建,进程id为%d,名字为:%wZ\n", ProcessId, CreateInfo->ImageFileName);

                UNICODE_STRING uniDstName = RTL_CONSTANT_STRING(L"\\??\\C:\\Windows\\System32\\notepad.exe");

                if (RtlEqualUnicodeString(&uniDstName, CreateInfo->ImageFileName, TRUE))

                {

                        CreateInfo->CreationStatus = STATUS_UNSUCCESSFUL;

                }

        }

        else

        {

                DbgPrintEx(77, 0, "[进程回调]进程销毁,进程id为%d\n", ProcessId);        

        }

}
```

### 效果

![](https://attach.52pojie.cn/forum/202304/08/223049zvwswkgwgv3nz3sl.png)

image-20230408222833973.png (99.57 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYwNTg4M3xjZGIyNjYzMXwxNzkzNTczODI0fDE2ODA4NnwxNzcxNDA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNTg4M3xjZGIyNjYzMXwxNzkzNTczODI0fDE2ODA4NnwxNzcxNDA0&nothumb=yes)

2023-4-8 22:30 上传

## 0x6: 总结

回调被很多杀软利用，看自己想的思路吧。
