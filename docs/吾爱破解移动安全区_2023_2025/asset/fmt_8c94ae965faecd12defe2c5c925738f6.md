
# [Android 原创] luckfollowme arm学习篇 4 - 运行arm

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1778693
- 抓取时间: 2026-11-01T21:44:58

## luckfollowme arm学习篇 4 - 运行arm

上一章讲解了如何编译 `x86`、`x86_64`、`armeabi-v7a`、`arm64-v8a` 的架构文件。

这章我们就要运行它，并且尝试使用 ida pro 进行 debug。

### 准备 emulator

我们需要准备 emulator（模拟器）。

注意模拟器我们要用 `arm64` 的，这样才能运行我们的 `arm` 文件。

`arm64` 是在 Android 5 出来的，也就是 Android API 21 的时候开始，所以我们最低选择 Android API 21 的。

下面是图文教程：

1. create device 创建驱动

   ![](https://attach.52pojie.cn/forum/202304/26/073832vsnzrvpitat5mm4v.png)

   [`01.png` (25.85 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2M3xiMzRhZTAyZHwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

   2023-4-26 07:38 上传

2. choose a device definition 选择您的手机驱动尺寸

   ![](https://attach.52pojie.cn/forum/202304/26/073834uog8ff3olj38oe5l.png)

   [`02.png` (39.43 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2NHw3NWExNzg0YnwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

   2023-4-26 07:38 上传

3. 选择 `arm64` 镜像，注意最好选 API level 21 的。因为有的 CPU 不支持高版本的 `arm`。

   ![](https://attach.52pojie.cn/forum/202304/26/073836anonnz7bqa1jq2o2.png)

   [`03.png` (67.34 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2NXxhODk1MTM3YXwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

   2023-4-26 07:38 上传

#### 命令行运行 emulator

选择好后，您可以关闭 visual studio 了。

再次之前，打开 SDK 的目录，找到 `Android\Sdk\emulator\emulator.exe` 文件。

并参考下面的资料：

[developer.android.google.cn/studio/run/emulator-commandline?hl=zh-cn](https://developer.android.google.cn/studio/run/emulator-commandline?hl=zh-cn)

然后再在我们的项目的 `scripts` 目录中编写 `emulator-start.bat` 脚本。

在此之前，先看我们的设备名字：

```
emulator -list-avds
Pixel_4_API_21
```

然后您创建的脚本中添加如下内容：

```
C:\Users\hp\AppData\Local\Android\Sdk\emulator\emulator -avd Pixel_4_API_21 -netdelay none -netspeed full
```

然后通过 vscode 中终端启动它，启动你会发现它一直卡在如图所示的界面。

![](https://attach.52pojie.cn/forum/202304/26/073838varv7o3fhz7aff8f.png)

[`04.png` (84.21 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2Nnw5YzI1YmFiMXwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

2023-4-26 07:38 上传

这种情况是正常的，因为 `x86` 架构模拟 `arm` 是很慢。

但是我们不用等到它启动，因为实际上它内置的系统已经跑起来了。

如果你正确配置了 Android Studio platform-tools 到 `path` 环境下，您输入 `adb advices` 应该可以看到这个设备：

```
adb devices
List of devices attached
emulator-5554   device
```

您输入 `adb shell "ls -l"` 应该就能看到里面的目录情况了：

```
PS C:\Users\hp> adb shell "ls -l"
dr-xr-xr-x root     root              1970-01-01 00:00 acct
drwxrwx--- system   cache             2023-04-22 17:32 cache
lrwxrwxrwx root     root              1970-01-01 00:00 charger -> /sbin/healthd
dr-x------ root     root              1970-01-01 00:00 config
lrwxrwxrwx root     root              1970-01-01 00:00 d -> /sys/kernel/debug
drwxrwx--x system   system            1970-01-01 00:01 data
-rw-r--r-- root     root          281 1970-01-01 00:00 default.prop
drwxr-xr-x root     root              1970-01-01 00:00 dev
lrwxrwxrwx root     root              1970-01-01 00:00 etc -> /system/etc
-rw-r--r-- root     root        11276 1970-01-01 00:00 file_contexts
-rw-r----- root     root          922 1970-01-01 00:00 fstab.goldfish
-rw-r----- root     root          850 1970-01-01 00:00 fstab.ranchu
.....
```

### 运行我们的项目

### 修改项目

再次之前我们改进下我们的项目。之前我们用的 `hello.cpp` 使用 c++ 标准库导致生成的内容过大。

这回我们创建个 `hello2.cpp` 只弄些简单的东西。

结构大致如下：

```
practice
    -source
        hello.cpp
        hello2.cpp
    CMakeLists.txt
```

我们将 `hello2.cpp` 代码改成如下：

```
#include "stdio.h"

int main()

{

    printf("hello world!");

    return 0;

}
```

`CMakeLists.txt` 中我们配置个 `hello2.cpp` 的构建可执行程序：

```
# 使用最小版本号
cmake_minimum_required(VERSION 3.5)
# 我们的项目名称
project(InlineHookPratice)
# 启动支持 asm 汇编
enable_language(ASM)

# 设置 c++ 和 c 的标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD 11)

# 查找的include 的位置
include_directories(
    ./source/    

)

# 设置我们要编译的资源 放入 SOURCE_CPP 变量里面
set(hello_source 
    ./source/hello.cpp
)

# hello2 的代码来源
set(hello2_source 
    ./source/hello2.cpp
)

# 将我们要编译的资源 加入可执行文件 InlineHookPratice 里面 到时候编译出来就叫 InlineHookPratice.exe 或者 InlineHookPratice... 根据环境变化
add_executable(hello ${hello_source})

# 将hello2 生成 可执行程序 
add_executable(hello2 ${hello2_source})
```

运行上一章的 `ndk_bulder.py` 脚本的时候注意下：

```
if __name__ == "__main__":

    ndk_path = r"C:\Users\hp\AppData\Local\Android\Sdk\ndk\25.1.8937393".replace('\\',"/")

    cmake_path = r"F:\c++\msys2\mingw64\bin\cmake".replace('\\',"/")

    print("ndk_path:", ndk_path)

    print("cmake_path:", cmake_path)

    for abi in abis:

        # target_name 改成 hello2

        builder = AndroidPlatformBuilder(android_nkd_dir=ndk_path, cmake_dir=cmake_path, arch=abi,target_name="hello2")

        builder.build()
```

### 运行项目

通过 vscode 打开终端。

进入到我们构建的目录：然后将项目通过 `adb push` 到 `/data/local/tmp` 目录下：

```
cd ..\build\arm64-v8a\  #进入build arm64
adb push .\hello2 /data/local/tmp/ # push hello2 到 模拟器上
adb shell "chmod 755 /data/local/tmp/hello2" # 修改执行权限
adb shell "/data/local/tmp/hello2" # 运行
hello world!
```

为了方便，我们将它写成 `adb_execute_arm64.bat` 脚本放在 `scripts` 目录下。

然后我们可以直接在 vscode 终端执行下面命令就可以完成运行了：

```
.\adb_execute_arm64.bat
```

### Ida Pro 进行 Debug

我们最终是要学习 `arm` 汇编的，所以使用 ida pro 肯定是少不了的。

后续我尽量讲清楚点。

由于我 ida pro 也不熟悉。包括 `arm64`，因为我没学多久。

我也不太会找教程，因为我找教程只懂搜 B 站。

但是我所见的教程都是着着书读，半天不讲正点，也不实践。迷迷糊糊看上一篇下一篇就忘了，对于我这种喜欢速成实践才能记住的人属实消化不了。

稍微看着可以的就是要收费的，可俺是个穷光蛋呀。

大多学习也是通过买的 gpt 账号，一通乱问。感觉它是我的 AI 导师也不为过。

在此还是期待正己大佬的视频，从他这里入坑看别的没味道。

#### 准备 Ida pro

如果您是第一次使用 ida pro，那么就看本次教程。

在吾爱破解论坛上：[www.52pojie.cn/forum.php?mod=viewthread&tid=1584115&highlight=ida%2Bpro](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1584115&highlight=ida%2Bpro) 下载 ida pro。

这个包含了很多常用的插件，省去了麻烦。

下载完双击 `IDA_InitTool.exe` 完成环境初始化。

#### 准备 android_server64

在 `ida/dbgsrv` 目录下有着 ida pro 用于 debug 的服务，我们将它放置 `/data/local/tmp` 目录下，然后运行后转发端口到电脑上：

```
adb push .\android_server64 /data/local/tmp/    # push 到模拟器
adb shell "chmod 777 /data/local/tmp/android_server64" # 设置执行权限
adb shell '/data/local/tmp/android_server64 & ' # 后台启动
adb shell 'netstat' # 查看端口使用情况
adb forward tcp:23946 tcp:23946 # 将 模拟器端口映射到电脑 
```

#### 调试我们的程序

为了方便我们调试，我们得修改下代码，弄成死循环：

```
#include "stdio.h"

int main()

{

    while (true)

    {

        //阻塞等待输入

        getchar();

        printf("hello world!");

    }

    return 0;

}
```

然后执行脚本重新编译并运行。
