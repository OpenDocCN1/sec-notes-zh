
## 绘图代码

```python
import struct
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
float_values = [  71.2234,       73.5287,             0,             1,
        1,             0,             1,        71.062,
  73.5287,             ...]
coordinates = [float_values[i:i+2] for i in range(0,len(float_values),2)]

fig = plt.figure()

ax = fig.add_subplot(111)

x_values = [x for x, _ in coordinates]
y_values = [y for _, y in coordinates]

ax.scatter(x_values, y_values)

plt.show()
```

## 运行结果

运行结果如图：

![](https://attach.52pojie.cn/forum/202408/06/235858hf4yekc2e8e0hh5t.png)

`image-20240624221016969.png` (34.11 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNTQxOHwyN2VjMDZiNXwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNTQxOHwyN2VjMDZiNXwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes)

2024-8-6 23:58 上传

这里有一个密集位置，箭头指向应该是让我们放大看。

![](https://attach.52pojie.cn/forum/202408/06/235905g9pxqrxpq009q0rb.png)

`image-20240624221254947.png` (36.06 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNTQyMHxmNjE4YWMwMHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNTQyMHxmNjE4YWMwMHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes)

2024-8-6 23:59 上传

确实出现了原图的形状，但是 flag 旗帜的点也嵌在其中，但是对比位置好像有点问题，而且点的数量也不够，还有一个问题就是为什么这些位置没有被实际展现出来。

## 问题分析

那我们先解决为什么没有展现出来的问题。根据之前的分析，这里的几个跳转语句可能是问题所在。这里类似一个 `switch` 语句，通过数组的索引和其中的值来决定下一步的运算，有加减和一些别的运算。有点类似虚拟机的 `opcode`。

![](https://attach.52pojie.cn/forum/202408/06/235907ofstqkrqmxftr33r.png)

`image-20240626100831665.png` (138.48 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNTQyMXw2YTMyYWVkZXwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNTQyMXw2YTMyYWVkZXwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes)

2024-8-6 23:59 上传

这里汇编有点没分析出来啥意思，拖到 IDA 里调试看看伪代码。

![](https://attach.52pojie.cn/forum/202408/06/235909xjeelnhfdugk9cfs.png)

`image-20240626103148199.png` (65.39 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNTQyMnxhNDdhNzY2MHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNTQyMnxhNDdhNzY2MHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes)

2024-8-6 23:59 上传

`dword_E3D1301` 应该就是 `opcode` 数组，然后 `v14` 应该是指八个寄存器。然后 `case4` 有点没看像是一种加解密运算；`case5` 和 `case6` 是两个相同的函数，传参只有一个不同，`0xFFFFFF00` 和 `0xFF2DDBE7`，可能与 `rgb` 的十六进制值有关，上在线转换网站上看一下。

![](https://attach.52pojie.cn/forum/202408/06/235910spsszo67qbwsbmg5.png)

`image-20240626103952814.png` (10.5 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNTQyM3xkNTM1NmQyMHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNTQyM3xkNTM1NmQyMHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes)

2024-8-6 23:59 上传

![](https://attach.52pojie.cn/forum/202408/06/235954krdifoq6sggdcqdd.png)

`image-20240626104016412.png` (10.56 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcxNTQyOXxhZjZlZmVkZHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNTQyOXxhZjZlZmVkZHwxNzkzNTcxOTg5fDE2ODA4NnwxOTUyMDcy&nothumb=yes)

2024-8-6 23:59 上传

应该就是 flag 的黄色和原图的蓝色了，但是为什么同一个函数，只是颜色不同，怎么会有不显示和显示的情况呢？不清楚，`hook` 一下看一下调用的时候其他参数有什么区别。

### `hook` 代码

```cpp
// dllmain.cpp : 定义 DLL 应用程序的入口点。

#include "pch.h"

#include 

#include 

#pragma comment(lib,"detours.lib")

//addr = [140008318] - 0x650

#define QWORDADDR 0x140008318

#define OFFSET 0x650

typedef __int64 (*funptr)(int, int, int, int, int, __int64, __int64, __int64, __int64, __int64);

INT64 func;

__int64 __fastcall funcHook(

    int a1,

    int a2,

    int a3,

    int a4,

    int a5,

    __int64 a6,

    __int64 a7,

    __int64 a8,

    __int64 a9,

    __int64 a10)

{

    FILE* f = fopen("C:\\Users\\15386\\Desktop\\out.txt", "a+");

    char in[100];

    sprintf(in,"a1:0x%08x,a2:0x%08x,a3:0x%08x,a4:0x%08x,a5:0x%08x\n", a1, a2, a3, a4, a5);

    fputs(in, f);

    fclose(f);

    funptr fpt;

    fpt = (funptr)func;

    fpt(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10);

    return 0;

}

void Start_Hook()

{

        DetourRestoreAfterWith();

        DetourTransactionBegin();

        DetourUpdateThread(GetCurrentThread());

        INT64 qword_addr;

    if (!ReadProcessMemory(GetCurrentProcess(), (LPCVOID*)QWORDADDR, &qword_addr, 8, NULL))

    {

        FILE* f = fopen("C:\\Users\\15386\\Desktop\\out.txt", "a+");

        fputs("error ReadProcessMemory\n", f);

        fclose(f);

        return;

    }

    func = qword_addr - OFFSET;

        DetourAttach((PVOID*)&func, funcHook);//msg

        DetourTransactionCommit();

}

void Exit_Hook()

{

        DetourTransactionBegin();

        DetourUpdateThread(GetCurrentThread());

        DetourDetach((PVOID*)&func, funcHook);

        DetourTransactionCommit();

}

BOOL APIENTRY DllMain( HMODULE hModule,

                       DWORD  ul_reason_for_call,

                       LPVOID lpReserved

                     )

{

    switch (ul_reason_for_call)

    {

    case DLL_PROCESS_ATTACH:

                Start_Hook();

                break;

    case DLL_THREAD_ATTACH:

    case DLL_THREAD_DETACH:

    case DLL_PROCESS_DETACH:

                Exit_Hook();

        break;

    }

    return TRUE;

}
```

得到前五个参数内容
