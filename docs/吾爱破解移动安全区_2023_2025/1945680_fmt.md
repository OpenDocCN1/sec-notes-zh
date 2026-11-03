
# [Android 原创] Frida 安装

原帖: <https://www.52pojie.cn/forum.php?mod=viewthread&tid=1945680>

抓取时间: 2026-11-01T16:55:01.725Z

逆向过程中，借助 `Frida` 可以更快的实现软件的逆向破解，`Frida` 不仅可以实现 Android 应用的逆向，对于 IOS 应用、Windows 应用同样可以实现不错的效果。本文可能由于 `frida` 代码的不断更新，可能出现某些错误或者过时，大家尽量以官方文档为主：

- `frida-server` 安装（Android）: <https://frida.re/docs/android/>
- `frida-tools` 安装：<https://frida.re/docs/installation/>
- `frida` 官方网站: <https://frida.re/>
- `frida` 官方文档：<https://frida.re/docs/>

本次安装涉及 `Frida` 的两个组件，一个是安装到手机或模拟器中的: `frida-server`，另外就是装在电脑上用于逆向分析的：`frida-tools`。

## `frida-server` 安装

`frida-server` 本质上是一个二进制文件，将其拷贝到需要进行 hook 的系统中运行即可，下面以 Android 系统为例：

首先需要从 github 的仓库中下载编译好的 `frida-server` 文件。仓库地址：<https://github.com/frida/frida>

从 Release 中下载对应版本的文件即可。`frida-server` 的命名规则为：`frida-server-{版本号}-{支持的系统}-{CPU架构}.{压缩文件扩展名}`

认准自己手机或模拟器的架构下载即可。例如我是 `arm64` 的 Android 手机，那么我就下载 `frida-server-16.4.5-android-arm64.xz` 即可。

> **注意：**
> 
> `frida` 的版本号需要注意一下，例如低版本的 Android（如 Android 6），可能出现 `frida-server` 运行时宿主系统出现黑屏、死机的问题。如果遇到此问题，手机未自动重启，手动强制重启后系统即可恢复正常。下面附上一张来自网络的 `frida` 版本与 Android 版本的对照表（官方并未说明是否有强制性要求 Android 版本与 `frida` 版本的对应关系，数据来源：<https://blog.51cto.com/u_16213445/9231363>）：

| Frida 版本 | Android 版本 |
| --- | --- |
| `Frida 12.6.13` | Android 4.1 - 4.3 |
| `Frida 12.7.0` | Android 4.4 |
| `Frida 12.8.1` | Android 5.0 - 5.1 |
| `Frida 12.9.7` | Android 6.0 - 6.0.1 |
| `Frida 12.9.8` | Android 7.0 - 7.1 |
| `Frida 12.10.4` | Android 8.0 - 8.1 |
| `Frida 12.11.7` | Android 9 |
| `Frida 12.12.0` | Android 10 |
| `Frida 12.12.2` | Android 11 |

如果需要下载历史版本，请从 github 的 `frida` 仓库 Release 中寻找历史版本。

下载到合适的 `frida-server` 后，使用 `adb push` 命令将二进制文件推送到目标设备中：

```
adb push frida-server /data/local/tmp/
```

如果推送不成功，也可以先推送到 `/sdcard/` 目录后，再使用 `adb shell` 将 `frida-server` 复制到 `/data/local/tmp/` 目录下：

```
adb push frida-server /sdcard/
adb shell
# 进入shell后，使用 su 命令切换到 root 用户
su
# 如果手机/模拟器提示需要授权，请在手机上点击授权即可，请留意手机
mv /sdcard/frida-server /data/local/tmp/
```

推送完成后，使用 `adb shell` 命令进入 shell，并赋予 `frida-server` 可执行权限：

```
adb shell
# 进入shell后，使用 su 命令切换到 root 用户
su
# 如果手机/模拟器提示需要授权，请在手机上点击授权即可，请留意手机
cd /data/local/tmp/
# 赋予 frida-server 可执行权限
chmod 777 frida-server
```

上述工作都做完后，可以启动 `frida-server` 了：

```
adb shell
# 进入shell后，使用 su 命令切换到 root 用户
su
# 如果手机/模拟器提示需要授权，请在手机上点击授权即可，请留意手机
cd /data/local/tmp/
# 赋予 frida-server 可执行权限
./frida-server
```

