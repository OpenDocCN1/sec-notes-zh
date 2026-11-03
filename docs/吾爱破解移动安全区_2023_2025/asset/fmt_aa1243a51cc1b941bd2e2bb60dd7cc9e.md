
[href="forum.php?mod=attachment&aid=MjYwNDgwMnwzMDE0N2Y1Y3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwMnwzMDE0N2Y1Y3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

## 插件安装

然后进行插件安装：`karen`->插件->`AppSync`->安装->确认->重启

![](https://attach.52pojie.cn/forum/202304/04/184121mfrf7fcjrocz7ac2.png)

2023040417.png (588.3 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDgwOHxhMzc5MThjZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwOHxhMzc5MThjZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

## frida-server

### 卸载历史版本

```
dpkg -l | grep frida 匹配已经安装的frida
dpkg -P re.frida.server卸载软件
```

下载最新`frida-server`进行安装

![](https://attach.52pojie.cn/forum/202304/04/184118ocmocvs4ivivcnim.png)

2023040418.png (52.31 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwNDgwN3xiZDU3NGNmNHwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwN3xiZDU3NGNmNHwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

`dpkg -i frida_16.0.10_iphoneos-arm.deb`

## 4. Frida/Objection 任意版本安装切换/动态分析

这里使用`r0env`,自带`pyenv`支持多版本`Frida`/`objection`切换，这里安装最新版本作为演示

### (1) 安装最新 frida

```
pyenv install 3.9.5
pyenv global 3.9.5
pip install frida==16.0.5
```

这里安装的时候不挂代{过}{滤}理会很慢，挂代{过}{滤}理后遇到了一个错误

![](https://attach.52pojie.cn/forum/202304/04/184116x3kxzzywhbhdjp9x.jpg)

2023040419.jpg (28.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwNDgwNnxhMzc4NTY4N3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwNnxhMzc4NTY4N3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

错误原因是没有`socks`相关库

`unset ALL_PROXY`

`pip install pysocks`

再挂代{过}{滤}理安装

```
export ALL_PROXY="socks://代{过}{滤}理IP:port"
pip install frida
pip install frida-tools
```

安装成功

![](https://attach.52pojie.cn/forum/202304/04/184114ne6nayyq7t5av45t.png)

2023040420.png (22.81 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwNDgwNXw0MDgyYThmZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwNXw0MDgyYThmZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

### (2) 安装 objection

```
pip install objection

objection -g 高考蜂背 explore
```

![](https://attach.52pojie.cn/forum/202304/04/184112fpqq12qkappz3q1a.png)

2023040421.png (118.35 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDgwNHxmZmUwOTg4YXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwNHxmZmUwOTg4YXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

## 5. iOS 砸壳/IPA 静态分析基本流程/IDA F5 函数定位

### (1) iOS 砸壳

iOS砸壳工具也有很多，这里我们推荐使用`frida-ios-dump`,下边是安装使用详细过程：

```
git clone https://github.com/AloneMonkey/frida-ios-dump
cd frida-ios-dump/
apt-get install usbmuxd
pip install -r requirements.txt --upgrade
iproxy 2222 22
./dump.py 高考蜂背
```

### (2) IPA 静态分析基本流程及 IDA F5 函数定位

因为本篇文章主题是环境搭建，因此这里仅演示简单流程

```
file 高考蜂背.ipa

unzip 高考蜂背.ipa

cd Payload/

cd Gkfb.app/

file * | grep -i mach

ida64 ./Gkfb
```

经过`frida-ios-dump`砸壳后我们拿到了`IPA`包对其进行解压，并检索内部`mach-o`可执行文件，随后用`IDA`分析文件。

![](https://attach.52pojie.cn/forum/202304/04/184110uq2qn9qzqn9vkk0w.png)

2023040422.png (99.87 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwNDgwM3w5MDEyNmQxMXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwM3w5MDEyNmQxMXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

![](https://attach.52pojie.cn/forum/202304/04/190058gp7h106ha1p1tv49.png)

20230404checkfile.png (31.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDgzOXwzMTEzOGQ5MXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgzOXwzMTEzOGQ5MXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 19:00 上传

分析过程检索发现`MD5`格式函数，使用`F5`即可看到其伪代码

![](https://attach.52pojie.cn/forum/202304/04/190114ffwv491f9bnl8q4q.png)

20230404idacheck.png (326.08 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYwNDg0MHwxZjI5NWQzZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDg0MHwxZjI5NWQzZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 19:01 上传

![](https://attach.52pojie.cn/forum/202304/04/190127r10a1f1ixx41pi28.png)

20230404code.png (198.13 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwNDg0MXwwYmZmNzJmN3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDg0MXwwYmZmNzJmN3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 19:01 上传

## 6. Linux 中 iPhone 手机信息获取: ID/名称/详情/签名/截屏/定位

在`Linux`中操作`iPhone`主要使用`libimobiledevice`库及依赖于它的一些开源工具，这里先下载安装该库

```
apt-get update
add-apt-repository ppa:pmcenery/ppa
apt-get install libimobiledevice-utils
```

下面是该库的一些简单使用:

### (1) 获取设备 ID

```
idevice_id 获取当前连接设备UUID
```

![](https://attach.52pojie.cn/forum/202304/04/184336um4y399e3j4lcmey.png)

20230404UUID.png (53.86 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDgxN3wwMGFjN2JiMXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgxN3wwMGFjN2JiMXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:43 上传

### (2) 获取设备名称

```
idevicename 查看当前连接设备名称
```

![](https://attach.52pojie.cn/forum/202304/04/184352ich55i4n7nx8h4zh.png)

20230404name.png (45.47 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDgyNHxkMWVlOGY2NXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgyNHxkMWVlOGY2NXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:43 上传

### (3) 获取屏幕截图

```
idevicescreenshot :从连接的设备获取屏幕截图
```

![](https://attach.52pojie.cn/forum/202304/04/184343ffcwiextcbqizb3f.png)

20230404screenshot.png (104.09 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYwNDgyMHwxOWYxZmJjN3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgyMHwxOWYxZmJjN3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:43 上传

### (4) 设置虚拟定位

```
idevicesetlocation [OPTIONS] -- 根据经纬度模拟定位
idevicesetlocation -- 35.10463 117.193626 山东枣庄
```

![](https://attach.52pojie.cn/forum/202304/04/184350tpl2env5pcs9v29s.png)

20230404position.png (347.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDgyM3xhNzFlY2I4YnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgyM3xhNzFlY2I4YnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:43 上传

### (5) 获取设备详情

```
ideviceinfo :查看手机设备详情列出全部相关信息
```

![](https://attach.52pojie.cn/forum/202304/04/190203ble70ph20t0rhxre.png)

20230404detail.png (104.81 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYwNDg0Mnw2NjgxZDA3MnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDg0Mnw2NjgxZDA3MnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 19:02 上传

## 7. 安装 APP/IPA: 安装/卸载/升级/备份/恢复 APP

`Linux`上对`iPhone`中软件操作这里是使用的依赖于`libimobiledevice`库的工具`ideviceinstaller`，在安装完`libimobiledevice`库后我们可以直接使用`apt`安装该工具

```
apt-get install ideviceinstaller
```

### (1) 安装软件

```
ideviceinstaller -i xxx.ipa 安装软件
```

![](https://attach.52pojie.cn/forum/202304/04/190214yez44rfhff43fmh6.png)

20230404install.png (193.31 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYwNDg0M3w5N2ExZGQxZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDg0M3w5N2ExZGQxZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 19:02 上传

### (2) 卸载软件
