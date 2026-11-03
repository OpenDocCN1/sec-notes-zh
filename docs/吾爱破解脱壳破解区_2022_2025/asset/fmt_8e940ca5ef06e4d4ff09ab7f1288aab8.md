
这里最开始是想通过`VirtualProtect`函数和写入`ShellCode`来实现`hook`，但是在调试的时候发现了报错。

![image-20240728204525969.png](https://attach.52pojie.cn/forum/202407/28/213052hr515kdq0r955cs9.png)

`image-20240728204525969.png` (35.51 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMjc1Nnw0ZmQ3ODEzMHwxNzkzNTcyMDEyfDE2ODA4NnwxOTQ4OTA0&nothumb=yes)

2024-7-28 21:30 上传

显示没有写入的权限，那只能先加载`dll`进去调试一番。

![image-20240728205314971.png](https://attach.52pojie.cn/forum/202407/28/213054s38iaaza8ayyx1oa.png)

`image-20240728205314971.png` (71.57 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMjc1N3w5OGU3NmRkYnwxNzkzNTcyMDEyfDE2ODA4NnwxOTQ4OTA0&nothumb=yes)

2024-7-28 21:30 上传

![image-20240728205359287.png](https://attach.52pojie.cn/forum/202407/28/213102epn3zmcnwino9qqc.png)

`image-20240728205359287.png` (105.9 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMjc1OHxkNzJjNGRhMXwxNzkzNTcyMDEyfDE2ODA4NnwxOTQ4OTA0&nothumb=yes)

2024-7-28 21:31 上传

根据调试结果发现，在`virtualProtect`时返回了`NULL`，步进分析。

![image-20240728205602337.png](https://attach.52pojie.cn/forum/202407/28/213103s1ft6e85olzt6h2n.png)

`image-20240728205602337.png` (78.2 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMjc1OXw5NDQ2NTAzY3wxNzkzNTcyMDEyfDE2ODA4NnwxOTQ4OTA0&nothumb=yes)

2024-7-28 21:31 上传

![image-20240728205621763.png](https://attach.52pojie.cn/forum/202407/28/213105ajljgas2a4a3lmks.png)

`image-20240728205621763.png` (62.35 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMjc2MHwxOWM1ZTIzN3wxNzkzNTcyMDEyfDE2ODA4NnwxOTQ4OTA0&nothumb=yes)

2024-7-28 21:31 上传

到这发现了一个奇怪的`jmp`指令，和附近的其他函数形成鲜明对比，猜测作者在这动了手脚，`hook`了`ZwProtectVirtualMemory`函数，根据附近汇编代码取消钩子后运行，成功跑出结果。

![image-20240728210222057.png](https://attach.52pojie.cn/forum/202407/28/213131d42okm2bh10blz8s.png)

`image-20240728210222057.png` (280.3 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMjc2MXwyOGNjMDBiZXwxNzkzNTcyMDEyfDE2ODA4NnwxOTQ4OTA0&nothumb=yes)

2024-7-28 21:31 上传

### 代码实现

可以通过注入器`UnHook` `ZwProtectVirtualMemory`函数，然后再进行注入。

注入器代码：

```
#include 
#include 
#include 
#include 
wchar_t dllPath[] = L"C:\\Users\\15386\\Desktop\\腾旭游戏安全竞赛赛题\\23年第一轮\\wp\\Tencent2023.dll";
wchar_t exeName[] = L"contest.exe";
DWORD64 UnHookAddr = 0x00007FFBC1F90990; // ZwProtectVirtualMemory地址
BYTE UnHookShellCode[] = {
0x4C, 0x8B, 0xD1, 0xB8, 0x50, 0x00, 0x00, 0x00, 0xF6, 0x04, 0x25, 0x08, 0x03, 0xFE, 0x7F, 0x01,
 0x75, 0x03, 0x0F, 0x05, 0xC3, 0xCD, 0x2E, 0xC3
};
/* Unhook还原ZwProtectVirtualMemory
00007FFBC1F90990 
```

dll代码：

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。

#include "pch.h"

DWORD64 HookAddrA = (DWORD64)GetModuleHandleA("contest.exe") + 0xCB90; //创建文件处,call rax,add rsp,3C

DWORD64 funcA = (DWORD64)GetModuleHandleA("contest.exe") + 0xD6A0; // call rax的rax对应地址,需要hook的函数

DWORD64 ShellCodeAddrA = (DWORD64)GetModuleHandleA("contest.exe") + 0x26B16;

BYTE HookShellCodeA[] = { 0xE9,0x81,0x9F,0x01,0x00,0x90 };

/* call rax 段代码

00007FF7E9D2CB8D   | 45:31C9                              | xor r9d,r9d                                   |

00007FF7E9D2CB90   | E9 819F0100                          | jmp contest.7FF7E9D46B16                      | 创建文件

00007FF7E9D2CB95   | 90                                   | nop                                           |

00007FF7E9D2CB96   | 48:8B45 A0                           | mov rax,qword ptr ss:[rbp-60]                 |

*/

BYTE ShellCodeA[] = {

0x48, 0xBE, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xFF, 0xD6, 0x48, 0x83, 0xC4, 0x60,

 0xE9, 0x6B, 0x60, 0xFE, 0xFF

};

/* shellCodeA

00007FF7E9D46B16   | 48:BE 13117A9FFB7F0000               | mov rsi,tencent2023.7FFB9F7A1113              |

00007FF7E9D46B20   | FFD6                                 | call rsi                                      |

00007FF7E9D46B22   | 48:83C4 60                           | add rsp,60                                    |

00007FF7E9D46B26   | E9 6B60FEFF                          | jmp contest.7FF7E9D2CB96                      |

00007FF7E9D46B2B   | CC                                   | int3                                          |

*/

typedef void (*funAptr)(

DWORD64 RCX, DWORD64 RDX, DWORD64 R8, DWORD64 R9,

DWORD64 Par5, DWORD64 Par6, DWORD64 Par7, DWORD64 Par8,

DWORD64 Par9, DWORD64 Par10, DWORD64 Par11);

DWORD64 HookAddrB = (DWORD64)GetModuleHandleA("contest.exe") + 0xCEFA;

DWORD64 ShellCodeAddrB = (DWORD64)GetModuleHandleA("contest.exe") + 0x26BC0;

BYTE HookShellCodeB[] = {0xE9, 0xC1, 0x9C, 0x01, 0x00, 0x90};

/*call rax 段代码

00007FF7E9D2CEF7   | 45:31C9                              | xor r9d,r9d                                   |

00007FF7E9D2CEFA   | E9 C19C0100                          | jmp contest.7FF7E9D46BC0                      | 写入文件

00007FF7E9D2CEFF   | 90                                   | nop                                           |

00007FF7E9D2CF00   | 48:8B45 A0                           | mov rax,qword ptr ss:[rbp-60]                 |

*/

BYTE ShellCodeB[] = {

0x48, 0xB8, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xFF, 0xD0, 0x48, 0x83, 0xC4, 0x50,

 0xE9, 0x2B, 0x63, 0xFE, 0xFF

};

/* shellCodeB

00007FF7E9D46BC0   | 48:B8 AD127A9FFB7F0000               | mov rax,tencent2023.7FFB9F7A12AD              |

00007FF7E9D46BCA   | FFD0                                 | call rax                                      |

00007FF7E9D46BCC   | 48:83C4 50                           | add rsp,50                                    |

00007FF7E9D46BD0   | E9 2B63FEFF                          | jmp contest.7FF7E9D2CF00                      |

00007FF7E9D46BD5   | CC                                   | int3                                          |

*/

DWORD64 funcB = (DWORD64)GetModuleHandleA("contest.exe") + 0xD6A0; // call rax的rax对应地址,需要hook的函数

typedef void (*funBptr)(

    DWORD64 RCX, DWORD64 RDX, DWORD64 R8, DWORD64 R9,

    DWORD64 Par5, DWORD64 Par6, DWORD64 Par7, DWORD64 Par8,

    DWORD64 Par9, DWORD64 Par10,DWORD64 Par11, DWORD64 Par12,

    DWORD64 Par13, DWORD64 Par14);

DWORD oldProtect = 0;

char plainText[] = "catchmeifyoucan";

char fileName[] = "myContest.txt";

void __fastcall HookFuncA(

    DWORD64 RCX,DWORD64 RDX,DWORD64 R8,DWORD64 R9,

    DWORD64 Par5,DWORD64 Par6,DWORD64 Par7,DWORD64 Par8,

    DWORD64 Par9,DWORD64 Par10,DWORD64 Par11)

{

    //第6个参数对应文件名地址

    memcpy((void*)Par6, fileName, sizeof(fileName)); 

    funAptr ptr = (funAptr)funcA;

    return ptr(RCX, RDX, R8, R9, Par5, Par6, Par7, Par8, Par9, Par10, Par11);

}

void __fastcall HookFuncB(

    DWORD64 RCX, DWORD64 RDX, DWORD64 R8, DWORD64 R9,

    DWORD64 Par5, DWORD64 Par6, DWORD64 Par7, DWORD64 Par8,

    DWORD64 Par9, DWORD64 Par10, DWORD64 Par11,DWORD64 Par12,

    DWORD64 Par13,DWORD64 Par14)

{

    //第7个参数对应明文地址

    //第8个参数的低四字节对应写入字节数

    Par8 = (Par8 & 0xFFFFFFFFFFFFFF00) |  (strlen(plainText));

    memcpy((void*)Par7, plainText, sizeof(plainText));

    funBptr ptr = (funBptr)funcB;

    return ptr(RCX, RDX, R8, R9, Par5, Par6, Par7, Par8, Par9, Par10, Par11,Par12,Par13,Par14);

}

void InstallHookA()

{

    VirtualProtect((LPVOID)HookAddrA, sizeof(HookShellCodeA), PAGE_EXECUTE_READWRITE, &oldProtect);

    memcpy((void*)HookAddrA, HookShellCodeA, sizeof(HookShellCodeA));

    VirtualProtect((LPVOID)ShellCodeAddrA, sizeof(ShellCodeA), PAGE_EXECUTE_READWRITE, &oldProtect);

    *(DWORD64*)&ShellCodeA[2] = (DWORD64)HookFuncA;
```
