
# [原创] Glibc 2.23 Ptmalloc源码分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990322
- 抓取时间: 2026-11-01T22:18:12

## 1.背景：

因为最近想复习一下PWN针对于heap的知识，发现很多地方都看不太懂了，所以回过头来继续再分析一下这个`glibc2.23`中，针对于`patmalloc`的`malloc`源码部分。

在阅读本文章前，请各位准备好一份在线源码，因为本篇文章属于是一镜到底形式的分析，并未做任何的分割。其中可能还需要对其中遇到的一些`arena`和`chunk`的结构体并未做说明，这个后续在做补充吧。

## 2.环境准备：

系统：`Ubuntu 16.04`

调试器: `Pwndbg`

gcc版本：5.4.0

### 2.1 编译带源码信息的glibc：

这里主要参考网上文章`https://gist.github.com/stefan1wan/5e4b3973aae578ac39f94d30a5555f19`中的步骤。编译有问题的直接看在线文档也可以：`https://elixir.bootlin.com/glibc/glibc-2.23/source/malloc/malloc.c`

#### 2.2 下载glibc源码文件

glibc2.23源码：`http://ftp.gnu.org/gnu/glibc/glibc-2.23.tar.gz`

#### 2.3 编译64位的glibc源码：

```
tar -zxvf glibc-2.23.tar.gz
cd glibc-2.23
mkdir build_x64 \
cd build_x64 \
mkdir glibc_x64 \
CFLAGS="-g -g3 -ggdb -gdwarf-4 -Og -Wno-error" \
CXXFLAGS="-g -g3 -ggdb -gdwarf-4 -Og" \
../configure --prefix=/home/g0fl1er/Desktop/glibc-2.23/build_x64/glibc_x64
make -j4
make install
```

### 2.4 编译32位的glibc源码：

```
tar -zxvf glibc-2.23.tar.gz
cd glibc-2.23
mkdir build_x32
cd build_x32
mkdir glibc_x32
CC="gcc -m32 -Wno-error=attributes" \
CXX="g++ -m32" \
CFLAGS="-g -g3 -ggdb -gdwarf-4 -Og -Wno-error" \
CXXFLAGS="-g -g3 -ggdb -gdwarf-4 -Og" \
../configure --prefix=/home/g0fl1er/Desktop/glibc-2.23/build_x32/glibc_x32/ --host=i686-linux-gnu
make -j4
make install
```

完成后就可以在各个glibc的文件夹下看到

