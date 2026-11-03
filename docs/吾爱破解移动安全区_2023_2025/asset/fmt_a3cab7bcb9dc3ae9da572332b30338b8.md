
# [Android 原创] Android 渗透测试学习笔记（一）：基础指北

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1918003
- 抓取时间: 2026-11-02T01:15:01

搬一下本人的学习笔记，新手勿喷{:1_919:}

图太多了，一个一个重传应该不行，先传md过来看看会不会被夹图

## 0 adb常见命令

一般模拟器（非真机模拟器）的安装目录下会有一个`adb.exe`。如果安装了Android Studio的话也会附带一个`adb`。如果两种情况都没有则需要去自己下载一个。调试远端的设备（本机上的模拟器一般不用，如雷电模拟器）需要先使用`adb connect`进行连接再`shell`。

- `adb shell` 进入`shell`，也就是获取一个手机命令行
- `adb push 电脑地址 手机地址` 将电脑文件导入到手机指定位置
- `adb install 电脑地址` 安装电脑地址上的apk
- `adb connect ip port` 链接指定位置的手机（适用于远程调试）
- `adb devices` 查看当前设备列表
- `adb forward agree:port agree:port` 转发端口
- `adb shell pm uninstall -k --user 0 包名` 卸载apk

## 1 代{过}{滤}理

### 1.1 挂代{过}{滤}理步骤

（这个方法真机模拟器无法使用，真机挂代{过}{滤}理方式请看下文第二节）

使用`burpsuite`、`mitmproxy`、`lamda`。

lamda github页面：[//github.com/rev1si0n/lamda/releases" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/rev1si0n/lamda/releases:rel="nofollow noopener" href="https)

首先需要准备一个root权限的手机。下载lamda的安装脚本`adb`中使用`sh`运行安装（详细步骤请自行查看lamda的ReadMe）。主机中使用`pip`安装lamda的客户端。

手机如果是模拟器，配置到与主机同一个网段则需要使用桥接模式。使用桥接模式后互相能`ping`通就可以了。

启动lamda的命令：

```
sh /data/local/tmp/x86/bin/launch.sh
```

先启动手机上的lamda。使用lamda是为了挂代{过}{滤}理+安装证书。

mitmproxy使用`pip`命令即可安装（它是一个`python module`）：

```
pip install mitmproxy
```

安装mitmproxy的证书（运行`mitmdump`挂起服务器，浏览器走mitm的代{过}{滤}理访问`mitm.it`即可安装证书）并挂上mitmproxy的代{过}{滤}理如下：
