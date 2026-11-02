
# [原创] 腾讯游戏安全大赛2023题解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1775114
- 抓取时间: 2026-11-01T22:56:27

## 前言

终于如愿参加一次腾讯游戏安全大赛，初赛也算是把所有题目都做出来了，进入决赛，但是自己太菜了，只会暴力`NOP`。

决赛题目就是一个占用率很高的服务进程，让我们分析解决，前几天都有事，最后一天 all in 写完的。

## 腾讯游戏安全大赛初赛题解

### 参赛人员信息

略

### 找到明文

1. 在64位Windows10系统上运行`contest.exe`, 找到明文的信息，作为答案提交（1分）。

答案：`catchmeifyoucan`

正常运行会在当前目录下的`contest.txt`去写入一个`ImVkImx9JG12OGtlImV+`。

CE打开，先通过CE的memory view找到一串一直在变化的内存。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/1.png)

虽然不确定，找到了`contest.txt`这个明文，但是`catchmeifyoucan`不确定它是不是`flag`，还得进一步确定，于是查找是谁访问了这个地址。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/2.png)

在第三项中，发现后面有一个读取`r10`所指示的内存的操作。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/3.png)

同样memory view查找`7FF713657190`地址，结果找到一串`base`表：

`QRSTUVWXYZabcdefABCDEFGHIJKLMNOPwxyz0123456789+/ghijklmnopqrstuv`

于是果断拿密文解密，发现得到了正确答案。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/4.png)

### 写入明文信息

2. 编写程序，运行时修改尽量少的`contest.exe`内存，让`contest.exe`由写入密文信息变为写入明文信息成功。（满分2分）

Base64编码的一组中，三个原文对应四个密文。第一个密文和第四个密文只会被原文第一个字符和第三个字符影响。第二个字符会被第一个原文和第二个原文字符影响，第三个字符会被第二个原文和第三个原文字符影响。

对`base`表进行访问查询，找到三个地址，对三个地址分析。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/5.png)

查看第一项访问地址

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/6.png)

右移两位进行查表，很明显就是对第一个密文的操作，因此把中间的操作全部`NOP`，只剩读取和写入操作。

再对第二个项目分析，很明显看到了对第四个密文的操作：

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/7.png)

这里的`and 0x3f`就是取得最低的六位，很明显是最后一位，同样也把中间的指令全部`NOP`掉。

不仅如此，同时注意到赋值的时候，对`r14+1`了，并且`r14`的值在后面+1。这里有一个很头疼的点就是明文密文长度不一致，三个原文对应四个密文，因此这里可以为这个点考虑起来了。如果我`r14`的值不给他+1，直接给`r14`的值赋值，结果会怎样呢？也就是对应指令修改为：

```
mov [r14],al
add r14,0
```

发现输出文件的内容果然变短了。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/8.png)

并且此时，我们再处理中间的一位，就能得到明文输出了。

我们来看第三个项目：

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/9.png)

这里同样把`BA5D-BA68`的代码段`NOP`掉，做完之后，发现`contest`输出明文了。

总结一下：

- `contest.exe+BA39~BA41`全部`NOP`掉
- `contest.exe+B9FD~BA04`全部`NOP`掉
- `contest.exe+BA5D~BA68`全部`NOP`掉
- `contest.exe+BA05`开始，把指令变成：

```
mov [r14],al
add r14,0
```

编写dll进行一次性修改：

但是在写内存的时候，发现`VirtualProtect`一直调用失败，拿火绒剑扫了一下，发现`VirtualProtect`被下了钩子。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/10.png)

尝试修复一下这个钩子，写一个注入器和dll

#### 思路讲解

DLL不能直接使用`VirtualProtect`去修改内存属性，所以我们需要在注入之前，使用`VirtualProtectEx`先修改内存权限，再通过`WriteProcessMemory`函数修复程序在API处下的一个钩子，这里是inline hook，因此直接遍历模块寻找`ZwProtectVirtualMemory`函数把钩子取消。

取消之后，即可使用远程线程注入的方式去注入dll，远程注入的思路就是开辟一块远程内存，写入dll路径，创建远程线程回调`LoadLibraryA`函数去加载DLL。

DLL主要就是作上面分析的一些`PATCH`内存的操作。

下面是源码：

#### 注入器

```
#include
#include
#include
#include
#include
DWORD old;
SIZE_T written;
DWORD FindProcess() {
    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe32;
    pe32 = { sizeof(pe32) };
    BOOL ret = Process32First(hSnap, &pe32);
    while (ret)
    {
        if (!wcsncmp(pe32.szExeFile, L"contest.exe", 11)) {
            printf("Find contest.exe Process %d\n", pe32.th32ProcessID);
            return pe32.th32ProcessID;
        }
        ret = Process32Next(hSnap, &pe32);
    }
    return 0;
}
void InjectModule(DWORD ProcessId, const char* szPath)
{
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, ProcessId);
    printf("进程句柄:%p\n", hProcess);
    LPVOID lpAddress = VirtualAllocEx(hProcess, NULL, 0x100, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
    SIZE_T dwWriteLength = 0;
    WriteProcessMemory(hProcess, lpAddress, szPath, strlen(szPath), &dwWriteLength);
    HANDLE hThread = CreateRemoteThread(hProcess, NULL, NULL, (LPTHREAD_START_ROUTINE)LoadLibraryA, lpAddress, NULL, NULL);
    WaitForSingleObject(hThread, -1);
    VirtualFreeEx(hProcess, lpAddress, 0, MEM_RELEASE);
    CloseHandle(hProcess);
    CloseHandle(hThread);
}
void UNHOOK(DWORD ProcessId) {
    BYTE INS[] = {0x4C,0x8B,0xD1,0xB8,0x50};
    HANDLE ths = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, ProcessId);
    MODULEENTRY32 me;
    me.dwSize = sizeof(me);
    UINT64 addr=0;
    if (Module32First(ths, &me))
    {
        do
        {
            if (addr=(UINT64)GetProcAddress(me.hModule, "ZwProtectVirtualMemory"))
            {
                printf("addr:%p\n", addr);
                break;
            }

        } while (Module32Next(ths, &me));
    }
    CloseHandle(ths);
    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, ProcessId);
    VirtualProtectEx(hProcess, (void *)addr, 0x5, PAGE_EXECUTE_READWRITE, &old);
    WriteProcessMemory(hProcess, (void *)addr, INS, 0x5, &written);
    printf("written:%d\n", written);
    VirtualProtectEx(hProcess, (void*)addr, 0x5, old, &old);
    CloseHandle(hProcess);
}
int main() {
    DWORD ProcessId = FindProcess();
    while (!ProcessId) {
        printf("未找到contest程序，等待两秒中再试\n");
        Sleep(2000);
        ProcessId = FindProcess();
    }
    printf("尝试去除钩子...\n");
    UNHOOK(ProcessId);//去除钩子
    printf("开始注入进程...\n");
    InjectModule(ProcessId, "C:\\Users\\xia0ji233\\source\\repos\\T-contest\\x64\\Debug\\T-contest.dll");
    printf("注入完毕\n");
}
```

