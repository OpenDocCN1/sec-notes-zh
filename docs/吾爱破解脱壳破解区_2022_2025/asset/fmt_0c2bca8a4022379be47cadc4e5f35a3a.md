
# [分享] 新手入坑IDA的一些小经验，贻笑大方了

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2049450
- 抓取时间: 2026-11-01T22:06:17

首先在吾爱网站上下载绿色版IDA（虽然版本比较老，但相差也不到一个版本，而且这个更安全）

从网盘里下下来的`portable zip`解压后即可使用，无需额外安装，解压完之后第一件事就是运行`绿化工具.exe`

下载时防火墙会狂叫中毒，不用管他

详细教程参照吾爱的教程 https://www.52pojie.cn/forum.php?mod=viewthread&tid=1874203

IDA也是很好操作的，

## 1. 判断要逆向的文件操作位数，是64位还是32位

这个可以通过记事本来实现，网上教程原文如下 https://blog.csdn.net/qq_34059233/article/details/105136933

64位就用`IDA64`打开，32位就用`IDA32`打开

这里打开的是我下小说用的软件，它是32位

## 2. IDA

接下来IDA会弹出一个这样的窗口让你设置参数

不要动任何东西，直接点`OK`，因为这些都是我们亲爱的IDA自己识别出来的，除非你有比它更高明的想法，不然就别动

然后就会弹出以下界面，这就是IDA给你的解析结果

点击相应的函数名，右边即会以汇编语言显示其内容及结构；使用`f5`可以反汇编为易读的高级语言

反汇编时可能会显示如下信息

```
The decompiler assumes that the segment '.rdata' is read-only because of its NAME.
All data references to the segment will be replaced by constant values.
This may lead to drastic changes in the decompiler output.
If the segment is not read-only, please change the segment NAME.
In general, the decompiler checks the segment permissions, class, and name
to determine if it is read-only.
```

原因是当反汇编器识别到`.rdata`时会自动将其归类为只读数据，
但有时`rdata`只是一个名字或名字的一部分，这时就可能将变量错误识别为常量，返回变量访问的硬编码地址

对于这种提示，可以先忽略其警告，然后在反汇编结果中观察是否有异常硬编码，再调回汇编语言界面自行判断

界面如`import`等字面上看就知道功能了，也挺容易就能读懂的。
