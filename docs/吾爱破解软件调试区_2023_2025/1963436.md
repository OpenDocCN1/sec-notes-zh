
# [调试逆向] WinDbg调试（三）-- 空格引发的悬案

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1963436
- 抓取时间: 2026-11-01T21:53:01

## 空格引发的悬案

### 1、描述现象

在代码中有一段利用`rmdir`指令删除目录代码，但是有用户反馈一直删除失败，但是有没有看到错误的日志信息，正好有同事能复现,所以今天好好探究一番。

### 2、思考过程

很好奇的一点就是为什么有的环境就是正常。

首先想到2个问题:

①代码有没有执行。

②假如执行，有没有错误。

关于这两个问题都有个难点,我该如何下断点：

#### 2.1、分析代码是否执行

删除目录的代码其实利用Python代码执行`rmdir`命令行，伪代码如下：

```
subprocess.Popen('rmdir /S /Q %s' % to_del, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
```

第一个问题下断点的难点在于，这段代码是在python中，同事的是发布版本没法直接调试python代码。

第二个难点就是在python环境中又起了一个进程，这个临时进程生命周期很短，如何attach上去呢。

如果找不到合适的断点后续分析使将无法下手。

起身走到阳台，思考几分钟，即是python的启动命令行，那么势必会创建新的进程。既如此那我可以利用WinDbg在`CreateProcess`函数下断点，

在创建进程时有两个版本的函数`CreateProcessA`和`CreateProcessW`，在不确定是哪个函数时，这两个函数我们的都要下断点。

此时我们配置下远程调试，连接到同事的电脑上。

输入如下指令：

```
bm kernelbase!CreateProcess?

  0: 00007ffa`8d1a0870 @!"KERNELBASE!CreateProcessA"
  1: 00007ffa`8d1a07f0 @!"KERNELBASE!CreateProcessW"
```

确认是这个思路，启动exe。挂上WinDbg下个断点来一个触发操作。

果然收到断点事件，查看下堆栈信息，调用的unicode版本的函数：

```
0:000> k
 # Child-SP          RetAddr               Call Site
00 00000015`b55fd5c8 00007ffa`8e28cef4     KERNELBASE!CreateProcessW
01 00000015`b55fd5d0 00007ff9`ff1a7fe0     KERNEL32!CreateProcessWStub+0x54
02 00000015`b55fd630 00007ff9`ff1a3004     xxx!_winapi_CreateProcess_impl+0x300
```

下面栈省略。。。

从MSDN官网文档再看这个出的原型：

```
    creationResult = CreateProcess(
        NULL,                   // No module name (use command line)
        cmdLine,                // Command line
        NULL,                   // Process handle not inheritable
        NULL,                   // Thread handle not inheritable
        FALSE,                  // Set handle inheritance to FALSE
        NORMAL_PRIORITY_CLASS | CREATE_NEW_CONSOLE | CREATE_NEW_PROCESS_GROUP, // creation flags
        NULL,                   // Use parent's environment block
        NULL,                   // Use parent's starting directory 
        &startupInfo,           // Pointer to STARTUPINFO structure
        &processInformation);   // Pointer to PROCESS_INFORMATION structure
```

第二个参数就是命令行的参数，值放在`rdx`寄存器中，执行如下命令：

```
0:000> du rdx
0000028e`c4d2a310  "C:\windows\system32\cmd.exe /c ""
0000028e`c4d2a350  "rmdir /S /Q F:\work\project 中文\Loca"
0000028e`c4d2a390  "lData\BackUp\ProjectName001\wo"
0000028e`c4d2a3d0  "rkTD\workTD_backup_2024-08-30"
0000028e`c4d2a410  "-11-18-52 - 副本""
```

在这里下断点还有个小技巧，就是断点命中，让其自动打印参数：

```
0:000> bm kernelbase!CreateProcess? "du rdx"

breakpoint 0 redefined

  0: 00007ffa`8d1a0870 @!"KERNELBASE!CreateProcessA"

breakpoint 1 redefined

  1: 00007ffa`8d1a07f0 @!"KERNELBASE!CreateProcessW"
```

找到命令行参数果然就是我要删除的目录。

到这里排除第一个疑问，其实代码是执行。

#### 2.2、代码执行出错了？

接下来就是证明第2个问题：为什么执行行为不是我期待的。

这个进程是python启动，执行完进程就退出，我根本没有机会截停这个过程怎么办。于是我心生一计，我可以直接把启动进程的参数复制出来，直接在`CMD`中模拟执行，再把WinDbg挂在`cmd`过程中就可以。

那么新的问题来了，断点在哪里下。

在`CMD`中执行的命令是`rmdir`，我在想肯定是执行Win32API函数删除目录，去MSDN管网去找对应的API,果然找到一个`RemoveDirectory`来下个断点。

复制当时的目录，`F:\work\project 中文\LocalData\BackUp\ProjectName001\workTD\workTD_backup_2024-08-30-11-18-52 - 副本`，打开`cmd`命令行。

先用WinDbg attach上，下个断点，在这里我不确定调用的是哪个函数，所以都下了断点。后面就是打印参数，按照unicode和ASIIC方式打印。

```
0:004> bm *!RemoveDirectory? "du rcx;da rcx"
  0: 00007ffa`8d1cf3b0 @!"KERNELBASE!RemoveDirectoryW"
  1: 00007ffa`8d27e980 @!"KERNELBASE!RemoveDirectoryA"
  2: 00007ffa`8e295250 @!"KERNEL32!RemoveDirectoryA"
  3: 00007ffa`8e295260 @!"KERNEL32!RemoveDirectoryW"
  4: 00007ffa`8eac48e8 @!"SHELL32!RemoveDirectoryW"
```

