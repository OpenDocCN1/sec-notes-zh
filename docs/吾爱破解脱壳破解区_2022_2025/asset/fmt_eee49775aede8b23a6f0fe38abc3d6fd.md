
# [原创] 用EDB（Linux下的ollydbg）分析破解某加密软件 （一）

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2047988
- 抓取时间: 2026-11-01T22:06:40

## 一、EDB的安装

之前写过一篇关于EDB的安装，版本是1.3.0，过程挺麻烦的，而且1.3.0版经常会有些显示不正常的问题，最近安装了下最新的1.5.0版，安装居然异常顺利，而且使用了一段时间，也很稳定，所以这里再简单分享下。

### 1. 准备工作，安装好需要用到的工具

以下为需要的工具列表（请注意版本，版本选得不好的话，可能后面会备受折磨，此处省略若干字......）

- 操作系统版本：CentOS Linux release 7.9.2009 (Core)
- `boost` => 1_75_0
- `gcc` => 9.2.0
- `cmake` => 3.12.1
- `capstone` => 4.0.1
- `graphviz` => 2.40.1
- `Qt` => 5.10.1

### 2. 设置下环境变量

（可以写入文件如`edb.setup.csh`，安装edb前，`source edb.setup.csh`）

```
setenv EDB_ROOT /YOUR_PATH/edb
setenv BOOSTDIR $EDB_ROOT/boost_1_75_0
setenv Boost_INCLUDE_DIR $BOOSTDIR/include/boost
setenv CMAKE_HOME $EDB_ROOT/cmake-3.12.1-Linux-x86_64
set path = ($CMAKE_HOME/bin $path)
setenv QMAKE_HOME $EDB_ROOT/Qt5.10.1/5.10.1
set path = ($QMAKE_HOME/gcc_64/bin $path)
################################################################
# settings below will make find capstone and libgvc
################################################################
setenv PKG_CONFIG_PATH $EDB_ROOT/capstone-4.0.1/lib/pkgconfig:/graphviz-2.40.1/lib/pkgconfig
setenv GCC_HOME /ux/Tools/GCC/9.2.0/x86_64/rhel7
set path = ($GCC_HOME/bin $path)
setenv LD_LIBRARY_PATH $EDB_ROOT/capstone-4.0.1/lib64:$GCC_HOME/lib64:$GCC_HOME/lib:$BOOSTDIR/lib:/usr/local/lib:$LD_LIBRARY_PATH
setenv CAPSTONE_INCLUDE_DIRS $EDB_ROOT/capstone-4.0.1/include
setenv CAPSTONE_LIBRARY_DIRS $EDB_ROOT/capstone-4.0.1/lib64
setenv GRAPHVIZ_INCLUDE_DIRS $EDB_ROOT/graphviz-2.40.1/include
setenv GRAPHVIZ_LIBRARY_DIRS $EDB_ROOT/graphviz-2.40.1/lib
setenv CPLUS_INCLUDE_PATH $EDB_ROOT/boost_1_75_0/include:$EDB_ROOT/graphviz-2.40.1/include:$EDB_ROOT/capstone-4.0.1/include
```

### 3. 安装edb

(a) 到下面网站下载最新版edb源文件 `edb-debugger-1.5.0.tgz`

https://github.com/eteran/edb-debugger

(b) 新建一个edb安装目录，解压 `tar zxvf edb-debugger-1.5.0.tgz`

(c) `cd edb-debugger`

(d) 修改文件

在文件 `src/arch/x86-generic/ArchProcessor.cpp` 中加入下面一行

```
#define __X32_SYSCALL_BIT 0x40000000
```

(e) `mkdir bin`

(f) `cd bin`，然后依次运行以下命令

```
souce edb.setup.csh
cmake ..
make
```
......

```
[100%] Linking CXX shared library ../../libHeapAnalyzer.so
[100%] Built target HeapAnalyzer
```

好了，大功告成。

## 二、用EDB分析某加密软件

### 1. 软件下载

https://pan.baidu.com/s/1LYZ-GwTzH0i7RcrFZ7ghNQ 提取码: 52pj

### 2. 引言

软件用途：加密某编程脚本的（因为脚本里有些信息需要保密），然后将加密后的脚本提供给终端用户使用。

破解背景：这款软件是十几年前下载的试用版本，license过期后只是简单修改了下软件中过期后的跳转指令，就一直用到现在，好好的都没什么问题。最近账户迁移，迁移后发现软件报License CRC错误，尝试将账户迁移回去，然后将软件从备份恢复，居然还是报错。原来的网站可以申请license，但是license格式有些变化（binary的），似乎不能用在旧版的软件上，而新版的加密软件要求使用新的操作系统，而我们的运行环境大部分还是比较老的操作系统，没办法，只能耐着性子去破解了。

### 3. 软件破解

先从下面的出错信息入手

`re` (`re`为程序文件名，不带参数执行软件)

```
License file CRC error.
......
Registration key: 3587C9F4FAF0316EC32488E937AE4D70
```

IDA载入文件`re`，String view里搜索`CRC error`，定位到如下位置

