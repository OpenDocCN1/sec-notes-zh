
# DLL 开发与使用示例

```cpp
// MyDll.cpp: implementation of the MyDll class.

#include "stdafx.h"
#include "MyDll.h"

_stdcall int add(int x,int y)
{
    return x+y;
}

_stdcall int sub(int x,int y)
{
    return x-y;
}
```

这样就写好了一个`dll`

`extern`就相当于是向别人说明我提供这个函数，来使用我啊，那别的模块怎么使用并且知道这个模块的函数，有几个函数呢？

这里就涉及`PE`知识里的导入导出表了，通常情况下，`exe`有导入表，`dll`导入导出都有，但是不绝对，`exe`也可以有导入表。别的模块就可以通过这些表来知道你有多少函数，分别是啥等

我们用一个工具来看看我们写的`dll`的函数

![](https://attach.52pojie.cn/forum/202405/24/001523u106s4cbcmtblz61.png)

image-20240511214646150.png (20.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU0Nnw5YTNjYTdlOXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU0Nnw5YTNjYTdlOXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:15 上传

随便查看`PE`文件都有查询功能，看到这个导出函数，有两个，所以从导出表就能看到提供了那些函数。这个函数名不是我们想要的，有点难看话，可以自己定义函数名

`extern`那部分删掉，留下函数的声明，新建一个文本文件，后缀改成`.def`

![](https://attach.52pojie.cn/forum/202405/24/001536ncprbbz79bez9jlu.png)

image-20240511215151473.png (24.75 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU0N3wzMWEyOWMxZXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU0N3wzMWEyOWMxZXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:15 上传

在`.def`上写上

也是固定格式

```
EXPROTS

函数名     @编号

函数名     @编号     NANAME
```

```
//B.def
EXPORTS

add     @12
sub     @13     NONAME
```

所以函数就有两种方式导出，一个是函数名，一个是编号

![](https://attach.52pojie.cn/forum/202405/24/001549fjma7ugngaxybb7y.png)

image-20240511220531756.png (19.76 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU0OHxlMmY4MzcxNXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU0OHxlMmY4MzcxNXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:15 上传

（一定要写对英文单词啊，就因为写错了，又找了一会儿错误QAQ）

看到上面一个有名字，一个没名字，要用的话也可以用序号，为什么要设置没有名字的呢？

对于一个大项目来说，一个函数的名字就能看出这个函数是在干啥了，如果你胡乱起名，其他人接手或者其他人使用的时候不利于交接，所以，为了不改函数名的条件下，但也不想让人知道这个函数做了什么，就可以用这个`NONAME`，但是对于懂底层的人来说，隐不隐藏都一样，切到汇编看一下就知道这函数做了什么，所以这招是防君子不防小人

## 使用 dll

首先先把`dll`复制到测试的工作路径下

然后根据步骤来写

```cpp
//步骤1：定义函数指针

typedef int(_stdcall *lpadd)(int,int);

typedef int(_stdcall *lpsub)(int,int);

//步骤2：声明函数指针变量

lpadd myadd;

lpsub mysub;

//步骤3：动态加载dll到内存中

HINSTANCE hDll = LoadLibrary("*.dll");

//步骤4：获取函数地址

myadd = (lpadd)GetProcAddress(hDll,"add");

mysub = (lpsub)GetProcAddress(hDll,(char*)0xD);

//步骤5：调用函数

int a = myadd(10,2);

int b = mysub(10,2);

//步骤6：释放动态链接库

FreeLibrary(hDll);
```

```cpp
//Test.cpp

#include 
#include 

typedef int(_stdcall *lpAdd)(int,int);
typedef int(_stdcall *lpSub)(int,int);

lpAdd myAdd;
lpSub mySub;

int main()
{
    HINSTANCE hDll = LoadLibrary("B.dll");//加载dll

    myAdd = (lpAdd)GetProcAddress(hDll,"add");
    mySub = (lpSub)GetProcAddress(hDll,(char*)0xD);

    int a = myAdd(1,2);
    int b = mySub(5,2);

    FreeLibrary(hDll);
    return 0;
}
```

注意调用约定一定要一致，不然会出现栈区混乱的问题

这些`API`感兴趣的也可以自己查查，这里就不多介绍了

## 隐式链接

在上一章，我们用代码使用了`dll`，这个方式我们称为显式链接，优点就是非常灵活，只用一个`dll`文件，缺点就是比较麻烦，需要一个一个的去获取函数的地址

隐式链接的优点就是只要配置完成，就可以直接调用函数

先去把上一章的`dll`复制一下，我这里把两个函数都设置成有名字了

![](https://attach.52pojie.cn/forum/202405/24/001608ypp065lk0nbl6fcp.png)

image-20240512230450092.png (19.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU0OXw2MmFlMjc3OXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU0OXw2MmFlMjc3OXwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:16 上传

在`dll`的项目里找到`dll`和`lib`这两个文件，在显式链接中，只需要`dll`就够了，但在隐式链接中则还需要`lib`文件，这个跟静态库的`lib`文件不同，这里存的都是些辅助信息，编译器需要这些辅助信息来找到函数在哪

把这两个文件复制到测试的项目里，也可以放到库目录里，配置一下就就可以了。这是第一步

第二步就是添加辅助信息把

第三步是声明函数

```cpp
// 隐式链接Test.cpp

#include "stdafx.h"

//第二步：添加辅助信息

#pragma comment(lib,"B.lib")

//第三步：声明函数

_declspec(dllimport) _cdecl int add(int x,int y);

_declspec(dllimport) _cdecl int sub(int x,int y);

int main(int argc, char* argv[])

{

    int a = add(2,3);

    int b = sub(5,2);

    return 0;

}
```

![](https://attach.52pojie.cn/forum/202405/24/001628bl55onvovtz5dq60.png)

image-20240512231219952.png (871 Bytes, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU1MHw2YjgwYmFlY3wxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU1MHw2YjgwYmFlY3wxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:16 上传

也是没有问题

配置好后可以直接使用函数，不用再去获取函数地址，因为在辅助信息中得到过了

所以可以根据别人的给的文件而来自由选择链接的方式

看到这个代码你可能会觉得这跟静态太库很相似，那就来看看汇编

![](https://attach.52pojie.cn/forum/202405/24/001642kd0f8737qe7q7q6q.png)

image-20240512231742268.png (14.97 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU1MXw4ZTllNWZiM3wxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU1MXw4ZTllNWZiM3wxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:16 上传

看这里并不是直接调用而是间接调用，调用的是一个固定的内存位置

也就是

```
call 0x40101234    这就是直接调用，这个地址就是函数的地址
call dword ptr [__imp_?add@@YAHHH@Z (0042a190)]     而这个就只是间接调用，每当加载dll获取函数地址时，这个地址就会存进42a190这个地址，中间有个过度
```

所以隐式链接并没有把函数写进`exe`文件里

还需要注意的是，如果使用的`extern "C"`的指令的方式来声明函数，那么在这儿的第三步也要加进去，调用约定还是要保持一致

## 如何实现的

在上一章我说到了导入导出表，当我们把上面的代码编译运行，编译器就会生成导入导出表

![](https://attach.52pojie.cn/forum/202405/24/001656vdyyw99r9ldr8t4t.png)

image-20240512233222085.png (17.78 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU1MnwwOGY5M2VmNnwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU1MnwwOGY5M2VmNnwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:16 上传

可以看到`B.dll`已经加载进去了，使用隐式链接的时候就导进去了，这个导入表就会详细记录这个`exe`需要哪些`dll`，那些函数

所以使用隐式链接的话，系统会帮我们使用`LoadLibrary`和`GetProceAddress`，因此显式也好，隐式也罢，实现的方法一样，只是一个是我们自己动手，一个系统帮忙

## DllMain函数

```cpp
BOOL WINAPI DllMain(
  HINSTANCE hinstDLL,  // handle to the DLL module
  DWORD fdwReason,     // reason for calling function
  LPVOID lpvReserved   // reserved
);
```

在平常的`exe`代码中，会有`main`函数，也叫主函数，也是整个程序的入口，然后跑到`return`就停止了

但是`DllMain`不一样，不一定只执行一次就结束了

先介绍参数

### hinstDLL

用来知晓`dll`被加载到了什么位置，一个`dll`可能会被许多的进程加载，比如说`A`进程把`dll`加载到了`10000`的位置，`B`进程把`dll`加载到了`20000`的位置，所以就是用来得知把`dll`加载到了什么位置

### fdwReason

这个`dll`被调用的原因，根据以下的情况分类

![](https://attach.52pojie.cn/forum/202405/24/001714m02ayqplnp11qq2z.png)

image-20240512235435601.png (72.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5NzU1M3w0NmUyMDkyZnwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NzU1M3w0NmUyMDkyZnwxNzkzNTcwMDMxfDE2ODA4NnwxOTI3NTg5&nothumb=yes)

2024-5-24 00:17 上传

这里提到了四种，那么就有四种情况被调用，那么就来一个个分析

#### DLL_PROCESS_ATTACH

> 
Indicates that the DLL is being loaded into the virtual address space of the current process as a result of the process starting up or as a result of a call to `LoadLibrary`. DLLs can use this opportunity to initialize any instance data or to use the `TlsAlloc` function to allocate a thread local storage (TLS) index.

这表示`DLL`正在被加载到当前进程的虚拟地址空间中，这是由于进程启动或调用`LoadLibrary`函数的结果。`DLL`可以利用这个机会来初始化任何实例数据，或者使用`TlsAlloc`函数来分配一个线程本地存储（TLS）索引。

这个是首次使用`LoadLibrary`时，会得到这个宏

后面的代码会实验

#### DLL_PROCESS_DETACH

> 
Indicates that the DLL is being unloaded from the virtual address space of the calling process as a result of unsuccessfully loading the DLL, termination of the process, or a call to `FreeLibrary`. The DLL can use this opportunity to call the `TlsFree` function to free any TLS indices allocated by using `TlsAlloc` and to free any thread local data. 

这表明`DLL`（动态链接库）正在从调用进程的虚拟地址空间中卸载，这可能是因为`DLL`加载失败、进程终止，或者调用了`FreeLibrary`函数。`DLL`可以利用这个机会调用`TlsFree`函数，以释放通过`TlsAlloc`分配的所有`TLS`（线程局部存储）索引，并释放任何线程本地数据。

这里是调用了`FreeLibrary`就会传进这个宏，但是后面的实验是，只输出了一个，所以推断只是最后一次使用`FreeLibrary`才会传这个宏

#### DLL_THREAD_ATTACH

>
