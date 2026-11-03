
# [原创] 基于BYOVD方法无限使用SandboxiePlus的高级功能

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2037796
- 抓取时间: 2026-11-01T22:08:23

### 前言

随着SandboxiePlus版本的迭代，从1.15.10版本开始，原有无限试用其高级功能的方法已经不再适用。

![verify_certinfo_active.png](https://attach.52pojie.cn/forum/202506/10/160607mw4ktqxqechx4cex.png)

上图中，`proces.c`中的`Process_Create`函数在目标进程启动的时候校验了证书的`active`字段。如果`active`是0，则在目标进程启动五分钟后，kill掉目标进程。否则，进程正常执行，不进行kill。

原来的破解方法是基于证书有效期的字段全在用户态进行校验，修改用户态校验的结果，进而达到无限试用高级功能的目的。但从1.15.10版本开始出现了驱动侧(r0)验证证书的有效性，自然而然原来的破解方法就失效了。本文使用BYOVD的方法修改驱动侧的全局变量`Verify_CertInfo`，实现在没有证书的情况下也可以免费试用其高级功能。

### 原理介绍

BYOVD(Bring Your Own Vulnerable Driver)，即攻击者向目标环境植入一个带有漏洞的合法驱动程序，再通过漏洞利用获得内核权限以杀死/致盲终端安全软件等。这项技术最初主要被如Turla和方程式这样的顶级APT组织所使用，而随着攻击成本的降低，其它攻击组织也逐渐开始使用这项技术。

在[这篇文章](https://www.52pojie.cn/thread-2017673-1-1.html)中，我介绍了SandboxiePlus的验证过程。它通过验证证书的内容和签名，来判断当前证书的类型和有效期，最后将这些信息写入到`Verify_CertInfo`这个全局变量中。每次使用带有高级功能的沙盒时，会获取该变量的信息，来判断是否可以使用高级功能，不能使用高级功能则出现如下提示：

![redbox_5_minutes.png](https://attach.52pojie.cn/forum/202506/10/160605oxnummdsuwojzku8.png)

如果能够直接修改驱动侧`Verify_CertInfo`变量的值为一个有效期证书验证之后的结果，那么即便没有证书，我们也可以实现无限使用高级功能的效果。而是实现驱动侧修改`Verify_CertInfo`变量的方法就是使用一个带有写任意地址漏洞的签名驱动。

### 过程分析

为了保证使用者系统的安全性，在需要破解的时候创建漏洞驱动服务，马上启动该服务。在修改变量结束之后，马上停止该漏洞服务，并删除。这里使用的漏洞驱动是[这个驱动](https://www.loldrivers.io/drivers/afb8bb46-1d13-407d-9866-1daa7c82ca63/)，漏洞利用的Github地址是 https://github.com/kite03/echoac-poc/tree/main/PoC。

#### 获取Verify_CertInfo变量地址

SandboxiePlus的安装目录中存在`SbieDrv.sys`和`SbieDrv.pdb`文件，可以解析这个PDB文件获取`Verify_CertInfo`变量的偏移offset。然后获取`SbieDrv`驱动的基地址，然后两者之和就是`Verify_CertInfo`地址。在Windows系统中，所有进程共享内核地址空间，因此当前进程修改`Verify_CertInfo`地址就可以实现预期的效果。

- 借助`DbgHelp`库中的函数获取`Verify_CertInfo`变量的偏移。

```cpp
// 参数是sbiedrv驱动路径和sbiedrv pdb路径
uint64_t getVerifyCertInfoOffset(const wchar_t* sysPath, const wchar_t* pdbPath)
{
    uint64_t res = 0;

    SymInitialize(GetCurrentProcess(), NULL, FALSE);
    // load sys file
    uint64_t baseAddress = SymLoadModuleExW(GetCurrentProcess(), NULL, sysPath, NULL, 0x10000000, 0, NULL, 0);
    if (baseAddress == 0)
    {
            printf("[!] Error load sys file\n");
            return 0;
    }

    // load pdb file
    if (!SymSetSearchPathW(GetCurrentProcess(), pdbPath))
    {
            printf("[!] Error load pdb file\n");
            SymUnloadModule(GetCurrentProcess(), baseAddress);
            SymCleanup(GetCurrentProcess());
            return 0;
    }

    char symbolBuffer[sizeof(SYMBOL_INFOW) + 0x100 * sizeof(wchar_t)] = { 0 };
    PSYMBOL_INFOW pSymbol = (PSYMBOL_INFOW)symbolBuffer;
    pSymbol->SizeOfStruct = sizeof(SYMBOL_INFOW);
    pSymbol->MaxNameLen = 0xff;

    if (SymFromNameW(GetCurrentProcess(), L"Verify_CertInfo", pSymbol))
    {
            res = pSymbol->Address - baseAddress;
    }

    SymUnloadModule(GetCurrentProcess(), baseAddress);
    SymCleanup(GetCurrentProcess());
    return res;
}
```

- 遍历当前系统中的驱动模块，获取`SbieDrv`模块的基地址。

```cpp
// 这里需要使用未公开的系统调用参数SystemModuleInformation，对应的系统调用是NtQuerySystemInformation

uint64_t getSbieDrvBaseAddress()
{
    NTSTATUS status = STATUS_SUCCESS;
    uint64_t sbiedrvBaseAddress = 0;
    PRTL_PROCESS_MODULES moduleInfo = (PRTL_PROCESS_MODULES)VirtualAlloc(NULL, 0x400 * 0x400, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    if (moduleInfo == NULL)
    {
        printf("[!] Error allocating module memory! Error Code: %lu\n", GetLastError());
        return 0;
    }

    if(!NT_SUCCESS(status = NtQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,
            moduleInfo, 0x400 * 0x400, NULL)))
    {
        printf("[!] Error: Unable to query module list (%#x)\n", status);
        VirtualFree(moduleInfo, 0, MEM_RELEASE);
        return 0;
    }

    for (ULONG i = 0; i NumberOfModules; i++)
    {
        if (!_stricmp((const char*)moduleInfo->Modules[i].FullPathName +
                moduleInfo->Modules[i].OffsetToFileName, "sbiedrv.sys"))
        {
            sbiedrvBaseAddress = (uint64_t)moduleInfo->Modules[i].ImageBase;
            break;
        }
    }

    return sbiedrvBaseAddress;
}
```

#### 修改Verify_CertInfo变量的值

前文`echoac-poc`项目首先需要将代签名的漏洞驱动`echo_driver.sys`运行起来，然后获取驱动创建的设备的句柄，并设置当前进程。

```cpp
DriverInterface::DriverInterface()
{
    // 获取对应的设备句柄
    hDevice = CreateFileA(
        "\\\\.\\EchoDrv",
        GENERIC_READ | GENERIC_WRITE,
        FILE_SHARE_READ | FILE_SHARE_WRITE,
        NULL,
        OPEN_EXISTING,
        NULL,
        NULL
    );

    //If driver handle failed to open print message and return
    if (hDevice == INVALID_HANDLE_VALUE) {
        std::cout 
```

- 调用`read_memory_raw`函数将合适的值写入到`Verify_CertInfo`变量中。

```cpp
HANDLE processHandle = Driver.get_handle_for_pid(GetCurrentProcessId());

uint64_t newValue = 0xfffffffff000fcc1;

// 0x000c3550 f000fcc1 -> 0xffffffff f000fcc1，其中0x000c3550表示证书的有效期，修改为0xffffffff表示可以无限长时间使用

Driver.read_memory_raw((void *)&newValue, (void *)(SbieDrvBaseAddress + verifyCertInfoOffset), 8, processHandle);

// read_memory_raw 函数如下

BOOL read_memory_raw(void* address, void* buf, size_t len, HANDLE targetProcess) 
{
    k_param_readmem req{};
    req.fromAddress = (void*)address;
    req.length = len;
    req.targetProcess = targetProcess;
    req.toAddress = (void*)buf;
    BOOL success = DeviceIoControl(hDevice, 0x60a26124, &req, sizeof(k_param_readmem), &req, sizeof(k_param_readmem), NULL, NULL);
    return success;
}
```

`read_memory_raw`函数参数的意思是将`targetProcess`进程的`address`地址的`len`字节内容写入到当前进程`buf`地址中。这里`processHandle`我们设置成当前进程，表示从当前进程中读取`newValue`对应的值写入到`Verify_CertInfo`变量中，而`newValue`变量的值是一个合法试用期证书的值，但将时间调至了无限。

#### 什么时候修改

前面两个小结介绍了修改变量`Verify_CertInfo`的方法，即怎么修改。还有一个关键问题：什么时候修改？通过阅读SandboxiePlus的代码，发现了什么时候会触发`Verify_CertInfo`变量的修改：

- `SbieDrv.sys`驱动加载时，会校验证书的有效性。

```cpp
_FX NTSTATUS DriverEntry(
    IN  DRIVER_OBJECT  *DriverObject,
    IN  UNICODE_STRING *RegistryPath)
{
    ......
    if (ok)
        MyValidateCertificate();
    ......
}

_FX NTSTATUS MyValidateCertificate(void)
{
    if(!*g_uuid_str)
        InitFwUuid();

    NTSTATUS status = KphValidateCertificate();

    if (status == STATUS_ACCOUNT_EXPIRED)
        status = STATUS_SUCCESS;

    return status;
}
```

- 用户态的校验。`SbieDrv`模块的源码中有一个`Conf_Api_Reload`，当用户态调用该函数且带有`SBIE_CONF_FLAG_RELOAD_CERT`参数时，会重置`Verify_CertInfo`变量的值。而且调用这个参数的地方有多出。

```cpp
_FX NTSTATUS Conf_Api_Reload(PROCESS *proc, ULONG64 *parms)
{
    NTSTATUS status;
    ULONG flags;

    if (proc)
        return STATUS_NOT_IMPLEMENTED;

    flags = (ULONG)parms[2];

    if (flags & SBIE_CONF_FLAG_RELOAD_CERT) {
        status = MyValidateCertificate();
        goto finish;
    }

    ......
}
```

解决方法：

- 第一种触发情况，其实不必处理，因为SandboxiePlus必须依赖`SbieDrv`驱动。可以将第一种情况归类于第二种情况。

- 第二种触发情况，可以选择在用户态调用`Conf_Api_Reload`之后，执行一下我们的修改程序。幸运的是，调用`Conf_Api_Reload`的地方都在dll中，可以修改dll中的代码，然后编译替换对应的模块即可。这里修改的代码为`SbieAPI.cpp`文件中的`ReloadConf`函数，修改的函数如下：
