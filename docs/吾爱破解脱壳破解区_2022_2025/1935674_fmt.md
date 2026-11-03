
# [原创] 关于64位进程注入32位进程的分析

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1935674`
- 抓取时间: `2026-11-01T22:28:42`

故事开始于有人在我的项目中提了`issue`，也是我注册 `github` 来收到的第一个`issue`，因此我也非常重视。

## 前言

`issue` 的内容提到了，我的项目 `Xprocess` 注入器，没有办法实现注入 32 位进程的操作。他也给出了出错的原因，我没有在代码中获取远端的 `LoadLibraryW` 函数的地址。我一开始会以为很简单，网上应该有很多的实现，但是事实上，居然很难找到现成的代码。

## 解决思路

常见的方法可以获取目标模块的 `kernel32.dll` 的地址然后获取到 `LoadLibraryW` 函数的地址，但是遍历模块发现 64 位的程序无法使用 `HANDLE ths = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE,PID);` 的方式去获得模块的地址，因为只能获取到 `ntdll` 和 `Wow` 开头的几个模块，而在 32 位下编译就可以使用这个 API 获取到真实的模块地址，但是这与我们的目标不符，因此不考虑。

后面搜搜找找找到了一个可以用的 API 是 `EnumProcessModulesEx`。它能够获取 32 位进程远程模块加载的基地址。获取了基地址之后我又想了很久想怎么找到 `LoadLibraryW`。最初的一个想法是希望 `ntdll` 中存在函数 `GetProcAddress`，然后先通过一个远线程调用得到返回之后，等待线程返回就可以找到这个函数的地址了。可惜现实给了我当头一棒，它也在 `kernel32.dll` 里导出的。

最后我找到了一篇手动实现 `GetProcAddress` 的帖子[1]，于是有了一个灵感，将这个手动实现 `GetProcAddress` 去实现，然后替换为远程版本的。

## 实现过程

首先确定这个代码是可运行且无误的。

```c
DWORD MyGetProcAddress(
    HMODULE hModule,    // handle to DLL module
    LPCSTR lpProcName   // function name
)
{

    int i=0;
    PIMAGE_DOS_HEADER pImageDosHeader = NULL;
    PIMAGE_NT_HEADERS pImageNtHeader = NULL;
    PIMAGE_EXPORT_DIRECTORY pImageExportDirectory = NULL;

    pImageDosHeader=(PIMAGE_DOS_HEADER)hModule;
    pImageNtHeader=(PIMAGE_NT_HEADERS)((DWORD)hModule+pImageDosHeader->e_lfanew);
    pImageExportDirectory=(PIMAGE_EXPORT_DIRECTORY)((DWORD)hModule+pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);

    DWORD *pAddressOfFunction = (DWORD*)(pImageExportDirectory->AddressOfFunctions + (DWORD)hModule);
    DWORD *pAddressOfNames = (DWORD*)(pImageExportDirectory->AddressOfNames + (DWORD)hModule);
    DWORD dwNumberOfNames = (DWORD)(pImageExportDirectory->NumberOfNames);
    DWORD dwBase = (DWORD)(pImageExportDirectory->Base);

    WORD *pAddressOfNameOrdinals = (WORD*)(pImageExportDirectory->AddressOfNameOrdinals + (DWORD)hModule);
    DWORD dwName = (DWORD)lpProcName;
    if ((dwName & 0xFFFF0000) == 0)
    {
        goto xuhao;
    }
    for (i=0; i dwBase + pImageExportDirectory->NumberOfFunctions - 1)
    {
        return 0;
    }
    return (pAddressOfFunction[dwName - dwBase] + (DWORD)hModule);
}
```

这个 `hModule` 其实就是当前模块的地址。可以发现它采用解析 PE 文件的方式去遍历模块的导出表。

本地的实现了下一步就是实现远程的版本，这里需要非常仔细地去研究每一个访存的位置，因为在这个代码里一个简简单单的变量访问很有可能在远程版本中就需要通过 `ReadProcessMemory` 来实现。

下面我给出我写好的结果（只适配了32位的，64位的需要改一下 NT 头结构体）：

```c
FARPROC GetRemoteProcAddress(HANDLE hProcess, HMODULE hModule, LPCSTR lpProcName) {

    BYTE buffer[4096];

    SIZE_T bytesRead;

    if (!ReadProcessMemory(hProcess, hModule, buffer, sizeof(buffer), &bytesRead)) {

        return NULL;

    }

    PIMAGE_DOS_HEADER dosHeader = (PIMAGE_DOS_HEADER)buffer;

    PIMAGE_NT_HEADERS32 ntHeaders = (PIMAGE_NT_HEADERS32)((BYTE*)buffer + dosHeader->e_lfanew);

    DWORD RVAForExpDir = ntHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;

    if (!ReadProcessMemory(hProcess, (BYTE*)hModule + RVAForExpDir, buffer, sizeof(IMAGE_EXPORT_DIRECTORY), &bytesRead)) {

        return NULL;

    }

    PIMAGE_EXPORT_DIRECTORY exportDir = (PIMAGE_EXPORT_DIRECTORY)buffer ;

    DWORD funcAddr = (DWORD)( exportDir->AddressOfFunctions);

    DWORD nameAddr = (DWORD)( exportDir->AddressOfNames);

    DWORD nameOrdAddr = (DWORD)( exportDir->AddressOfNameOrdinals);

    for (DWORD i = 0; i NumberOfNames; i++) {

        char name[256];

        DWORD TrueNameAddr;

        WORD TrueOrd;

        DWORD TrueFuncAddr;

        if (!ReadProcessMemory(hProcess, (BYTE*)hModule + nameAddr + sizeof(DWORD)*i, &TrueNameAddr, sizeof(TrueNameAddr), &bytesRead)) {

            return NULL;

        }

        if (!ReadProcessMemory(hProcess, (LPCVOID)((BYTE*)hModule + (DWORD)TrueNameAddr), name, sizeof(name), &bytesRead)) {

            return NULL;

        }

        if (stricmp(name, lpProcName) == 0) {

            DWORD LoadLibraryAddr = 0;

            if (!ReadProcessMemory(hProcess, (BYTE*)hModule + nameOrdAddr + sizeof(WORD)*i, &TrueOrd, sizeof(TrueOrd), &bytesRead)) {

                return NULL;

            }

            if (!ReadProcessMemory(hProcess, (BYTE*)hModule + funcAddr + sizeof(DWORD)*(TrueOrd), &TrueFuncAddr, sizeof(TrueFuncAddr), &bytesRead)) {

                return NULL;

            }

            return (FARPROC)(TrueFuncAddr + (BYTE*)hModule);

        }

    }

    return NULL;

}
```

最后再判断注入的目标进程是不是 32 位的来选择合适的获取地址的方式去注入，最后实现也非常成功。

`https://github.com/xia0ji233/Xprocess/issues/2`
`https://github.com/xia0ji233/Xprocess/commit/cb091a4347fdcc3a1d812b1740f6a14d214b7888`

特此分享一下本次的经历，也给各位师傅们一个 64 位注入 32 位进程的参考案例。

## 参考文献

- [1] `https://cloud.tencent.com/developer/article/1471341`
