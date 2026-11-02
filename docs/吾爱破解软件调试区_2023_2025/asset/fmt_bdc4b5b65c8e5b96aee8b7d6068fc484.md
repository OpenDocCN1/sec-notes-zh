
# [游戏安全] 逆向UE4最简单的一集

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1838396
- 抓取时间: 2026-11-01T21:55:43

## 逆向UE4最全的一集

潜水时间过长，水一篇文章

在视频网站上看到许多课程，其中流行使用`CE`（Cheat Engine）来寻找数据。然而，这种介于模糊搜索、扫描和过滤的方式，不仅效率低下，对于逆向工程帮助甚微。有时候甚至换个游戏都难以取得有效成果。这时候就需要学习更高级的技巧。

随着时间的推移，对于`UE4`的逆向已经形成了一套成熟的方法和技术

以下介绍几个我认为很强的项目：

- [UnrealDumper-4.25](https://github.com/guttir14/UnrealDumper-4.25)，适用于Windows，早期版本的dump工具
- [UEDumper](https://github.com/Spuckwaffel/UEDumper)，适用于Windows，强大的工具
- [UE4Dumper](https://github.com/kp7742/UE4Dumper)，适用于Android的简单dump数据工具
- [AndUE4Dumper](https://github.com/MJx0/AndUE4Dumper)，适用于Android的可扩展dump工具，可导入`Android Studio`食用
- [iOS_UE4Dumper](https://github.com/MJx0/iOS_UE4Dumper)，适用于iOS的可扩展dump工具，可导入`Xcode`食用
- [UE4SDKGenerator](https://github.com/s4m33r89/UE4SDKGenerator)，适用于Android，强大的工具

通过上述项目，我们可以看到，在逆向`UE4`游戏时，对于引擎级别的逆向毫无疑问是最为主流和有效的。这些项目提供了强大的功能，能够避免低效的搜索方法，同时能生成可导入到相应平台开发环境中使用的SDK，进而通过调用游戏自身提供的函数和方法来实现相应的功能。

掌握上述项目的使用方法，可轻松应对`UE`系列的游戏，这里分享我在实际操作中遇到的一些困难。

### 游戏说明

游戏名: `StateOfDecay2` (4.13，无调试符号)

游戏名: `Absolver` (4.18，有调试符号)

我会通过两者的不同，来介绍当其中一种方法失效后应该怎么做

### 获取基础信息

首先需要获取游戏的一些基础信息，比如游戏使用的引擎版本、`GUObjectArray`、`GName`等，下面是针对4.23以下版本的演示，思路是通用的

#### 引擎版本

- `IDA`中搜索字符串：`"+UE"`

![](https://attach.52pojie.cn/forum/202310/08/184312vmnk4cjk2g9qada4.png)

2023-09-25-23-11-19-image.png (13.21 KB, 下载次数: 6)

[href="forum.php?mod=attachment&aid=MjY0NzY3OHwyNmUzYTQ4ZHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY3OHwyNmUzYTQ4ZHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:43 上传

- 在对应目录检查
- Windows: 游戏目录下运行的可执行文件，右键->属性->详细信息->产品版本

![](https://attach.52pojie.cn/forum/202310/08/184355b7pbqcbgklq4abzg.png)

2023-09-25-23-21-55-image.png (26.53 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY0NzY3OXwzNWExZjliZHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY3OXwzNWExZjliZHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:43 上传

- Android: 游戏apk中的`AndroidManifest.xml`

- iOS: 没试过

#### `GUObjectArray`

`IDA`中搜索字符串：`"Max UObject count is invalid. It must be a number that is greater than 0."`

有调试符号：

![](https://attach.52pojie.cn/forum/202310/08/184434xkg6p7qk42t5k4zx.png)

2023-09-25-23-29-26-image.png (301.06 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjY0NzY4MHw1NDZlNzc5ZnwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4MHw1NDZlNzc5ZnwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:44 上传

![](https://attach.52pojie.cn/forum/202310/08/184453edl2pa13anm3u15p.png)

2023-09-25-23-30-53-image.png (30.77 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY0NzY4MXwxYmQ1YzAxNHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4MXwxYmQ1YzAxNHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:44 上传

无调试符号：

![](https://attach.52pojie.cn/forum/202310/08/184510axgfcxp2zqq9rqr2.png)

2023-09-25-23-48-14-image.png (84.17 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0NzY4Mnw1MTRhMTdiYXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4Mnw1MTRhMTdiYXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:45 上传

![](https://attach.52pojie.cn/forum/202310/08/184535p0zae5bx1u1ry6no.png)

2023-09-25-23-49-53-image.png (233.64 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0NzY4M3xkYTlmNWFhY3wxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4M3xkYTlmNWFhY3wxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:45 上传

如果字符串搜索不到，可在源代码中寻找其他的字符串

#### `GName`

`CE`搜索字符串：`"ByteProperty"`，在`None`前下断点

![](https://attach.52pojie.cn/forum/202310/08/184621qmqwd9wkp0krpo7s.png)

2023-09-26-00-48-33-image.png (123.91 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY0NzY4NXxjYjE3NzRlZXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4NXxjYjE3NzRlZXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:46 上传

![](https://attach.52pojie.cn/forum/202310/08/184619lgq5pqz10pc18hgy.png)

2023-09-26-00-46-25-image.png (116.6 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0NzY4NHxlYTAwYjBmOHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4NHxlYTAwYjBmOHwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:46 上传

![](https://attach.52pojie.cn/forum/202310/08/184623s05izs5yyl7ps57l.png)

2023-09-26-00-49-51-image.png (69.72 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY0NzY4NnwwOWE0NjFiYXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4NnwwOWE0NjFiYXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:46 上传

```
//对应路径: `UnrealEngine-4.xx.x-release\Engine\Source\Runtime\Core\Private\UObject\UnrealNames.cpp`

FString FNameEntry::GetPlainNameString() const

{

if( IsWide() )

{

    return FString(WideName);

}

else

{

    return FString(AnsiName);

}

}
```

`IDA`对应伪代码：

有调试符号：

![](https://attach.52pojie.cn/forum/202310/08/184718ux6eghp154xjeopo.png)

2023-09-26-00-55-33-image.png (235.52 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0NzY4N3wzZmE1OGY5ZXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4N3wzZmE1OGY5ZXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:47 上传

无调试符号：

![](https://attach.52pojie.cn/forum/202310/08/184743naa6789qtyw92n9b.png)

2023-09-26-00-56-25-image.png (206.14 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY0NzY4OHw5OTg2MmVhZnwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4OHw5OTg2MmVhZnwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:47 上传

找到此函数后按`X`，寻找上层引用(默认是第一个)

有调试符号：

![](https://attach.52pojie.cn/forum/202310/08/184910c9hsm0m950g9jrvo.png)

2023-09-26-01-00-23-image.png (74.28 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0NzY4OXxhMDg3MDMyMXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY4OXxhMDg3MDMyMXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:49 上传

![](https://attach.52pojie.cn/forum/202310/08/184912cc886jpkjukqauqq.png)

2023-09-26-01-01-26-image.png (63.16 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY0NzY5MHxkMzlmMjNiNXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzY5MHxkMzlmMjNiNXwxNzkzNTcwMTQxfDE2ODA4NnwxODM4Mzk2&nothumb=yes)

2023-10-8 18:49 上传

无调试符号：

![](https://attach.52pojie.cn/forum/202310/08/184941fzoldfokllol9lzj.png)

2023-09-26-01-00-54-image.png (42.86 KB, 下载次数: 2)
