
# [调试逆向] 微信逆向：hook 强制输出调试信息
- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1940040
- 抓取时间: 2026-11-01T21:53:40

本文仅用于学习研究，请勿用于非法用途和商业用途！如因此产生任何法律纠纷，均与作者无关！

## 环境准备
- wechat x64版本
- x64dbg
- ida 或者 ghidra
- debugview++
- Visual Studio 2022

## 定位调试信息函数
对于大型软件来说，由于其庞大复杂的功能，为了开发人员更好的定位 bug，会输出调试信息辅助调试，对于逆向来说，有调试信息也会有助于更好的定位功能点

调试信息一般是固定格式的字符串，例如：日期 - 功能 - 函数名 - 文件名

定位调试信息的方法：找到固定格式大量出现的字符串，然后都点进去看看是不是同一个函数

对于微信来说，在查看核心模块 `wechatwin.dll` 的时候，就会看到很多疑似调试信息的伪代码，例如：（这里我进行了函数重命名）

```
    log_message(
      2,
      (__int64)"D:\\Tools\\agent\\workspace\\MicroMsgWindowsV3911\\MicroMsgWin\\01_ui\\chatroom\\ChatMsgList.cpp",
      35,
      (__int64)"ChatMsgList::InsertChatItem",
      "ChatMsgList",
      "InsertChatItem msg svrid : %d, pre localid : %d",
      &v13,
      &v12,
      v17,
      &v16,
      &v15,
      &v14);
```

后来调试发现，确实是这个函数在拼接调试信息，但是没有打印出来，这里应该是微信禁用了调试信息的输出

既然如此，那就手动打印调试信息吧，`hook` 该函数获取调试信息进行手动打印输出

>

还有一个思路找日志函数（参考资料[0]），搜索代码文件的 `.cpp` 后缀

## 分析 hook 点
参考资料[0]找的日志信息位于日志函数执行时候的栈信息，要实现从栈中拿数据，只需要写 `dll` 注入，`hook` 有数据的那个地方，读取栈指针去读取内容

但是我是个懒人，能自动就不想手动，`hook` 库 `detours` 似乎只能 `hook` 函数开头的位置（`hook` 其他地方之后，补的指令有问题）

所以就找找内部某个函数的参数或者返回值是否满足我的要求：具有完整的日志信息

完整伪代码：

```
void log_message(

        int a1,

        __int64 a2,

        int a3,

        __int64 a4,

        const char *a5,

        const char *a6,

        __int128 *a7,

        __int128 *a8,

        __int128 *a9,

        __int128 *a10,

        __int128 *a11,

        __int128 *a12,

        ...)

{

  __int128 *v15; // rax

  __int64 v16; // rcx

  const char *v17; // rsi

  int v18; // ecx

  __int64 v19; // r9

  __int128 *v20; // r8

  char *v21; // rdi

  __int64 v22; // rdi

  _BYTE *v23; // rdx

  DWORD CurrentThreadId; // [rsp+50h] [rbp-B0h]

  int v26; // [rsp+80h] [rbp-80h] BYREF

  char *v27; // [rsp+88h] [rbp-78h]

  __int64 v28; // [rsp+90h] [rbp-70h]

  __int64 v29; // [rsp+98h] [rbp-68h]

  int v30; // [rsp+A0h] [rbp-60h]

  char v31[12]; // [rsp+A4h] [rbp-5Ch] BYREF

  __int64 v32; // [rsp+B0h] [rbp-50h]

  __int64 v33; // [rsp+B8h] [rbp-48h]

  __int64 v34; // [rsp+C0h] [rbp-40h]

  struct _SYSTEMTIME SystemTime; // [rsp+D0h] [rbp-30h] BYREF

  void *Src; // [rsp+E0h] [rbp-20h] BYREF

  int v37; // [rsp+E8h] [rbp-18h]

  int v38; // [rsp+ECh] [rbp-14h]

  char v39[4104]; // [rsp+F0h] [rbp-10h] BYREF

  __int128 v40[6]; // [rsp+10F8h] [rbp+FF8h] BYREF

  int v41; // [rsp+1158h] [rbp+1058h]

  int v42; // [rsp+115Ch] [rbp+105Ch]

  __int128 v43[6]; // [rsp+1160h] [rbp+1060h] BYREF

  char Buffer[16]; // [rsp+11C0h] [rbp+10C0h] BYREF

  __int128 v45; // [rsp+11D0h] [rbp+10D0h]

  __int128 v46; // [rsp+11E0h] [rbp+10E0h]

  __int128 v47; // [rsp+11F0h] [rbp+10F0h]

  char v48[256]; // [rsp+1200h] [rbp+1100h] BYREF

  if ( sub_183CEBBD0(a1) )

  {

    v15 = v40;

    v16 = 6i64;

    do

    {

      *(_BYTE *)v15++ = -1;

      --v16;

    }

    while ( v16 );

    v39[0] = 0;

    Src = v39;

    v38 = 4096;

    v17 = 0i64;

    v37 = 0;

    v41 = 0;

    v18 = 0;

    v42 = 0;

    v43[0] = *a7;

    v43[1] = *a8;

    v43[2] = *a9;

    v43[3] = *a10;

    v43[4] = *a11;

    v43[5] = *a12;

    v19 = 0i64;

    v20 = v43;

    do

    {

      if ( *(_BYTE *)v20 == 0xFF )

        break;

      v40[v18] = *v20;

      v18 = ++v42;

      ++v19;

      ++v20;

    }

    while ( v19 
```