在`cmd`中执行

```
rmdir F:\work\project 中文\LocalData\BackUp\ProjectName001\workTD\workTD_backup_2024-08-30-11-18-52 - 副本
```

WinDbg立马断点下来，这里初现端倪，文件路径被截断了。

```
0:004> g
000001cc`fbf1f4e0  "F:\work\project"
000001cc`fbf1f4e0  "F"
KERNELBASE!RemoveDirectoryW:
00007fff`5cf7f3b0 48895c2408      mov     qword ptr [rsp+8],rbx ss:00000056`7fd0f010=000001ccfbf17c56
```

看下堆栈信息：

```
0:000> k
 # Child-SP          RetAddr               Call Site
00 00000056`7fd0f008 00007ff6`a0f6aab0     KERNELBASE!RemoveDirectoryW
01 00000056`7fd0f010 00007ff6`a0f5c862     cmd!RdWork+0x197dc
02 00000056`7fd0f360 00007ff6`a0f5bea1     cmd!FindFixAndRun+0x242
03 00000056`7fd0f800 00007ff6`a0f6ebf0     cmd!Dispatch+0xa1
04 00000056`7fd0f890 00007ff6`a0f68ecd     cmd!main+0xb418
05 00000056`7fd0f930 00007fff`5dc07374     cmd!__mainCRTStartup+0x14d
06 00000056`7fd0f970 00007fff`5f49cc91     KERNEL32!BaseThreadInitThunk+0x14
07 00000056`7fd0f9a0 00000000`00000000     ntdll!RtlUserThreadStart+0x21
```

此时依旧是老套路，查看`KERNELBASE!RemoveDirectoryW`原型，发现第一个参数就是路径地址，此时发现路径已经被截断

```
0:000> du rcx
000001cc`fbf1f4e0  "F:\work\project"
```

至此可以分析出导致的原因，路径被截断。再仔细对比分析发现路径中存在了空格。

我们再接着往下分析，更加验证之前的猜想。

再从堆栈分析，当前的栈执行完之后会返回的地址`00007ff6a0f5c862`，可以反汇编看下`cmd!FindFixAndRun+0x242`执行逻辑：

```
0:000> u cmd!RdWork+0x197dc

cmd!RdWork+0x197dc:

00007ff6`a0f6aab0 0f1f440000      nop     dword ptr [rax+rax]

00007ff6`a0f6aab5 85c0            test    eax,eax

00007ff6`a0f6aab7 0f85a769feff    jne     cmd!RdWork+0x190 (00007ff6`a0f51464)

00007ff6`a0f6aabd 48ff15148b0100  call    qword ptr [cmd!_imp_GetLastError (00007ff6`a0f835d8)]

00007ff6`a0f6aac4 0f1f440000      nop     dword ptr [rax+rax]

00007ff6`a0f6aac9 33d2            xor     edx,edx

00007ff6`a0f6aacb 8bc8            mov     ecx,eax

00007ff6`a0f6aacd 8bd8            mov     ebx,eax
```

此时我们在`00007ff6a0f6aabd`下个断点，输入单步执行指令`g`，

```
0:000> g
Breakpoint 2 hit
cmd!RdWork+0x197e9:
00007ff6`a0f6aabd 48ff15148b0100  call    qword ptr [cmd!_imp_GetLastError (00007ff6`a0f835d8)] ds:00007ff6`a0f835d8={KERNELBASE!GetLastError (00007fff`5cf3b610)}
```

这里提示调用的`KERNELBASE!GetLastError`来查找错误。再执行`p`，单步执行一次，让这条语句执行完。

此时我们调用`!glt`，就可以获取到当前的错误码。

```
0:000> !gle

LastErrorValue: (Win32) 0x2 (2) - The system cannot find the file specified.

LastStatusValue: (NTSTATUS) 0xc0000034 - Object Name not found.
```

果然和上面的分析一致。

此时你设想下，要是你的路径下存在`F:\work\project`被删除了，那是多么可怕的事件。

### 3、解决方案

解决方案很简单，就是在路径上加上双引号就可以了。

```
subprocess.Popen('rmdir /S /Q \"%s\"' % to_del, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
```
