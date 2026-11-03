
# [分享] 利用detours库（含编译教程）进行`loadlibrary`函数的hook，并输出监控日志

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2050481
- 抓取时间: 2026-11-01T22:06:03

这是过程中用得到的所有代码文件，网盘自取

包含编译完成的detours库源代码和Visual Studio的一整个hooking项目

链接: https://pan.baidu.com/s/1z5Lr20ss1Os1VjLWq6HzfA?pwd=2apb 提取码: `2apb`

detours官方源码可以在 https://github.com/microsoft/Detours/releases/tag/v4.0.1 下载

以下教程截图量少，因为是做的时候操作数量过多（试错），没有留存截图，大家将就着看吧

detours编译参考（不是搬运或转载）了这位大神的教程 https://blog.csdn.net/weixin_41245990/article/details/139401667，不能完全适用，后面一些报错主要是结合AI解决的。以下是他的版权声明

![](https://static.52pojie.cn/static/image/smiley/laohu/laohu6.gif)

版权声明：本文为博主（那位大神）原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。

原文链接：https://blog.csdn.net/weixin_41245990/article/details/139401667

## 编译准备

首先我们把官网上下下来的源码解压

要在命令行中使用`nmake`，需要将`nmake.exe`所在目录添加到系统环境变量中

由于Visual Studio安装方式的不同，每个人的`nmake.exe`文件位置有所不同（主要是前面的部分）。这里贴出两种供大家参考找到自己的`nmake.exe`位置

教程里那位大神的位置为`D:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Tools\MSVC\14.40.33807\bin\Hostx64\x64\nmake.exe`

我的位置为`"D:\Visual Studio\VC\Tools\MSVC\14.43.34808\bin\Hostx64\x64\nmake.exe"`

把地址去掉`nmake.exe`，增加到环境变量中，我们就可以在命令行使用`nmake`了

打开解压后源码所在文件夹，在文件地址栏输入`cmd`，打开`cmd`终端，输入`nmake`

出现报错：

1. `fatal error C1034: windows.h: 不包括路径集`

解决：打开`build`目录，即Visual Studio下 `VC\Auxiliary\Build` 目录，找到`vcvars`系列文件

如果要在32位系统下生成32位代码，就执行`vcvars32`

如果要在64位系统下生成64位代码，就执行`vcvars64`

如果要在64位系统下生成32位代码，就执行`vcvarsall amd64_x86`

如果要在32位系统下生成64位代码，就执行`vcvarsall x86_amd64`

具体执行方法：把文件拖到`cmd`窗口里执行。完成后输入`nmake`编译

2. 此时可能在结尾出现报错

```
D:\Detours-4.0.1\Detours-4.0.1\samples\traceapi\_win32.cpp(4323): error C2065: “GetThreadLocale”: 未声明的标识符D:\Detours-4.0.1\Detours-4.0.1\samples\traceapi\_win32.cpp(7050): error C2065: “SetThreadLocale”: 未声明的标识符NMAKE : fatal error U1077: “cl /nologo /Zi /MT /Gm- /W4 /WX /Od /I..\..\include /Fdobj.X64\vc.pdb /Foobj.X64\ /c trcapi.cpp ”: 返回代码“0x2”Stop.NMAKE : fatal error U1077: “"D:\Visual Studio\VC\Tools\MSVC\14.43.34808\bin\HostX64\x64\nmake.exe" /NOLOGO /L ”: 返回代码“0x2”Stop.NMAKE : fatal error U1077: “"D:\Visual Studio\VC\Tools\MSVC\14.43.34808\bin\HostX64\x64\nmake.exe" /NOLOGO / ”: 返回代码“0x2”Stop.
```

解决方案：修改`makefile`文件，位于`detours`目录下` samples\traceapi`目录下（注意不是`detours`目录下那个裸露的`makefile`，那个是顶层`makefile`）

在首行`!include ..\common.mak`下面、`all：dirs\`上面添加

```
CFLAGS=$(CFLAGS) /D_SILENCE_WIN32_DEPRECATION_WARNINGS /D_WIN32_WINNT=0x0501
```

```
LIBS=$(LIBS) kernel32.lib gdi32.lib user32.lib shell32.lib advapi32.lib ole32.lib ws2_32.lib
```

回到`detours`目录，输入`nmake clean`，清理后用`nmake`重新编译，问题解决。

## Visual Studio 2022 配置

接下来在`VS2022`中进行有关配置以使`detours`库能被正常调用。可以参考链接中的帖子，他是正常编写可执行文件情况下的。

我是在编写`.dll`文件，其配置不确定是否可以沿用，亦不知我的配置能否用于可执行文件情况下。请读者自行分辨，且欢迎评论区指点

以下是`dll`文件的编写及有关配置过程：

首先创建一个空项目（不要选`dll`，因为我们不需要使用预编译头），创建源文件新建项`dllmain.cpp`。点击上方工具栏中项目选项，点击最底端属性选项，进入项目属性页，在此处“配置类型”点击右边那个`v`形按钮，选择动态库（不要自己输入）

包含目录设置（根据你的具体`include`目录位置调整，且只需增加蓝条内容，其他不管，下同），未配置出现报错：`#include "detours.h"`时找不到文件

库目录设置，未配置报错：无法打开文件“`detours.lib`”

![](https://attach.52pojie.cn/forum/202508/02/110045qywgw29g1hglu0d1.png)

屏幕截图 2025-08-02 110026.png (195.69 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc5Mzk4OHwwNmMwNWU0NnwxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mzk4OHwwNmMwNWU0NnwxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes)

2025-8-2 11:00 上传

然后编辑预处理器定义，将原有的内容全部替换为以下内容：`WIN32;_WINDOWS;_USRDLL;DLLMONITORHOOK_EXPORTS;DETOURS_SOURCE`

![](https://attach.52pojie.cn/forum/202508/02/120907kdkpk8yqyiabuab1.png)

屏幕截图 2025-08-02 120803.png (126.32 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc5Mzk5NXw1NjhlNGZhNHwxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mzk5NXw1NjhlNGZhNHwxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes)

2025-8-2 12:09 上传

编辑附加依赖项如图

![](https://attach.52pojie.cn/forum/202508/02/121025gzpbmb141uobzagp.png)

屏幕截图 2025-08-02 121000.png (147.54 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5Mzk5Nnw5ZTQyMzc3MHwxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mzk5Nnw5ZTQyMzc3MHwxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes)

2025-8-2 12:10 上传

最后配置子系统（注意还是点右边`v`形按钮选择，不要自己输入）

![](https://attach.52pojie.cn/forum/202508/02/121304av6vgvggo8nymptf.png)

屏幕截图 2025-08-02 121247.png (136.5 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc5Mzk5N3w1Njc2NWM5Y3wxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5Mzk5N3w1Njc2NWM5Y3wxNzkzNTcwNzYxfDE2ODA4NnwyMDUwNDgx&nothumb=yes)

2025-8-2 12:13 上传

[C++] 纯文本查看 复制代码