`frida-server` 启动后不会有任何屏幕的提示信息。请注意这个命令行窗口不要关闭，如果命令窗口关闭后，`frida-server` 也会关闭。

## `frida-tools` 安装

需要注意的是，上文提到的 `frida-server` 需与 `frida` 版本保持一致，安装 `frida-tools` 时又会自动安装 `frida`。那么如何得知 `frida-tools` 版本呢？

首先打开 `frida` 仓库：<https://github.com/frida/frida>，在 Release 中寻找 `frida` 版本并查找到 `frida` 的发布日期。

获取到 `frida` 发布日期后，打开 `frida-tools` 仓库：<https://github.com/frida/frida-tools>，寻找上述 `frida` 发布日期的相同或之后的一个 `frida-tools` 版本，请记住这个版本号！

以 Windows 系统为例，电脑中需要存在 Python 环境：

> **注意：**
> 
> 官方对于 Python 版本未进行说明，理论上 python3 都是可以的，但经过测试，`frida-tools` 版本与 Python 版本有关联，如果版本不对应，可能出现安装失败或安装后运行时报错等问题！

如何寻找 `frida-tools` 版本支持的 Python 版本？

打开 pip 包搜索网站：<https://pypi.org/project/frida-tools/>，点击左侧的 Release history，在右侧寻找对应的 `frida-tools` 版本并点击进入。下滑并注意左侧的 Classifiers 下的 Programming Language。下面写的最新的 Python 版本就是一定可以支持此版本 `frida-tools` 的 Python 版本。

如果有需求进行 Python 版本切换的读者，可以使用 `conda` 进行 Python 环境的管理。此处就不再赘述 `conda` 的相关安装和使用了。

使用以下命令安装 `frida-tools`：

```
pip3 install frida-tools=={frida-tools版本号}
```

> **注意：**
> 
> 安装 `frida-tools` 时会自动安装 `frida`，所以不要像网上某些教程教的一样分别安装，不然有可能会出现安装失败或安装后运行时报错等问题！

如果安装时出现网络连接问题，或者一些与网络相关的错误，请多试几遍，也可使用以下方法：

- 使用 `-i` 参数指定软件源：`pip3 install frida-tools=={frida-tools版本号} -i https://pypi.tuna.tsinghua.edu.cn/simple`
- 使用 `--proxy` 参数: `pip3 install frida-tools=={frida-tools版本号} --proxy='socks5://127.0.0.1:7890'`

安装完成后，使用命令 `frida-ps` 命令验证是否安装成功：

```
C:\Users\LemonGo97> frida-ps

  PID  Name
-----  ----------------------------
  832  chrome.exe
27076  chrome.exe
30132  chrome.exe
22788  chrome.exe
21040  cmd.exe
30480  cmd.exe
 1872  cmd.exe
13060  conhost.exe
 1880  dllhost.exe
 8544  explorer.exe
13700  frida-ps.exe
15032  nvcontainer.exe
26740  rundll32.exe
27464  rundll32.exe
26948  rundll32.exe
 2076  rundll32.exe
 9824  sihost.exe
 9860  svchost.exe
 9868  svchost.exe
 9928  svchost.exe
11208  svchost.exe
20020  svchost.exe
10020  taskhostw.exe
10084  taskhostw.exe
...
```

执行命令后出现的就是当前机器的进程列表，证明 `frida-tools` 单独运行是没问题的。那么如何测试是否能正确的与运行在手机或模拟器中的 `frida-server` 通信呢？

将手机与电脑连接后，按照上文说的启动 `frida-server`，在命令行中使用 `frida-ps -U` 命令查看手机进程：

```
C:\Users\LemonGo97> frida-ps -U
  PID  Name
-----  --------------------------------------------
  683  ATFWD-daemon
24558  adbd
  672  adsprpcd
  686  alipayservice
11860  android.process.acore
22740  android.process.contacts
11653  android.process.media
  690  audiod
  664  bspCriticalLog
  662  cnd
  680  cnss-daemon
12137  com.amap.android.location
12262  com.android.incallui
24490  com.android.keyguard
22711  com.android.mms
12125  com.android.phone
14538  com.android.providers.downloads
```

如果执行结果如上所示，那么基本上你的 `frida` 安装已经成功了，如果出现错误，请仔细检查以上的所有步骤是否正确。
