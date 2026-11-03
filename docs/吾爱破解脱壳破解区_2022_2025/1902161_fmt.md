
# [原创] 华硕 fx63vd7700 充电保护程序“`ASUS Battery Health Charging`”逆向分析

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1902161>
- 抓取时间：2026-11-01T22:34:55

## 0. 前言

最近好奇这个充电保护程序为什么可以控制最高充电百分比，所以我想分析一下。

该程序适用于华硕飞行堡垒 fx63vd7700 型号，如果适用华硕其它型号的可以试试看。

注意该充电保护程序不适用于其它品牌、或华硕其它型号的笔记本电脑。

这个软件的界面如下图所示。

![软件主界面](https://attach.52pojie.cn/forum/202403/17/213953doffy5aoa4gloy6f.png)

`1.png`（397.19 KB，下载次数: 2） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzA5M3w2NjYyOGMxOXwxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

模式有三种：完整充电模式、平衡保养模式、最佳保养模式。三种模式限制充电百分比分别为 100%，80%、和 60%。

目录如下：

![目录](https://attach.52pojie.cn/forum/202403/17/214351cka7h7r4h4io2aaa.png)

`2.png`（17.61 KB，下载次数: 4） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzA5NXw1OTU0YTZmY3wxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

## 1. `x64dbg` 动态调试

双击 `BhcApp.exe`，启动充电保护软件。`x64dbg` 附加“`BhcMgr.exe`”。

在 `DeviceIoControl` 处插入断点，这个函数用于和设备通信，随便选中一个模式然后点确定就会触发断点。

例如我选中的“平衡保养模式”。

调用堆栈如下图所示。

![调用堆栈](https://attach.52pojie.cn/forum/202403/17/214844lphzrpsrlidia8li.png)

`3.png`（37.57 KB，下载次数: 2） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzEwMXw3NTkzYTRlM3wxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

`DeviceIoControl` 的参数如下：

```c
BOOL DeviceIoControl(
  [in]                HANDLE       hDevice,
  [in]                DWORD        dwIoControlCode,
  [in, optional]      LPVOID       lpInBuffer,
  [in]                DWORD        nInBufferSize,
  [out, optional]     LPVOID       lpOutBuffer,
  [in]                DWORD        nOutBufferSize,
  [out, optional]     LPDWORD      lpBytesReturned,
  [in, out, optional] LPOVERLAPPED lpOverlapped
);
```

回到 `DeviceIoControl` 上层函数“`sub_321C90`”。观察 `DeviceIoControl` 的入参：

![DeviceIoControl 入参](https://attach.52pojie.cn/forum/202403/17/215410z3r1f6r316lwf57f.png)

`4.png`（72.13 KB，下载次数: 2） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzEwMnwwNmE1NjljNXwxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

- `HANDLE hDevice = 0x274`
- `DWORD dwIoControlCode = 0x22240C`
- `LPVOID lpInBuffer = 0xB7F010`
- `DWORD nInBufferSize = 0x10`
- `LPVOID lpOutBuffer = 0x93F238`
- `DWORD nOutBufferSize = 0x400`
- `LPDWORD lpBytesReturned = 0x93F234`
- `LPOVERLAPPED lpOverlapped = 0`

### 1.1 查看缓冲区 `lpInBuffer`：`0xB7F010`

如下图所示为缓冲区内容，大小 `0x10` 字节。

![缓冲区内容](https://attach.52pojie.cn/forum/202403/17/220021pe2z53qqxezynyc9.png)

`5.png`（19.65 KB，下载次数: 2） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzEwM3xhNjUxZWU2NnwxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

这个可以视为一个结构体。代码如下。

```c
struct BatteryIOCtlInfo
{
        DWORD head; // 头部
        DWORD nextSize; // 后续结构体成员大小
        DWORD num2; // 未知四字节数，固定为0x120057
        DWORD perecent; // 充电百分比，0x50为80%，0x3C为60，0x64为100%
};
```

结构体各成员值填写为：

```c
        BatteryIOCtlInfo btinfo;
        btinfo.head = 0x53564544;
        btinfo.nextSize=8;
        btinfo.num2 = 0x120057;
        // 0x3c 60%
        // 0x50 80%
        // 0x64 100%
        btinfo.perecent = 0x3c;
```

### 1.2 输出缓冲区 `lpOutBuffer`：`0x93F238`

只有头四节被填充 `0x1`，剩余的 `0x400 - 4` 个字节均为 `0`。

![输出缓冲区](https://attach.52pojie.cn/forum/202403/17/220817gm9exdy43vox39vo.png)

`6.png`（21.3 KB，下载次数: 2） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzEwN3w5OTFiNmViM3wxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

输出缓冲区里的内容没有什么特别的，只需保证有输出缓冲区，以及该缓冲区有 `0x400` 字节大小即可。

## 2. `IDA` 静态分析

以上分析了 `DeviceIoControl` 的入参，但是还不知道设备句柄是怎么获取的。

进入 `sub_321C90` 的代码段，在 `IDA` 中为“`sub_401C90`”，`00401D0C` 为调用 `DeviceIoControl` 的语句。

![调用 DeviceIoControl](https://attach.52pojie.cn/forum/202403/17/221405o5flggw63g0gflb3.png)

`7.png`（24.57 KB，下载次数: 2） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzEwOXw4ZjBkM2NmMnwxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

选中 `hDevice`，按下 X 键看看哪个语句写入了这个 `hDevice`。如下图所示。（Type：`r` 表示读取 `read`，`w` 表示写入 `write`）

![hDevice 引用](https://attach.52pojie.cn/forum/202403/17/221553eisss6srrtzr6ld4.png)

`8.png`（18.35 KB，下载次数: 0） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzExMHwwMjUyYjZmYnwxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

进入了函数 `sub_401D70`。

![sub_401D70](https://attach.52pojie.cn/forum/202403/17/221734r1g23yff3fgqq17l.png)

`9.png`（24.59 KB，下载次数: 1） [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MzExMXw4NWM3MGIxNHwxNzkzNTcyNDkzfDE2ODA4NnwxOTAyMTYx&nothumb=yes)

原来是 `CreateFileW` 打开（电池）设备的句柄。

`CreateFileW` 参数如下：

```c
HANDLE CreateFileW(
  [in]           LPCWSTR               lpFileName,
  [in]           DWORD                 dwDesiredAccess,
  [in]           DWORD                 dwShareMode,
  [in, optional] LPSECURITY_ATTRIBUTES lpSecurityAttributes,
  [in]           DWORD                 dwCreationDisposition,
  [in]           DWORD                 dwFlagsAndAttributes,
  [in, optional] HANDLE                hTemplateFile
);
```

- `LPCWSTR lpFileName: "\\.\ATKACPI"`
- `DWORD dwDesiredAccess: 0xC0000000 ( GENERIC_READ | GENERIC_WRITE == 0xC0000000 )`
- `DWORD dwShareMode: 0x3 (FILE_SHARE_READ | FILE_SHARE_WRITE == 0x3)`
- `LPSECURITY_ATTRIBUTES lpSecurityAttributes: NULL`
- `DWORD dwCreationDisposition: 0x3 (OPEN_EXISTING == 3)`
- `DWORD dwFlagsAndAttributes: NULL`
- `HANDLE hTemplateFile: NULL`

## 3. 代码复现

因此控制笔记本最高充电百分比的 C++ 代码如下：

```cpp
#include 
#include 
#include 

struct BatteryIOCtlInfo
{
	DWORD head;
	DWORD nextSize;
	DWORD num2;
	DWORD perecent;
};

int main(int argc, char** argv, char** envp)
{
	// GENERIC_READ | GENERIC_WRITE == 0xC0000000
	// FILE_SHARE_READ | FILE_SHARE_WRITE == 0x3
	// OPEN_EXISTING == 3
	HANDLE hhh = CreateFileW
		(
			L"\\\\.\\ATKACPI", 
			GENERIC_READ | GENERIC_WRITE,
			FILE_SHARE_READ | FILE_SHARE_WRITE, 
			NULL,
			OPEN_EXISTING,
			NULL,
			NULL
		);
	//
	BatteryIOCtlInfo btinfo;
	btinfo.head = 0x53564544;
	btinfo.nextSize=8;
	btinfo.num2 = 0x120057;
	// 0x3c 60%
	// 0x50 80%
	// 0x64 100%
	btinfo.perecent = 0x3c;  // 修改此处数值
	// out buffer
	void* out_buf = malloc(0x400);
	DWORD returnBytes = 0;
	//
	DeviceIoControl(
		hhh,
		0x22240C,
		&btinfo,
		sizeof(btinfo),
		out_buf,
		0x400,
		&returnBytes,
		NULL);
	CloseHandle(hhh);
	free(out_buf);
	return 0;
}
```

经测试可以修改最高充电百分比，但是只能有 60%，80% 和 100% 三种数值。如果修改为其它数值，则默认最高充电百分比为 100%。
