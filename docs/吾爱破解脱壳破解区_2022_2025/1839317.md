
# [原创] Baymax64 过 VMP 加壳保护的程序

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1839317>
- 抓取时间：2026-11-01T22:44:56

有很多人在咨询工具的使用方案，这里通过我写的一个 CrackMe 来演示几种异常补丁方案的使用。

## 源码中的判断函数

```c
DWORD __stdcall NsCheckInputThread( LPVOID lpThreadParameter )
{
        HWND hWnd = (HWND)lpThreadParameter;
        CHAR szBuffer[MAX_PATH] = {0};
        int nStrLen = GetDlgItemTextA( hWnd, IDC_EDIT, szBuffer, MAX_PATH );
        if ( nStrLen ){
                unsigned char szSrc[0x10];
                unsigned char szDest[0x10];
                CalcMD5( (unsigned char*)szBuffer, nStrLen, szSrc );

                *(PDWORD)((PDWORD)szDest + 0) = 0x7de2f924;
                *(PDWORD)((PDWORD)szDest + 1) = 0x058698b1;
                *(PDWORD)((PDWORD)szDest + 2) = 0xe2c6cdc6;
                *(PDWORD)((PDWORD)szDest + 3) = 0x02a1bd7b;

                if( memcmp( szSrc, szDest, 0x10 ) == 0 ){
                        MessageBox( hWnd, L"Success!",L"Info", 0);
                        return 1;
                }
        }
        MessageBox( hWnd, L"Error!",L"Info", 0);
        return 0;
}
```

我们编译源码后，对 x64 产物使用 VMP3.8 加壳保护，比较函数未调用 VMP 保护，调试器载入，搜索字符串来到比较关键点：

![400.png](https://attach.52pojie.cn/forum/202309/28/163138bat2uut2etccettk.png)

用 Baymax 对付加壳的程序，第一步要先找到壳的解码时机，通俗来说，就是要通过 HOOK API 等方式等待被补丁的地址数据解码，如果使用 HOOK API 方案，可以调用壳自身使用的 API，或者在执行到模块被补丁地址前调用过的 API 均可。由于这个 CrackMe 获取窗口输入文本调用的 API 为 `GetWindowTextA`，所以我们就直接 HOOK 这个 API 了，这样加任何壳补丁方案都可以通杀了。

![001.png](https://attach.52pojie.cn/forum/202309/28/162039bev13a4094cxfyjb.png)

由于代码中调用了 VMP 壳内存校验的函数，所以不要直接 Patch 汇编指令，而要使用异常机制来绕过内存校验实现补丁。

## 方案一、修改标志寄存器实现比较结果为真

特征码1：`85 C0 75 23 45 33 C9`，偏移为 2。

注意：我们在使用特征码搜索地址时，一定要确保其唯一性，比如可使用插件来查找结果。

![401.png](https://attach.52pojie.cn/forum/202309/28/163145esl3b3q3tykfbby3.png)

设置 `ZF` 标志位为 1 即可。

![002.png](https://attach.52pojie.cn/forum/202309/28/162207l9trp9oy6z65ky9v.png)

最后，异常设置这里使用对当前线程设置异常即可，不要勾选对所有线程设置异常，会被 VMP 检测到被调试。

![003.png](https://attach.52pojie.cn/forum/202309/28/162256nj3jau8ajtiuujhh.png)

## 方案二、修改 `memcpy` 比较函数的参数

```asm
00007FF74A7017DC | 41:B8 10000000          | mov r8d,10                                      |
00007FF74A7017E2 | 48:8D9424 50010000      | lea rdx,qword ptr ss:[rsp+150]                  | rdx:EntryPoint
00007FF74A7017EA | 48:8D8C24 60010000      | lea rcx,qword ptr ss:[rsp+160]                  |
00007FF74A7017F2 | E8 99940000             | call testcheckmemory64.7FF74A70AC90             | memcmp 比较函数
00007FF74A7017F7 | 85C0                    | test eax,eax                                    |
00007FF74A7017F9 | 75 23                   | jne testcheckmemory64.7FF74A70181E              | 关键跳转
```

特征码地址：`E8 ?? ?? ?? ?? 85 C0 75 23`。

该方案比较简单，直接将 `RDX` 寄存器数值修改为 `RCX` 即可。

![101.png](https://attach.52pojie.cn/forum/202309/28/162350r4uqto4u6qa6qo4j.png)

## 方案三、修改参与比较的 MD5 内存数据

特征码地址：`E8 ?? ?? ?? ?? 85 C0 75 23`。

直接将 `RDX` 寄存器指向的内存的数据修改为 `RCX` 指向内存的数据即可，长度为 `0x10`。注意：`Baymax` 输入的数值均为 HEX 数值。

![201.png](https://attach.52pojie.cn/forum/202309/28/162510d5xmifyf55yyzama.png)

## 方案四、修改 `memcpy` 函数返回值

我们进入 `memcmp` 函数：

```asm
00007FF74A70AC90 | 48:2BD1                 | sub rdx,rcx                                     | rdx:EntryPoint
00007FF74A70AC93 | 49:83F8 08              | cmp r8,8                                        |
00007FF74A70AC97 | 72 22                   | jb testcheckmemory64.7FF74A70ACBB               |
00007FF74A70AC99 | F6C1 07                 | test cl,7                                       |
00007FF74A70AC9C | 74 14                   | je testcheckmemory64.7FF74A70ACB2               |
00007FF74A70AC9E | 66:90                   | nop     
```

提取函数入口特征码：`48 2B D1 49 83 F8 08 72 22 F6 C1 07 74 14`。

1. 勾选修改函数返回值，设置函数返回时修改类型，栈偏移为 0。
2. 设置 `RAX` 寄存器为 0，即 `memcmp` 函数返回 0，比较内存数据相同。
3. 条件断点参数：`r8d == 0x10`。

![301.png](https://attach.52pojie.cn/forum/202309/28/162649i8tgnzqql6fbtign.png)

## 下载

目标程序+源码+补丁方案下载：

- 链接：<https://pan.baidu.com/s/1afZOKMIdKUFI5vLbt4jmBg>
- 提取码：`cgr5`
- 解压密码：`Baymax64`