#### DLL

```
#include

#include

#include

DWORD oldprot,ret;

PROC HookedFunction;

UINT64 Offset[3] = { 0xBA39 ,0xB9FD ,0xBA5D }, Len[3] = { 9,8,12 };//PATCH偏移和PATCH长度，这里皆patch为0x90(NOP)

BYTE Ins[] = { 

    0x41,0x88,0x06,     //mov [r14],al

    0x90,               //nop

    0x49,0x83,0xC6,0x00 //add r14, 0

};

UINT64 InsOffset = 0xBA05,InsLen=sizeof(Ins);

SIZE_T num;

BYTE buf = 0x90;

BYTE NOP[] = { 0x90,0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90 };

void patch() {

    UINT64 Base = (UINT64)GetModuleHandle(nullptr);

    for (int i = 0; i 
```

#### 更改写入文件

在一开始的时候我们在内存看到了`contest.txt`，于是尝试直接修改，发现可以成功更改写入的文件，因此为了达到目的我们可以直接修改这里的内存，但是因为它一直在变化，因此可以查看什么写了这个内存。

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/11.png)

主要是做了一个异或运算，直接`NOP`即可。

需要注意在把指令`patch`掉之后，一定要写一下这里存储`Name`的地方，把`Name`替换为自己想要的名字即可。

但是发现`flag`写进去时而变化，究其原因还是因为`flag`内存一直在变化，因此`patch`修改`flag`的内存让它也不再变化即可。

注入器不变，DLL需要稍微改变一下。

```
#include
#include
#include
DWORD oldprot,ret;
PROC HookedFunction;
UINT64 Offset[3] = { 0xBA39 ,0xB9FD ,0xBA5D }, Len[3] = { 9,8,12 };//PATCH偏移和PATCH长度，这里皆patch为0x90(NOP)
BYTE Ins[] = { 
    0x41,0x88,0x06,     //mov [r14],al
    0x90,               //nop
    0x49,0x83,0xC6,0x00 //add r14, 0
};

UINT64 InsOffset = 0xBA05,InsLen=sizeof(Ins);
SIZE_T num;
BYTE buf = 0x90;
BYTE NOP[] = { 0x90,0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90 };
void patch() {
    UINT64 Base = (UINT64)GetModuleHandle(nullptr);
    for (int i = 0; i 
```

#### 结果

![](https://xia0ji233.pro/2023/04/12/tencent-race-2023-pre/12.png)

## 腾讯游戏安全大赛决赛题解

### 结束进程

常规思路就是找进程匹配名字直接杀，但是发现会再生，用火绒剑分析了一下，发现有两个进程，应该是相互守护关系。并且`exploror.exe`进程一直在产生新的`working service`进程。猜测应该是运行了程序之后对这个进程下了钩子，于是火绒剑扫描一下钩子：

![](https://xia0ji233.pro/2023/04/17/tencent-race-2023-final/1.png)

上下图对比下图是没运行之前扫描的结果，可以发现对`ZwProtectVirtualMemory`下了钩子，和之前一样，先把这个钩子修复，然后再杀掉所有进程名为`WorkingService`的进程即可。

但是之后又发现一些很奇怪的事情，有时候可以完全杀掉，有时候杀不掉，究其原因，原来它对很多进程都进行了hook尝试，有次我打开了任务管理器没有考虑到就又复活了，因此选择遍历进程的这个模块，识别钩子并去掉。

去掉之后再循环寻找进程杀掉即可。

代码实现：

```
#include

#include

#include

#include

#include

#define EXP L"explorer"

#define PROCESSNAME L"WorkingService"

DWORD old;

SIZE_T written;

void UNHOOK() {

    DWORD ProcessId=0;

    BYTE INS[] = { 0x4C,0x8B,0xD1,0xB8,0x50 },buf;

    HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

    PROCESSENTRY32 pe32;

    pe32 = { sizeof(pe32) };

    BOOL ret = Process32First(hSnap, &pe32);

    while (ret)

    {

        if (TRUE) {

            ProcessId=pe32.th32ProcessID;

            HANDLE ths = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, ProcessId);

            if (!ths) {

                ret = Process32Next(hSnap, &pe32);

                continue;

            }

            MODULEENTRY32 me;

            me.dwSize = sizeof(me);

            UINT64 addr = 0;

            if (Module32First(ths, &me))

            {

                do

                {
```