```
.text:08050A07 loc_8050A07: ; CODE XREF: sub_8050970+104↓j
.text:08050A07 C7 04 24 D8 37 05 08 mov dword ptr [esp], offset aLicenseFileCrc ; "License file CRC error."
.text:08050A0E E8 AD 82 FF FF call _puts
.text:08050A0E
.text:08050A13 8D 8D E8 FB FF FF lea ecx, [ebp+dest]
.text:08050A19 89 4C 24 0C mov [esp+0Ch], ecx
.text:08050A1D 8B 35 60 55 05 08 mov esi, ds:dword_8055560
```

edb载入程序在`08050A07`处下断点

![](https://attach.52pojie.cn/forum/202507/23/154936o3n88bhbee8eyw7c.jpg)

Snap1.jpg (198.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTI4M3w0NzY2YmJkMnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTI4M3w0NzY2YmJkMnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 15:49 上传

F8 run过，可以看到ASCII “License File CRC error.”

继续F8 run至`08050a1d`，查看地址`08055560`中存放的是`088de410` （记住这个地址`08055560`）

查看`088de410`处，如下图，看到`3587C9F4FAF0316EC32488E937AE4D70`了吧？就是前面的Registration key

![](https://attach.52pojie.cn/forum/202507/23/155515hmzzsz770i71l7d8.jpg)

Snap2.jpg (203.29 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTI5MXxkMDJlNDhmZHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTI5MXxkMDJlNDhmZHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 15:55 上传

好，我们Debug -> Restart 重新运行程序，前面的地址`08055560`还记得吧？在Data Dump区定位到这个地址，并下硬件写入断点，如下图

![](https://attach.52pojie.cn/forum/202507/23/180057kro88mudhovvobur.jpg)

Snap3.jpg (128.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTQwMXxiZGQyNjAwYXwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTQwMXxiZGQyNjAwYXwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 18:00 上传

F9运行断在下面的位置`0804C455`

![](https://attach.52pojie.cn/forum/202507/23/160319o6iw66ufu53s825f.jpg)

Snap4.jpg (41.02 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMwMHwwOTkyZDYzOHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMwMHwwOTkyZDYzOHwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:03 上传

看下Data Dump区看下`08055560`

![](https://attach.52pojie.cn/forum/202507/23/161005sfbucsd6klofza2c.jpg)

Snap5.jpg (46.36 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMwNnw0OTc5MDljYnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMwNnw0OTc5MDljYnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:10 上传

去`08f27410`看一下，Registration key已经出来了。

![](https://attach.52pojie.cn/forum/202507/23/161208m4yw3440n7c3cjwy.jpg)

Snap6.jpg (48.69 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMwN3xlY2Q0ODBhNnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMwN3xlY2Q0ODBhNnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:12 上传

IDA里看下`0804C455`附近的代码

```
.text:0804C445 loc_804C445: ; CODE XREF: sub_804C400+20↑j
.text:0804C445 89 04 24 mov [esp], eax ; filename
.text:0804C448 E8 C3 FE FF FF call sub_804C310
.text:0804C448
.text:0804C44D 83 F8 01 cmp eax, 1
.text:0804C450 A3 60 55 05 08 mov ds:dword_8055560, eax
.text:0804C455 19 C9 sbb ecx, ecx
.text:0804C457 0F B6 D1 movzx edx, cl
```

`sub_804C310`应该是关键call，在IDA里看下`sub_804C310`的Function Call

![](https://attach.52pojie.cn/forum/202507/23/161703nawonuo6oogufkpu.jpg)

Snap7.jpg (24.29 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMxMnwwOWI3ODQ1MnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMxMnwwOWI3ODQ1MnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:17 上传

很少的几个子程序，从后往前看下，edb里Debug -> Retart重新run，在`0804C3AB`处下断点

```
.text:0804C3AB 89 54 24 08 mov [esp+8], edx ; int
.text:0804C3AF E8 6C 18 00 00 call sub_804DC20
```

注意ESI里的内容`10237010`

![](https://attach.52pojie.cn/forum/202507/23/163036rny64fybrwffrw6f.jpg)

Snap10.jpg (9.54 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMxOHw5ZWI0Y2ZlM3wxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMxOHw5ZWI0Y2ZlM3wxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:30 上传

选中EDX，右键Follow In Dump，来到`ff9d0d70`，

![](https://attach.52pojie.cn/forum/202507/23/162337qsc86nm8an2hn86s.jpg)

Snap8.jpg (199.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMxM3wyYjRmYjIxYnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMxM3wyYjRmYjIxYnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:23 上传

![](https://attach.52pojie.cn/forum/202507/23/162533ypxvujl3saevai3p.jpg)

Snap9.jpg (8.74 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5MTMxNHwwMGI1NzQyNnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MTMxNHwwMGI1NzQyNnwxNzkzNTcwNzk4fDE2ODA4NnwyMDQ3OTg4&nothumb=yes)

2025-7-23 16:25 上传

按两次F8，run过`0804C3AF`停在`0804C3B4`

```
.text:0804C3AF E8 6C 18 00 00 call sub_804DC20
```
