
# [CTF] 2023腾讯游戏安全PC决赛复现

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1954225
- 抓取时间: 2026-11-01T22:26:16

## 前言

最近一直在做往年腾讯游戏安全PC端的复现，但是2023年决赛的题做到第三问就开始有点做不动了，想在网上找找题解但是并没有找到，于是便想分享一下自己的解题经验和第三问的思路。鉴于知识的局限性，文中可能存在疏漏或不足之处，如果发现任何错误或不准确之处，请不吝赐教。

## 题目

![](https://attach.52pojie.cn/forum/202408/13/200153ugqiiiu265jui2gi.png)

image-20240813190641633.png (135.45 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcxNzIyNHwyNzNiZWE0OXwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIyNHwyNzNiZWE0OXwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:01 上传

## （一）解题过程

第一问让我们杀死进程，可以尝试根据 `windowsAPI` 提供的 `TerminateProcess` 函数去停止进程，写份代码测试下

```
bool KillProcessByName(const char* processName) {
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe;
    pe.dwSize = sizeof(PROCESSENTRY32);
    if (Process32First(hSnapshot, &pe)) {
        do {
            if (strcmp(pe.szExeFile, processName) == 0) {
                HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pe.th32ProcessID);
                if (hProcess) {
                    TerminateProcess(hProcess, 0);
                    CloseHandle(hProcess);
                    Num++;
                    printf("Kill suc:[%d]\n", Num);
                }
            }
        } while (Process32Next(hSnapshot, &pe));
    }

    CloseHandle(hSnapshot);
    return true;
}
int main()
{
    while(1)
    {
        KillProcessByName("WorkingService.exe");
    }
    return 0;
}

```

以管理员身份启动测试

启动前：

![](https://attach.52pojie.cn/forum/202408/13/200155gg5qjqur5jqqjl9q.png)

image-20240813191830392.png (16.44 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcxNzIyNXwyYzMyNTAzYnwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIyNXwyYzMyNTAzYnwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:01 上传

启动后

![](https://attach.52pojie.cn/forum/202408/13/200157z44jqqst4vt62she.png)

image-20240813191935364.png (46.86 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcxNzIyNnxiNTVhMTgwOHwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIyNnxiNTVhMTgwOHwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:01 上传

![](https://attach.52pojie.cn/forum/202408/13/200159vdqd07o93qzdqtp7.png)

image-20240813192038178.png (14.34 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNzIyN3xlODNiYTk1ZnwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIyN3xlODNiYTk1ZnwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:01 上传

## （二）解题过程

先 `DIE` 查壳发现有 `VMP`，只能动调入手，先观察程序功能。程序启动后会有两个进程启动，并且其中之一占用CPU性能较高；程序目录会创建十六个文件，并会做重复的删除和重新创建的操作；观察任务管理器发现进程会自动重启，手动杀死进程后就会重生一个新的进程。

拖进 `xdbg` 里观察符号，发现引入了 `ShellExcuteA` 这个函数，正好是启动进程函数，对这个函数下断点，发现确实断了下来，观察传参窗口。

![](https://attach.52pojie.cn/forum/202408/13/200201el3khztqtcvk37l3.png)

image-20240810184600095.png (42.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcxNzIyOHxjYTg1MzA2YXwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIyOHxjYTg1MzA2YXwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:02 上传

第六个参数不同，对应的是 `lpParameters`（传入进程的参数），一个是 `working`，一个是 `daemon restart`

根据这个在控制台上运行加上参数 `working`，发现只有一个工作进程，CPU占用100%，同理加上 `daemon restart`，只有一个守护进程CPU占用不到1%。

调试参数为 `working` 的进程，观察线程发现有16个线程运行同一个函数，刚好对应16个 `txt` 日志文件的输出！！！

![](https://attach.52pojie.cn/forum/202408/13/200059iwby5ytvhjvzhhyb.png)

image-20240810190419709.png (21.63 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzIwMHw5NTExYmM4MXwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIwMHw5NTExYmM4MXwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:00 上传

于是进入线程入口开始分析，对所有可能的写入文件的 `windowsAPI` 下断，发现都没有断下来，于是向 `CreateFile` 等API下断，

所有线程在 `CreateFileA` 上成功断下，观察传参确定这是对日志文件的操作函数

![](https://attach.52pojie.cn/forum/202408/13/200101l7sayj2nuv6skyo4.png)

image-20240810192612468.png (16.14 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcxNzIwMXxmYzc3OGYyNnwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzIwMXxmYzc3OGYyNnwxNzkzNTcxOTczfDE2ODA4NnwxOTU0MjI1&nothumb=yes)

2024-8-13 20:01 上传

有了这个依据，能确认的就是16个线程会循环通过 `CreateFileA` 打开文件，这可能是占用CPU的一个主要因素。但是跟之前运行的结果不同，调试发现，写入文件的API一直没有被调用，而正常运行的结果是，十六个文件会循环的做一个过程，文件被创建->文件被写入->文件被删除->新文件创建…。我通过加 `working` 参数运行，只会有一个循环过程，即不会有新文件诞生，文件内容也不会被修改。所以，十六个线程循环 `CreateFileA` 打开文件过程是无用功，只会徒增CPU负担！那么我们是不是可以HOOK `CreateFileA` 这个函数，然后每个线程调用它的第二次的时候我们进行线程终止，是不是可以减少CPU的压力了呢？写份代码跑一下看看

```
#include "pch.h"

#include 

#include 

#include 

#include 

#pragma comment(lib,"detours.lib")

#define _DEBUG

#define DBGMGEBOX(fmt, ...) \

    do { \

         /* 假设最大长度为1024，根据需要调整大小 */ \

        wsprintfA(out, fmt, __VA_ARGS__); \

        MessageBoxA(NULL, out, "提示", MB_OK); \

    } while(0)

char out[100];

DWORD tlsIndex;//tls索引

typedef BOOL(WINAPI* ShellExecuteExA_t)(SHELLEXECUTEINFOA*);

typedef HANDLE (WINAPI* CreateFileA_t)(

    LPCSTR                lpFileName,

    DWORD                 dwDesiredAccess,

    DWORD                 dwShareMode,

    LPSECURITY_ATTRIBUTES lpSecurityAttributes,

    DWORD                 dwCreationDisposition,

    DWORD                 dwFlagsAndAttributes,

    HANDLE                hTemplateFile

);

ShellExecuteExA_t TrueShellExecuteExA = NULL;

CreateFileA_t TrueCreateFileA = NULL;

BOOL WINAPI HookedShellExecuteExA(SHELLEXECUTEINFOA* pExecInfo) {

#if 1

    //执行第一个ShellExecuteExA守护进程

    static int Num = 0;

    DBGMGEBOX("ShellExecuteExA 被调用:Num = %d\nhProcess = %p", Num, pExecInfo->hProcess);

    if (Num == 0)

    {

        Num++;

        return TrueShellExecuteExA(pExecInfo);

    }

    else

    {

        return TrueShellExecuteExA(pExecInfo);

    }

#else

    //执行第二个ShellExecuteExA病毒进程

    static int Num = 0;

    DBGMGEBOX("ShellExecuteExA 被调用:Num = %d \n调用者窗口句柄 = 0x%p\n", Num, pExecInfo->hwnd);

    if (Num == 0)

    {

        Num++;

        DBGMGEBOX("[2]:当前线程ID：%d", GetCurrentThreadId());

        pExecInfo->lpFile = "C:\\Users\\Administrator\\Desktop\\自动F8直到call.txt";//修改参数导致重启失败;

        return TrueShellExecuteExA(pExecInfo);

    }

    else

    {

        DBGMGEBOX("[1]:当前线程ID：%d", GetCurrentThreadId());

        return TrueShellExecuteExA(pExecInfo);

    }

#endif

}

HANDLE WINAPI HookCreateFileA(

    LPCSTR                lpFileName,

    DWORD                 dwDesiredAccess,

    DWORD                 dwShareMode,

    LPSECURITY_ATTRIBUTES lpSecurityAttributes,

    DWORD                 dwCreationDisposition,

    DWORD                 dwFlagsAndAttributes,

    HANDLE                hTemplateFile

)

{

    //判断线程是否是第一次运行CreateFileA，是的话就放行，不是第一次运行就终止线程

    // 获取当前线程的TLS值

    LPVOID tlsValue = TlsGetValue(tlsIndex);

    if (tlsValue == NULL)

    {

        // 第一次运行，设置TLS值

#ifdef _DEBUG

        DBGMGEBOX("放行\nlpFileName:%s\n", lpFileName);

#endif 

        TlsSetValue(tlsIndex, (LPVOID)1);

    }

    else

    {

        // 不是第一次运行，终止线程

#ifdef _DEBUG

        DBGMGEBOX("终止\nlpFileName:%s\n", lpFileName);

#endif 

        ExitThread(0);

    }

    return CreateFileA(lpFileName, dwDesiredAccess, dwShareMode, lpSecurityAttributes, dwCreationDisposition, dwFlagsAndAttributes, hTemplateFile);

}

BOOL APIENTRY DllMain(HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {

    switch (ul_reason_for_call) {

    case DLL_PROCESS_ATTACH:

        DetourTransactionBegin();

        DetourUpdateThread(GetCurrentThread());

        /*TrueShellExecuteExA = (ShellExecuteExA_t)DetourFindFunction("shell32.dll", "ShellExecuteExA");

        DetourAttach(&(PVOID&)TrueShellExecuteExA, HookedShellExecuteExA);*/

        tlsIndex = TlsAlloc();//初始化TLS
```

