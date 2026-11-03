
# [原创] 代码寻洞, 从原位 Patch 到添加复杂代码——Windows 右键菜单魔改系列02

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1992132
- 抓取时间: 2026-11-01T22:17:51

## 1. 背景

这一篇原本打算是和上一篇写在一起的，奈何上一篇篇幅实在太长，我都担心有没有朋友会有耐心真正看完
没有看上一篇的朋友请移步[//www.52pojie.cn/thread-1992024-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1992024-1-1.html:href="https)
添加分隔符稍微复杂一些。需要用到代码找洞 (Code Caving) 以及重定向控制流到自己添加的代码。
上一篇文章中，@JieW_L 大佬提出了可以用注册表实现的方法，非常简洁方便，膜拜了。若使用注册表方案，只需要加一行`"SeparatorAfter"=""`即可解决，再次感谢大佬分享！(❁´◡`❁)

## 2. 修改思路

首先还是一样需要定位到我们的关键位置。即：第一个菜单项“新建文件夹”是什么时候被插入到我们的右键菜单中的？随后我们需要在它插入这个菜单项之后立马执行命令插入一个分隔符。
因为这两个操作都需要一个必要的参数，即右键菜单的句柄`hMenu`。这个参数相当于这个菜单的把柄，知道了这个就可以随意进行修改了。紧接着上一个操作执行就可以利用这个参数，也省得自己去找该在哪个位置插入分隔符。

## 3. 开始操作

### 3.1 定位关键位置

首先打开我们的反编译工具，打开位于`%ProgramFiles%\Bandizip\`下的`bdzshl.x64.dll`文件。这一次我们使用 `IDA`，当然 `Binary Ninja` 也是大差不差。各种工具都要熟悉一下嘛。
按`G`打开跳转到地址，然后输入`aTextMenuCreate`回车（即 "`TEXT_MENU_CREATE_NEW_FOLDER`" 字符串在程序里的标识符 `Label`）

![](https://attach.52pojie.cn/forum/202412/17/213553fd18md5mwdw87288.png)
2024-12-13-[16-04-59].png (299.5 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDMzOHwzMzY2ODIwY3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDMzOHwzMzY2ODIwY3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:35 上传

随后我们来到了这些字符串的老窝。在刚才对应的这个字符串上右键，选List cross references to... 可以找到这个字符串在程序中用到过的位置。双击跳转。

![](https://attach.52pojie.cn/forum/202412/17/213633rxwo68x89n58zzwk.png)
2024-12-13-[16-09-40]-裁剪.png (111.83 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDMzOXxjNDIwODg4OXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDMzOXxjNDIwODg4OXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:36 上传

![](https://attach.52pojie.cn/forum/202412/17/213649wyn1fz1mfqpf6mff.png)
2024-12-13-[16-09-56].png (25.74 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0NDM0MHw0YWE5ZTI5Y3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0MHw0YWE5ZTI5Y3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:36 上传

接下来我们便来到了和上一篇中差不多的位置。可以按一下空格关掉默认的流程图显示。特别注意：这时应立即按下`Alt+M`保存一个书签 不然待会儿你就很难找的回来这个位置啦ヾ(≧▽≦*)o

![](https://attach.52pojie.cn/forum/202412/17/213711s23ttnvli3kbkxzt.png)
2024-12-17-[16-49-21].png (266.93 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDM0MXwxODhiNjBiMHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0MXwxODhiNjBiMHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:37 上传

稍微往下一点点可以看到这个函数调用

![](https://attach.52pojie.cn/forum/202412/17/213743sikiq9xfkg3p2xkg.png)
2024-12-17-[16-52-43].png (362.37 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDM0MnwzZWQ5ZjZjMXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0MnwzZWQ5ZjZjMXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:37 上传

双击它进去，然后按`F5`，是不是豁然开朗？只能说 `IDA` 的 C 伪代码反编译是真的好用 (≧∇≦)ﾉ。这个函数的作用就是构造一个菜单信息结构体变量，然后调用 `User32.dll` 里的 [//learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-insertmenuitemw" target="_blank" rel="noopener noreferrer nofollow"](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-insertmenuitemw:rel="nofollow noopener" href="https)进行菜单(项)的创建及插入。
注意这个函数的参数传递。我们最关心的就是那个前面讲过的“菜单的把柄”`hMenu`，根据上方 `InsertMenuItemW` 函数的定义，我们可以分析出来这个`HMENU a2`就是我们要找的菜单句柄。它位于函数传入参数的第二个位置（记住这个后面要用）

![](https://attach.52pojie.cn/forum/202412/17/213805xog469m44on1oomc.png)
2024-12-17-[17-00-53].png (263.93 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0NDM0M3wyNWY4MTFlY3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0M3wyNWY4MTFlY3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:38 上传

随后，根据[//blog.csdn.net/m0_37567738/article/details/140840104" target="_blank" rel="noopener noreferrer nofollow"](https://blog.csdn.net/m0_37567738/article/details/140840104:rel="nofollow noopener" href="https)，第二个参数是放在`rdx`寄存器里传递的。因此，我们只需要看一下程序在`call`这个函数之前，把啥放到`rdx`里就好了。
按一下 `IDA View-A` 回到汇编视图，再按一下返回回到刚才的位置

![](https://attach.52pojie.cn/forum/202412/17/213818dezyeeka8jrykdoz.png)
2024-12-17-[17-08-27].png (347.73 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDM0NHwxOGRiN2QxN3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0NHwxOGRiN2QxN3wxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:38 上传

回来之后，可以发现（我这边是）`rdi`寄存器（版本不一样的话会不同），请记下这个寄存器

![](https://attach.52pojie.cn/forum/202412/17/213839ftqmcnhcbm7i7w7q.png)
2024-12-17-[17-11-12].png (365.19 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDM0NXxiZWM4NzY2NnwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0NXxiZWM4NzY2NnwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:38 上传

然后看到这个函数下面的这三行代码，这里就是我们需要调用函数插入分隔符的地方。但是要调用函数的话需要很多行代码，原来这个位置写不下。因此接下来需要进行代码找洞，找出`.text`段内空的位置，写上自己的代码，然后从原位置跳到这边，执行完再跳回去。

![](https://attach.52pojie.cn/forum/202412/17/213936rqpjxcolkkt0480c.png)
2024-12-17-[19-32-32].png (357 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0NDM0NnxlZTk0ZGVlZHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0NnxlZTk0ZGVlZHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:39 上传

先新开一个`notepad`然后拷贝上方这三行代码到里面保存（每次更新后可能会不一样）

```
MOV EDX, 0BBAh
MOV RCX, RSI
CALL sub_xxxxxxxx
```
②然后主窗口里选中这三行 右键 -> `NOP`（失误了的话可以撤销或者右键`Revert Patch`）
③随后找到`NOP`完下面的第一行后续的代码，右键`Rename`或者按`N`，输入`Label`标签，这里我输入的是`createNewFolder`，随便啥都行

### 3.2 代码寻洞

代码找洞的含义非常简单，就是找到足够长的一段连续的空的区域来写自己的代码。可以使用菜单中 `Search -> Sequence of bytes...` 来查找连续的空字节，当然绝大多数情况下 `.text` 段末尾都会留一些，和开头那些空的字节不同，开头那些可能是变量定义，结尾这些一般都是真正空的。菜单 `View -> Open subviews -> Segments`，然后`.text`下面第一个这个点两下，然后往上滚动，就可以找到 `.text`（代码）段的结尾。

![](https://attach.52pojie.cn/forum/202412/17/214000ehzdfla7ph88hgg7.png)
2024-12-13-[16-53-32].png (259.57 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0NDM0N3w5M2VhMWUyOXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0N3w5M2VhMWUyOXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)
2024-12-17 21:40 上传

点一下`align 200h`，然后 `Edit -> Data` 点一下，可以空几行，然后点菜单 `Edit -> Patch program -> Assemble...` 就可以开始写汇编代码了。注意要从上往下写，因为很多命令都是多个字节长度的，然后汇编的话是从光标位置向后覆盖。从下往上很难估计长度。
在 Patching 窗口中从上往下写入如下代码（每行写完按回车，全部写完再关闭）：

```
LEA RAX, QWORD PTR [NewItem]

MOV [RSP+20h], RAX

XOR R9D, R9D

MOV R8D, 0C00h

MOV EDX, 1

MOV RCX, RDI  (此处和上方第2步①对应，输入的是菜单的句柄)

CALL QWORD PTR [InsertMenuW]
```
然后接着写上刚才保存在`notepad`里的代码：
