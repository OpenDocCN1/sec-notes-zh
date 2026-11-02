
# [分享] OLLVM混淆环境搭建与去平坦化

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1880845
- 抓取时间: 2026-11-01T22:39:05

之前遇到了很多次 OLLVM 平坦化的题目，所以想学习一下，参照了很多前人的方法，特此记录一下。

## 环境

VMware 虚拟机，Ubuntu 20.04

## OLLVM 定义

LLVM（Low Level Virtual Machine）是一个开源的编译器基础架构，它包含了一组模块化、可重用的编译器和工具，支持多种编程语言和目标架构，包括 x86、ARM 和 MIPS 等。LLVM 的核心思想是将编译器分为前端和后端两个部分，前端负责将源代码转换为中间表示（IR），后端负责将中间表示转换为目标机器的汇编代码。这种设计使得 LLVM 可以支持多种编程语言，因为只需要为每种语言编写一个前端，就可以利用后端的通用性支持多种目标架构。

OLLVM（Obfuscator-LLVM）是瑞士西北应用科技大学安全实验室于 2010 年 6 月份发起的一个项目，这个项目的目标是提供一个 LLVM 编译套件的开源分支，能够通过代码混淆和防篡改，所使用的编译器是 `clang`。

## 搭建 OLLVM 环境

### 下载 OLLVM 源码

OLLVM GitHub 仓库：

https://github.com/obfuscator-llvm/obfuscator

git 下载 OLLVM 源码：

```
git clone -b llvm-4.0 --depth=1 https://github.com/obfuscator-llvm/obfuscator.git
```

### 配置编译工具

配置 `cmake`、`gcc`、`g++`。

安装 `cmake`。可以去官网下载 https://cmake.org/download/ 或者直接使用 `sudo` 安装：

```
sudo apt install cmake
```

安装 `gcc-8`、`g++-8` 降低版本。查看 `gcc` 和 `g++` 版本，若为 9 则要降低版本。

![image](https://attach.52pojie.cn/forum/202401/15/175040ttt3giq4uyt5ratu.png)

安装 `gcc-8` 和 `g++-8`：

```
sudo apt-get install gcc-8 g++-8 -y
```

配置软件的优先级，可以根据需要去选择默认的版本：

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 8
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 9
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 9
```

### 切换 gcc 和 g++ 版本

默认 `gcc` 8：

```
sudo update-alternatives --config gcc
```

输入选择版本的数字。

![image](https://attach.52pojie.cn/forum/202401/15/175042k07w3vep0m39p0to.png)

默认 `g++` 8：

```
sudo update-alternatives --config g++
```

![image](https://attach.52pojie.cn/forum/202401/15/175044icere3jrcqoep3r3.png)

### 修改 OLLVM 源码

进入 `ollvm目录/include/llvm/ExecutionEngine/Orc/OrcRemoteTargetClient.h`，定位到第 690 行，把 `char` 改成 `uint8_t`。

在 `config.guess` 文件中存在一些 DOS 行结尾（DOS line endings），需要将其转换为 Unix 行结尾：

```
sudo apt-get install dos2unix
dos2unix obfuscator/cmake/config.guess
```

![image](https://attach.52pojie.cn/forum/202401/15/175047bca4ls7yd3yk738l.png)

### 编译 OLLVM

```
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_INCLUDE_TESTS=OFF ../obfuscator/
```

![image](https://attach.52pojie.cn/forum/202401/15/175049qcwvwnd39sivzszq.png)

```
make -j4
```

这个过程会很久。

```
[ 97%] Built target llvm-objdump
[ 97%] Linking CXX executable ../../bin/llvm-mc
[ 97%] Built target llvm-mc
[ 97%] Linking CXX executable ../../bin/llvm-dwp
[ 97%] Linking CXX executable ../../bin/llvm-lto2
[ 97%] Built target llvm-dwp
[ 97%] Built target llvm-lto2
make: *** [Makefile:152: all] Error 2
```

可能是环境配置出了问题，编译到 97% 时就停止了，几个主要的 `clang` 文件都没编译成功。试了好几次都卡在这个。

![image](https://attach.52pojie.cn/forum/202401/15/175051uysc32qsuy3jy6yc.png)

查阅资料，猜测是 swap 区空间不够，增加 swap 区空间为 4g，扩大虚拟机容量为 60g。如果卡住了，不要慌，不要删，继续 `make -j`。

![image](https://attach.52pojie.cn/forum/202401/15/175053rwxg3hxhg3xtdwws.png)

### 混淆

两种方法，一种是配置 NDK 环境混淆，另一种是直接将 `clang` 作为程序进行混淆。推荐第二种，第一种无法控制混淆的方法。注意，混淆时一定要采用 `gcc8` 和 `g++8`，9 以上的环境会混淆失败。

### 配置 NDK 环境

执行命令，打开配置文件：

```
gedit ~/.bashrc
```

把下面这两行粘贴到文件末尾，注意替换路径成自己下载的 NDK 路径：

```
export NDK_HOME=/home/bag/android/android-ndk-r16b/
export PATH=$NDK_HOME:$PATH
```

最后执行命令，使配置文件生效：

```
source ~/.bashrc
```

复制编译好的 4 个 `clang` 文件到 NDK 目录：`clang`、`clang++`、`clang-4.0`、`clang-format`，粘贴到 NDK 目录：

`ndk目录/toolchains/llvm/prebuilt/linux-x86_64/bin`

复制 build 目录的 3 个头文件到 NDK 目录：

```
fatal error 'Stddef.h' file not found
fatal error 'Stdarg.h' file not found
fatal error '__stddef_max_align_t.h' file not found
```

到 `android-ndk-r16b/sources/cxx-stl/system/include`

新建 `jni` 文件夹（名字改了的话在对应的配置文件得该指定路径方式）。在文件夹中新建 `Android.mk`、`Application.mk`、`try.cpp`。

`Android.mk`：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE := bag
LOCAL_SRC_FILES := try.cpp

include $(BUILD_EXECUTABLE)
```

`Application.mk`：

```
APP_ABI := armeabi-v7a arm64-v8a

APP_PIE := true

APP_CPPFLAGS := -frtti -std=c++11 -mllvm -fla -mllvm -bcf -mllvm -sub
```

`try.cpp`：

```
#include <stdio.h>
int main()
{
  int d;
  scanf("%d",&d);
  if(d==1)printf("d = 1");
  else if(d==2)printf("d = 2");
  else printf("not 1 or 2");
  return 0;
}
```

在 `jni` 的上级路径，输入 `ndk-build`：

![image](https://attach.52pojie.cn/forum/202401/15/175055ard3wwlzrroolqqo.png)

成功了！

混淆后的文件在 `libs` 内：

![image](https://attach.52pojie.cn/forum/202401/15/175108u20f62m555int0fy.png)

在 `ida` 中查看混淆效果：

![image](https://attach.52pojie.cn/forum/202401/15/175111bougwaouoaakcaoa.png)

### clang 混淆

这里对源文件进行平坦化混淆处理。来到 `build/bin` 目录下，将待混淆的源文件放在该目录下。命令格式为：

```
./clang -mllvm -fla 源文件 -o 目标文件
```
