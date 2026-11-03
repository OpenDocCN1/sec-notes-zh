
# [原创] Win11环境下使用BYOVD方法破解SandboxiePlus验证

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=2061773`
- 抓取时间: `2026-11-01T22:02:42`

前几天有评论说 `win11` 环境下之前的方法不行，试了一下确实不行。主要原因还是 `echo_driver.sys` 驱动加载失败了，返回的错误码是 `0x800B010C`，对应的含义如下：

![](https://attach.52pojie.cn/forum/202509/22/094026zmh5wxs0v1ezohms.png)

error_code.png (128.67 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgwNDQxMXxhMmU4MGYwOXwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNDQxMXxhMmU4MGYwOXwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes)

错误码

2025-9-22 09:40 上传

需要重新寻找新的具有任意地址读写漏洞的具有合法签名驱动文件。

经过一段时间的寻找，功夫不负有心人，终于找到了一个符合需求的项目 `BYOVD_read_write_primitive`，相关项目地址如下：[//github.com/0xJs/BYOVD_read_write_primitive" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/0xJs/BYOVD_read_write_primitive:rel="nofollow noopener" href="https) 。重新编写程序的过程中，发现一个有意思的问题：`win11` 环境下，管理员执行 `NtQuerySystemInformation` 获取 `SbieDrv` 驱动模块基地址的竟然是 `0`，而在 `win10` 环境下符合预期。

本篇文章的第一部分主要探究这个问题，第二部分给出基于 `BYOVD_read_write_primitive` 的破解思路。

## NtQuerySystemInformation 系统调用问题

破解程序使用该系统调用获取 `SbieDrv` 驱动模块的基地址，它的函数原型如下：

```
NTSTATUS NtQuerySystemInformation(
  [in]            SYSTEM_INFORMATION_CLASS SystemInformationClass,
  [in, out]       PVOID                    SystemInformation,
  [in]            ULONG                    SystemInformationLength,
  [out, optional] PULONG                   ReturnLength
);
```

第一个参数为 `SystemModuleInformation` (`11`) 时，获取当前所有驱动模块的信息，比较文件名即可获取 `SbieDrv` 驱动模块的基地址。

### Win10 环境

当 `NtQuerySystemInformation` 系统调用的第一个参数为 `SystemModuleInformation` 时，`NtQuerySystemInformation` 会调用 `ExpQuerySystemInformation` 函数，该函数相关的代码如下：

![](https://attach.52pojie.cn/forum/202509/22/094441kd0v1n71x1i2i4xd.png)

enum_module.png (25.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgwNDQxN3wzN2EzNTZiYXwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNDQxN3wzN2EzNTZiYXwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes)

函数代码

2025-9-22 09:44 上传

- `ExIsRestrictedCaller` 的 `IDA` `F5` 反汇编代码如下：

```
_BOOL8 __fastcall ExIsRestrictedCaller(char a1)
{

  BOOLEAN v1; // bl
  _BOOL8 result; // rax
  struct _SECURITY_SUBJECT_CONTEXT SubjectSecurityContext; // [rsp+50h] [rbp-28h] BYREF
  NTSTATUS AccessStatus; // [rsp+80h] [rbp+8h] BYREF
  ACCESS_MASK GrantedAccess; // [rsp+88h] [rbp+10h] BYREF
  AccessStatus = 0;

  GrantedAccess = 0;

  memset(&SubjectSecurityContext, 0, sizeof(SubjectSecurityContext));

  result = 0;

  if ( a1 )

  {

    SeCaptureSubjectContext(&SubjectSecurityContext);

    v1 = SeAccessCheck(

           SeMediumDaclSd,  // SecurityDescriptor

           &SubjectSecurityContext, // SubjectSecurityContext

           0,   // SubjectContextLocked

           0x20000u,    // DesiredAccess, READ_CONTROL -> 

           0,           // PreviouslyGrantedAccess

           0i64,        // Privileges

           (PGENERIC_MAPPING)&ExpRestrictedGenericMapping,  // GenericMapping

           1,   // AccessMode，KernelMode -> 0, UserMode -> 1

           &GrantedAccess,  // GrantedAccess

           &AccessStatus);  // AccessStatus

    SeReleaseSubjectContext(&SubjectSecurityContext);

    if ( !v1 || AccessStatus 
```

如果 `ExIsRestrictedCaller` 返回一个 `true`，则 `NtQuerySystemInformation` 系统调用返回一个 `0xC0000022`，表示 `STATUS_ACCESS_DENIED`。管理员启动一个进程的话，`ExIsRestrictedCaller` 返回一个 `False`，非管理员启动则返回一个 `True`。

- 接下来执行 `ExpQueryModuleInformation` 函数，

![](https://attach.52pojie.cn/forum/202509/22/094717v149qpz4sttptsil.png)

ExpQueryModuleInformation_win10_ida.png (84.36 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgwNDQyMnwxYzZlZTk0MHwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNDQyMnwxYzZlZTk0MHwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes)

IDA ExpQueryModuleInformation_win10

2025-9-22 09:47 上传

![](https://attach.52pojie.cn/forum/202509/22/094740b26qz5z655ht0kq2.png)

ExpQueryModuleInformation_win10_windbg.png (65.67 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgwNDQyM3wyNTkwMjgzOHwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNDQyM3wyNTkwMjgzOHwxNzkzNTcwNTYwfDE2ODA4NnwyMDYxNzcz&nothumb=yes)

windbg ExpQueryModuleInformation_win10

2025-9-22 09:47 上传

从 `PsLoadModuleList` 链表中遍历已经加载的内核模块，提取信息赋值给 `NtQuerySystemInformation` 的第二个参数 `SystemInformation`，它是一个 `PRTL_PROCESS_MODULES` 结构体。该结构体的内容如下：

```
typedef struct _RTL_PROCESS_MODULE_INFORMATION
{
    HANDLE Section;
    PVOID MappedBase;
    PVOID ImageBase;
    ULONG ImageSize;
    ULONG Flags;
    USHORT LoadOrderIndex;
    USHORT InitOrderIndex;
    USHORT LoadCount;
    USHORT OffsetToFileName;
    UCHAR FullPathName[256];
} RTL_PROCESS_MODULE_INFORMATION, * PRTL_PROCESS_MODULE_INFORMATION;
typedef struct _RTL_PROCESS_MODULES
{
    ULONG NumberOfModules;
    RTL_PROCESS_MODULE_INFORMATION Modules[1];
}
//
```

总结：在 `win10` 环境下，只要使用管理员启动进程来调用 `NtQuerySystemInformation` 系统调用即可获取所有内核模块的信息，包括基地址。

### Win11 环境

`Win11` 下 `NtQuerySystemInformation` 系统调用的执行和 `win10` 相似，主要在 `ExIsRestrictedCaller` 和 `ExpQueryModuleInformation` 函数中有些不同。

- `ExIsRestrictedCaller` 函数的不同，增加了是否拥有 `SeDebugPrivilege` 权限的验证。

```
__int64 __fastcall ExIsRestrictedCaller(KPROCESSOR_MODE a1, _DWORD *a2)
{

  unsigned int v2; // edi
  BOOLEAN v5; // bl
  struct _SECURITY_SUBJECT_CONTEXT SubjectContext; // [rsp+50h] [rbp-28h] BYREF
  NTSTATUS AccessStatus; // [rsp+80h] [rbp+8h] BYREF
  ACCESS_MASK GrantedAccess; // [rsp+88h] [rbp+10h] BYREF
  v2 = 0;

  AccessStatus = 0;

  GrantedAccess = 0;

  memset(&SubjectContext, 0, sizeof(SubjectContext));

  if ( a2 )
    *a2 = 0;

  if ( !a1 )
    return 0i64;

  // 不同之处，新增
  if ( a2 && (unsigned int)Feature_RestrictKernelAddressLeaks__private_IsEnabledDeviceUsageNoInline() )
    *a2 = SeSinglePrivilegeCheck(SeDebugPrivilege, a1) == 0;    

  SeCaptureSubjectContext(&SubjectContext);

  v5 = SeAccessCheck(

         SeMediumDaclSd,

         &SubjectContext,

         0,

         0x20000u,

         0,

         0i64,

         (PGENERIC_MAPPING)&ExpRestrictedGenericMapping,

         1,

         &GrantedAccess,

         &AccessStatus);

  SeReleaseSubjectContext(&SubjectContext);

  if ( !v5 )
    return 1i64;

  LOBYTE(v2) = AccessStatus 
```

如果拥有 `SeDebugPrivilege` 权限，将 `a2` 指针处设置为 `False`，否则为 `True`。从 `Feature_RestrictKernelAddressLeaks__private_IsEnabledDeviceUsageNoInline` 这个符号名也可以看出，此处是为了限制非 `SeDebugPrivilege` 权限的进程获取内核模块的基地址的。

- `ExpQueryModuleInformation` 函数的不同，`ExIsRestrictedCaller` 函数验证通过之后，会将 `a2` 指针处的值传递给 `ExpQueryModuleInformation` 函数的第一个参数。

```
__int64 __fastcall ExpQueryModuleInformation(int a1, _DWORD *a2, unsigned int a3, _DWORD *a4)
{
  __int64 result; // rax
  unsigned int v8; // ecx
  _QWORD v9[2]; // [rsp+30h] [rbp-38h] BYREF
  unsigned int v10; // [rsp+40h] [rbp-28h]
  int v11; // [rsp+44h] [rbp-24h]
  _DWORD *v12; // [rsp+48h] [rbp-20h]
  int v13; // [rsp+50h] [rbp-18h]
  int v14; // [rsp+54h] [rbp-14h]
  v9[0] = 0i64;
  v13 = a1;
  v14 = 0;
  v12 = a4;
  v11 = 8;
  v9[1] = a2 + 2;
  v10 = a3;
  // v9是一个结构体，
  result = MmEnumerateSystemImagesShared(ExpQueryModuleInformationImage, v9);   
  v8 = 0xC0000004;
  if ( (int)(result + 0x80000000) = 8 )
    {
      *a2 = v14;
      return LODWORD(v9[0]);
    }
    return v8;
  }
  return result;
}
__int64 __fastcall MmEnumerateSystemImagesShared(__int64 a1, __int64 a2)
{
  return MiEnumerateSystemImages(a1, a2, 2i64);
}
```

- `MiEnumerateSystemImages` 函数的反汇编代码如下：

```
 // a1 -> ExpQueryModuleInformationImage函数
// a2 -> ExpQueryModuleInformation的v9
__int64 __fastcall MiEnumerateSystemImages(__int64 a1, __int64 a2, __int64 a3)

{

  struct _KTHREAD *CurrentThread; // r9
  int v4; // edi
  unsigned int v5; // r12d
  __int64 Lock; // rax
  PVOID *v8; // rbx
  __int64 v9; // rbp
  __int64 v10; // r8
  __int64 v11; // r9
  PVOID *i; // rbx
  CurrentThread = KeGetCurrentThread();

  v4 = 0;

  v5 = a3;

  if ( (struct _KTHREAD *)qword_140E2D610 == CurrentThread )
  {
```