![](https://attach.52pojie.cn/forum/202412/11/153525txoxr3urp5rxapcd.png)

2.4.png (44.98 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkxOXwyZjUzZWU0M3wxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkxOXwyZjUzZWU0M3wxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

### 2.5 Patchelf替换：

这里直接替换我们刚才装好的

```
patchelf --set-interpreter /home/admin/Desktop/glibc-2.23/build_x64/glib64/lib/ld-linux-x86-64.so.2 --set-rpath /home/admin/Desktop/glibc-2.23/build_x64/glib64/lib/ 1
```

检查之后发现没有错误

![](https://attach.52pojie.cn/forum/202412/11/153533paxya8y8xup88ddz.png)

2.5.PNG (37.29 KB, 下载次数: 2)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyMXwzMDAxYmQxMHwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyMXwzMDAxYmQxMHwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

## 3.malloc

源码：

```
#include 
#include 
void main()
{
    char *a = (char *)malloc(20);
}
```

编译：

```
gcc 1.c -o 1
```

### 3.1 开始分析：

```
gdb -q 1
b main
r
```

![](https://attach.52pojie.cn/forum/202412/11/153535mj3aom6odtcwwd0o.png)

3.1.0.PNG (249.03 KB, 下载次数: 0)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyMnw1ODE5NTc2ZXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyMnw1ODE5NTc2ZXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

一直ni走到call malloc这句话的地方，然后ni进入到他的那个函数内部

![](https://attach.52pojie.cn/forum/202412/11/153537h92jcw2wlzwcj7aa.png)

3.1.1.png (88.03 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyM3w3NTRmY2Q1MXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyM3w3NTRmY2Q1MXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

一路ni，走过这个`dl_resolve`的过程，直接到他的`malloc`的开始，也就是到`__libc_malloc`这个函数这里。

![](https://attach.52pojie.cn/forum/202412/11/153539n1ydzgq1gg2a9t73.png)

3.1.2.png (118.67 KB, 下载次数: 0)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyNHxmNzhkN2YwNXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyNHxmNzhkN2YwNXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

首先可以看到`malloc`的入口会进入到`__libc_malloc`函数进行，接着会调用`atomic_forced_read`函数对`__malloc_hook`变量的值进行读取

![](https://attach.52pojie.cn/forum/202412/11/153541x7umkmhuj7o30h9m.png)

3.1.3.png (116.44 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyNXxiYTExOGZhMnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyNXxiYTExOGZhMnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

此时的`__malloc_hook`是有值的,并且读取到的值是`malloc_hook_ini`

![](https://attach.52pojie.cn/forum/202412/11/153544gubdmilkouhht9mh.png)

3.1.4.png (7.99 KB, 下载次数: 2)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyNnxhNGEzN2E3ZnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyNnxhNGEzN2E3ZnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

接着会判断`__malloc_hook`是否为空，如果不为空，则会将`__malloc_hook`的值当做函数名称进行执行。这也就是为什么发生任意地址写的时候，为什么要修改`__malloc_hook`这个地址的值的原因了。

![](https://attach.52pojie.cn/forum/202412/11/153546e59q9fuj0j4f2f2q.png)

3.1.5.png (52.77 KB, 下载次数: 0)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyN3w0YWM2ZWFlMnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyN3w0YWM2ZWFlMnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

这里第一次在调用`malloc`的时候`malloc_hook`的值为`malloc_hook_ini`，所以这里会调用`malloc_hook_ini`函数。

![](https://attach.52pojie.cn/forum/202412/11/153548jgoeckvcgtkc0cut.png)

3.1.6.png (126.22 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyOHxkZjVjNDU0OXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyOHxkZjVjNDU0OXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

进入到`malloc_hook_ini`这个函数内部可以看到具体的内容，首先会将`__malloc_hook`的值设置为空，避免下次调用`malloc`时，会造成重复调用。接着会调用`ptmalloc_init()`函数，这个函数是用于初始化`ptmalloc`的，最后返回到`malloc`函数的`__libc_malloc`入口处，并且将这次`malloc`的`size`作为参数，相当于再次调用`malloc`函数。

![](https://attach.52pojie.cn/forum/202412/11/153550mvpivkgp3pggff6k.png)

3.1.7.png (43.22 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyOXwxYWFjYjQxZHwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkyOXwxYWFjYjQxZHwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

初始化完毕`ptmalloc`后，将回到`malloc`入口函数继续执行一遍上述的内容，因为这里经过了第一次的`__malloc_hook_ini`函数的清空`_malloc_hook`的值，所以不会再调用。

![](https://attach.52pojie.cn/forum/202412/11/153553c6bg67zeghgzzgb4.png)

3.1.8.png (6.78 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkzMHxhNzg4NGE0Y3wxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkzMHxhNzg4NGE0Y3wxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

![](https://attach.52pojie.cn/forum/202412/11/153555v1gil8llg15wsgaz.png)

3.1.9.png (118.68 KB, 下载次数: 0)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkzMXwyNTQzNGY1MnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkzMXwyNTQzNGY1MnwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

继续向下走，会进入到`arena_get`函数。

这个函数的会对`arena`进行加锁操作，既线程互斥锁`mutex`，以防止后续存在多个线程同时`malloc`时产生冲突。`arena_get`函数还会检测`arena`对象是否为空并且`flags`标志的第四个`bit`是否不为空，如果有一个不满足，则从系统中获取一个`arena`用于后续的`heap`的申请使用。

![](https://attach.52pojie.cn/forum/202412/11/153557q2laqp8aiaarca8x.png)

3.1.10.png (122.64 KB, 下载次数: 1)

[https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkzMnw1YmFiNGVjNXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjkzMnw1YmFiNGVjNXwxNzkzNTcxNDkwfDE2ODA4NnwxOTkwMzIy&nothumb=yes)

2024-12-11 15:35 上传

![](https://attach.52pojie.cn/forum/202412/11/153559d6ple5np15n3nac5.png)

3.1.11.png (6.15 KB, 下载次数: 2)
