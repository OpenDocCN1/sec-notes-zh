
```c
kprintf(("Line %d:xia0ji233: RegistryPath = %S\n"), __LINE__, RegistryPath->Buffer);

target = MmCopyMemory;

kprintf(("Line %d:xia0ji233: MmCopyMemory=%p\n"), __LINE__, target);

g_Object = DriverObject;

if (target) {
    for (int i = 0; i
```

结果直接没有调用过。后面还同样的方法 `hook` 了其它的 `API`，诸如 `KeStackAttachProcess` 的，同样没有调用，于是陷入沉思，既然它没有把 `token` 写到 `r3` 那大概率在 `r0` 层。

同时在一次巧合中（开了 `DbgView` 的 `verbose`），发现 `token` 直接打印出来了。

![](https://attach.52pojie.cn/forum/202404/19/154123hkg5gcc0cz7e730s.png)

3.png (96.98 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjY5MDQxMnw2MmViMmQ0NXwxNzkzNTcyMzg2fDE2ODA4NnwxOTEzNjIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQxMnw2MmViMmQ0NXwxNzkzNTcyMzg2fDE2ODA4NnwxOTEzNjIz&nothumb=yes)

2024-4-19 15:41 上传

那么 `token2` 就出来了 `token2`: 8b3f14a24d64f3e697957c252e3a5686

所以 `flag` 就是 `flag{757F4749AEBB1891EF5AC2A9B5439CEA-8b3f14a24d64f3e697957c252e3a5686}`

## 5> 编写程序，运行时修改尽量少的内存，让两段token输出成功。（满分2分）

首先看看内核层的输出吧，因为它本来就可以输出，直接调用的 `DbgPrintEx` 函数，只不过某个 `Level` 无法正常被接受罢了，尝试 `hook` 一下，看看 `Level`

```c
#include 
#include 
#include 
#include 
#define MAX_BACKTRACE_DEPTH 20
#define SYMBOL L"\\??\\xia0ji2333"
#define kprintf(format, ...) DbgPrintEx(DPFLTR_IHVDRIVER_ID, DPFLTR_ERROR_LEVEL, format, ##__VA_ARGS__)

UINT64 BaseAddr=NULL, DLLSize=0;

void DeleteDevice(PDRIVER_OBJECT pDriver) {
    kprintf(("Line %d:xia0ji233: start delete device\n"), __LINE__);
    if (pDriver->DeviceObject) {
        UNICODE_STRING Sym;
        RtlInitUnicodeString(&Sym, SYMBOL);//CreateFile 
        kprintf(("Line %d:xia0ji233: Delete Symbol\n"), __LINE__);
        IoDeleteSymbolicLink(&Sym);
        kprintf(("Line %d:xia0ji233: Delete Device\n"), __LINE__);
        IoDeleteDevice(pDriver->DeviceObject);
    }
    kprintf(("Line %d:xia0ji233: end delete device\n"), __LINE__);
}

char newcode[] = {
    0x48,0xB8,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,//mov rax,xxx
    0xFF,0xE0//jmp rax
};
char oldcode[] = {
    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
    0x00,0x00,
};

char *target;

KIRQL WPOFFx64()
{
    KIRQL irql = KeRaiseIrqlToDpcLevel();
    UINT64 cr0 = __readcr0();
    cr0 &= 0xfffffffffffeffff;
    __writecr0(cr0);
    _disable();
    return irql;
}

void WPONx64(KIRQL irql)
{
    UINT64 cr0 = __readcr0();
    cr0 |= 0x10000;
    _enable();
    __writecr0(cr0);
    KeLowerIrql(irql);
}

NTSTATUS Unhook() {
    KIRQL irql = WPOFFx64();
    for (int i = 0; i DriverSection;
    PLIST_ENTRY LinkList;
    ;
    int i = 0;
    LinkList = TE->InLoadOrderLinks.Flink;  
    while (LinkList != &TE->InLoadOrderLinks)
    {
        Tmp = (LDR_DATA_TABLE_ENTRY*)LinkList;
        if (RtlCompareUnicodeString(&Tmp->BaseDllName, name, FALSE))
        {
        }
        else
        {
            kprintf(("Found Module!\n"));
            *pBaseAddr = (UINT64)(Tmp->DllBase);
            *pSize = (UINT64)(Tmp->SizeOfImage);
        }
        LinkList = LinkList->Flink;
        i++;
    }

}

typedef ULONG(*FuncPtr) (ULONG ComponentId,ULONG Level, PCSTR Format, ...);

ULONG  myDbgPrintEx( ULONG ComponentId,ULONG Level,PCSTR Format, ... ) {

    Unhook();
    FuncPtr func = (FuncPtr)target;
    kprintf(("call DbgPrintEx(%p,%p,%p)"), ComponentId, Level, Format);
    DbgBreakPoint();
    NTSTATUS s = func(ComponentId,Level,Format);

    Hook();

    return s;
}

void DriverUnload(PDRIVER_OBJECT pDriver) {
    kprintf(("Line %d:xia0ji233: start unload\n"), __LINE__);
    Unhook();
    DeleteDevice(pDriver);
}

NTSTATUS DriverEntry(
    _In_ PDRIVER_OBJECT DriverObject, _In_ PUNICODE_STRING RegistryPath
) {
    DriverObject->DriverUnload = DriverUnload;
    kprintf(("Line %d:xia0ji233: RegistryPath = %S\n"), __LINE__, RegistryPath->Buffer);
    target = DbgPrintEx;
    kprintf(("Line %d:xia0ji233: DbgPrintEx=%p\n"), __LINE__, target);
    g_Object = DriverObject;
    if (target) {
        for (int i = 0; i
```

持续输出了 `call DbgPrintEx(0000000000000000,0000000000000005,FFFFF1001067EB90) Level=5` 几乎不能输出任何内容了，因此尝试 `hook` 替换让它可以输出，但是这里 `Hook` 还是太麻烦了，于是我选择打下断点之后栈回溯一下看看情况

![](https://attach.52pojie.cn/forum/202404/19/154125fy8uyz5iaonsnyi6.png)

4.png (258.94 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY5MDQxM3wyMmZkNzZiOHwxNzkzNTcyMzg2fDE2ODA4NnwxOTEzNjIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQxM3wyMmZkNzZiOHwxNzkzNTcyMzg2fDE2ODA4NnwxOTEzNjIz&nothumb=yes)

2024-4-19 15:41 上传

可以发现关键 `call` 之前，有对 `edx` 赋值为 `5`，那么直接修改这个指令，把 `hook` 解掉看看能否输出。

这里我选择手动改一下，一共发现了三个位置，特征都是差不多的，都把 `mov edx,5` 改成 `mov edx,0`。

![](https://attach.52pojie.cn/forum/202404/19/154127u9mgh4sxupuso1l7.png)

5.png (260.32 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjY5MDQxNHwzOWJlMzg3NnwxNzkzNTcyMzg2fDE2ODA4NnwxOTEzNjIz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQxNHwzOWJlMzg3NnwxNzkzNTcyMzg2fDE2ODA4NnwxOTEzNjIz&nothumb=yes)

2024-4-19 15:41 上传

发现改完之后 `token` 成功输出了。

但是这里应该是驱动加载时候分配的内存写入的 `shellcode`，如果能知道地址，那么能去改掉这些指令，但是地址是我通过栈回溯找到的，如果没有 `hook` 掉系统 `API`，那么我根本不太可能去获取到 `shellcode` 的地址。于是想到它既然是不停地在打印的，必然创建了一个内核线程，那么我先遍历一下内核线程。

```c
#include 

#define kprintf(format, ...) DbgPrintEx(DPFLTR_IHVDRIVER_ID, DPFLTR_ERROR_LEVEL, format, ##__VA_ARGS__)

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath);

VOID UnloadDriver(PDRIVER_OBJECT DriverObject);

NTSTATUS EnumerateKernelThreads();

typedef NTSTATUS (*ZWQUERYSYSTEMINFORMATION)(ULONG, PVOID, ULONG, PULONG);

typedef struct _SYSTEM_PROCESS_INFORMATION {

    ULONG NextEntryOffset;

    ULONG NumberOfThreads;

    LARGE_INTEGER Reserved[3];

    LARGE_INTEGER CreateTime;

    LARGE_INTEGER UserTime;

    LARGE_INTEGER KernelTime;

    UNICODE_STRING ImageName;

    ULONG BasePriority;

    HANDLE ProcessId;

    HANDLE InheritedFromProcessId;

} SYSTEM_PROCESS_INFORMATION, *PSYSTEM_PROCESS_INFORMATION;

typedef struct _SYSTEM_THREAD_INFORMATION {

    LARGE_INTEGER KernelTime;

    LARGE_INTEGER UserTime;

    LARGE_INTEGER CreateTime;

    ULONG WaitTime;

    PVOID StartAddress;

    CLIENT_ID ClientId;

    ULONG Priority;

    LONG BasePriority;

    ULONG ContextSwitchCount;

    LONG State;

    LONG WaitReason;

} SYSTEM_THREAD_INFORMATION, *PSYSTEM_THREAD_INFORMATION;

typedef enum _SYSTEM_INFORMATION_CLASS {

    SystemProcessInformation = 5

} SYSTEM_INFORMATION_CLASS;

#define SystemModuleInformation 11

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {

    UNREFERENCED_PARAMETER(RegistryPath);

    DriverObject->DriverUnload = UnloadDriver;

    kprintf(("Driver Loaded\n"));

    return EnumerateKernelThreads();

}

VOID UnloadDriver(PDRIVER_OBJECT DriverObject) {

    UNREFERENCED_PARAMETER(DriverObject);

    kprintf(("Driver Unloaded\n"));

}

NTSTATUS EnumerateKernelThreads() {

    UNICODE_STRING routineName;

    RtlInitUnicodeString(&routineName, L"ZwQuerySystemInformation");

    ZWQUERYSYSTEMINFORMATION ZwQuerySystemInformation = (ZWQUERYSYSTEMINFORMATION)MmGetSystemRoutineAddress(&routineName);

    if (!ZwQuerySystemInformation) {

        return STATUS_UNSUCCESSFUL;

    }

    ULONG returnLength = 0;

    ZwQuerySystemInformation(SystemProcessInformation, NULL, 0, &returnLength);

    PVOID buffer = ExAllocatePool(NonPagedPool, returnLength);

    if (!buffer) {

        return STATUS_INSUFFICIENT_RESOURCES;

    }

    NTSTATUS status = ZwQuerySystemInformation(SystemProcessInformation, buffer, returnLength, &returnLength);

    if (!NT_SUCCESS(status)) {

        ExFreePool(buffer);

        return status;

    }

    PSYSTEM_PROCESS_INFORMATION current = (PSYSTEM_PROCESS_INFORMATION)buffer;

    while (TRUE) {

        PSYSTEM_THREAD_INFORMATION threadInfo = (PSYSTEM_THREAD_INFORMATION)(current + 1);

        for (ULONG i = 0; i NumberOfThreads; i++) {

            if (((UINT64)(threadInfo->StartAddress) & 0xFFFF000000000000) == 0xFFFF000000000000) {

                 kprintf(("Thread StartAddress: %p\n"), threadInfo->StartAddress);

            }
```