中间有 `snprintf` 的拼接字符串，完整的日志信息在这后面，分析代码可以看到，日志信息保存在了 `Src` 变量里，而这个 `v39` 也是来自 `Src` 变量

一般来说，一个函数的前面都是在为功能做准备，做好准备后执行，最后的函数就很可疑：`sub_183CEBC10`，但是劫持这里没法从参数中获得完整日志信息，只能获得部分

经过分析，看反汇编：

```
.text:000000018260F0C4                 nop
.text:000000018260F0C5                 lea     rax, [rbp+1250h+var_1260] ; v39
.text:000000018260F0C9                 mov     rcx, [rbp+1250h+Src] ; Block
.text:000000018260F0CD                 cmp     rcx, rax
.text:000000018260F0D0                 jz      short loc_18260F0D7
.text:000000018260F0D2                 call    free
.text:000000018260F0D7
.text:000000018260F0D7 loc_18260F0D7:                          ; CODE XREF: log_message+56↑j
.text:000000018260F0D7                                         ; log_message+320↑j
.text:000000018260F0D7                 mov     rcx, [rbp+1250h+var_50]
.text:000000018260F0DE                 xor     rcx, rsp        ; StackCookie
.text:000000018260F0E1                 call    __security_check_cookie
.text:000000018260F0E6                 add     rsp, 1318h
.text:000000018260F0ED                 pop     r15
.text:000000018260F0EF                 pop     r14
.text:000000018260F0F1                 pop     r13
.text:000000018260F0F3                 pop     r12
.text:000000018260F0F5                 pop     rdi
.text:000000018260F0F6                 pop     rsi
.text:000000018260F0F7                 pop     rbx
.text:000000018260F0F8                 pop     rbp
.text:000000018260F0F9                 retn
```

`log_message` 函数的末尾有一个比较，`v39` 和 `Src` 变量的比较，比较完成之后，如果相同就返回

这里 `log_message` 函数的类型是 `void`，无返回值，但实际返回的时候，`rax` 的值是 `Src` 指针，其实是返回了东西的，这个 `Src` 指针指向局部变量，很快就会被覆盖

所以这里的思路是，直接 `hook` `log_message` 函数，执行 `log_message` 函数，拿到返回值，复制走返回值的字符串，然后 `OutputDebugStringA` 打印调试信息

## 代码实现（核心）
定义函数原型定成返回 `char*` 来接收指针

接收到之后立马复制走里面的数据避免被覆盖，因为该指针指向的栈数组的生命周期已经结束了，数据随时有被覆盖的风险

然后这里的异常处理是为了应对一些特殊情况：

- 返回空，例如：`NULL`
- 返回非指针数据，例如：`1`

```
#include "pch.h"

#include "selHook.h"

typedef char*(__fastcall* _log_message)(

    int a1,

    __int64 a2,

    int a3,

    __int64 a4,

    const char* a5,

    const char* a6,

    __int128* a7,

    __int128* a8,

    __int128* a9,

    __int128* a10,

    __int128* a11,

    __int128* a12

    );

_log_message flog_message;

char* __fastcall Mylog_message(

    int a1,

    __int64 a2,

    int a3,

    __int64 a4,

    const char* a5,

    const char* a6,

    __int128* a7,

    __int128* a8,

    __int128* a9,

    __int128* a10,

    __int128* a11,

    __int128* a12

) {

    char* tmp = flog_message(a1, a2, a3, a4, a5, a6, a7, a8, a9, a10, a11, a12);

    __try {

        if (tmp == NULL) return NULL;

        int len = strlen(tmp);

        char log[0x1000];

        strncpy_s(log, tmp, len 
```

## 实现效果
使用 `debugview++` 接收调试信息，得到完整调试信息：
