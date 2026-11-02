
# [MacOS逆向] 破解 QCAD for Mac

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1849172
- 抓取时间：2026-11-01T22:44:07

## 破解 QCAD for Mac

因为今天要修改一个图，不能有其他公司的名字，光改PDF没用，所以只能从dxf文件下手。搜来搜去又不想装Autodesk CAD，太麻烦了，所以随便找了个QCAD破一下，好像这软件还挺好用。

### 1. 分析

没激活前是这样的

![](https://attach.52pojie.cn/forum/202310/26/232311pce244lfrzzcrrcy.jpg)

16983236052493.jpg (514.18 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTg5NnwyZTFlMTY3NnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTg5NnwyZTFlMTY3NnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

充斥着Trial等字样，右下角还有一个审判按钮。

其次，使用大约30-40分钟左右，会有退出弹窗

![](https://attach.52pojie.cn/forum/202310/26/232305dwgxgr7xgfx8044m.jpg)

16983233226012.jpg (518.49 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY1MTg5NXw0NTM1MThhNXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTg5NXw0NTM1MThhNXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

并显示我试用到期。

并且最重要的是，导出图纸到pdf有一很大的水印，如果没有水印我可能就不会破解他了。

我打算以“`This is a trial version of`”为切入点，暴力搜索所有文件试试。

![](https://attach.52pojie.cn/forum/202310/26/232313j8mt7tc9yfz4eucd.jpg)

16983242327040.jpg (66.7 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTg5N3w0OWY1YmQ5YnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTg5N3w0OWY1YmQ5YnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

就让审判来得更猛烈些吧！

### 2. 审判

常规搜索我们是搜索不到的。所以我们需要用一些特别的手段来暴力搜索所有二进制里的文件。

因为我们知道，英文字符串大部分就是原文在二进制中，所以我们暴力搜索一定能搜索到相关的内容。

```bash
find /Applications/QCAD.app/Contents -type f -exec sh -c 'strings "$1" | grep "This is a trial version of" && echo "$1"' sh {} \;
```

![](https://attach.52pojie.cn/forum/202310/26/232317nzrrxddx4pa0gnq4.jpg)

16983273366657.jpg (250.68 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY1MTg5OHxjNDkxYTZlZHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTg5OHxjNDkxYTZlZHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

差不多也就搜索到这了。

我们发现第一个文件和第二个文件都存在这个字符串，那么我们就着重搞这两个文件。

首先打开第一个：

![](https://attach.52pojie.cn/forum/202310/26/232321gcowaa6xxorwqoa7.jpg)

16983274665225.jpg (729.26 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTg5OXxmOTMwM2FlOHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTg5OXxmOTMwM2FlOHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

因为只有几百K，分析速度很快，我们一搜，还真有。

那这下不得不跳过去看下了：

![](https://attach.52pojie.cn/forum/202310/26/232326gvm4wgzvzz6p1dgz.jpg)

16983275016996.jpg (297.52 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTkwMHxjYjk1YTIyNHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwMHxjYjk1YTIyNHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

糟糕糟糕OMG，魔法怎么施灵辣！

却说这大魔导师落叶不愧是强者，事情发展出乎意料却也不急，冷笑着说道：“尔等不过区区Qt多语言，待我略施小技，必取你首级！”

那QCAD本就是小作坊出产，哪里见过这等阵仗？当下心中狂跳，嘴上却是硬道：“兀那乱臣贼子！休要在此胡言乱语妖言惑众，我等在此会你一二，看你还有什么手段？”

那大魔导师落叶出道以来哪里受到如此挫折？闻得QCAD猖狂挑衅，当下却是道心不稳，怒从心中起，恶言道：“区区蝼蚁也敢在本魔导师面前螳臂当车？死来！”

当下落叶长啸一声，法杖高举，大喝道：“天圆地方乾坤倒转，禁术·Hopper disassembler，启动！”

说时迟那时快，原本万里无云的蓝天诡异的传出一声巨响，那QCAD当即被一个邪异蓝色光罩团团围住，进出不得。QCAD惊怒交加，一时不察竟受制于人，怒不可遏：“贼子敢尔！”

那大魔导师落叶也不说话，匆忙的结印，操作着究极禁术HP疯狂攻击：

![](https://attach.52pojie.cn/forum/202310/26/232330yvdg85ezz48od344.jpg)

16983283509738.jpg (660.11 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY1MTkwMXxjOTFhMDg4ZXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwMXxjOTFhMDg4ZXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

此时令所有人震惊的一幕出现了，那究极禁术竟然对QCAD毫发无伤！

QCAD原本还略有慌乱，此时发现自己没有受到任何伤害，眼珠一转便哈哈大笑：“黄毛小儿，想害我？再去修炼几年！”

“可恶！”落叶面如金纸，噗的吐出一口黑血，原来这禁术无法施展太久，否则会反噬施术者。一时之间，竟然僵持不下。

远处，迦南世家伯爵迦南二世皱眉道：“此子成长速度太快，区区魔导师境界的魔力就已经能越级施展禁术，之前还得罪了他，如此看来只好斩草除根了！告诉QCAD大师，如果能当场斩杀魔子落叶，我们迦南世家将不遗余力的帮助他成为璃月大陆的风神代言人！”说到最后，竟然已经是咬牙切齿。

两名骑士对视一眼，不敢怠慢，当即领命去设法通知QCAD大师。

那QCAD见这所谓的大魔导师兼魔子对自己毫发无伤，更是轻视三分，又收到迦南二世伯爵送来的密信，当即冷笑一声：“黄口小儿，你的死期到了！”

当即吟唱法术，欲要突破牢笼。

此时落叶本就被反噬遭到重创，哪里敢让QCAD突破？拼了命维护牢笼，一时之间却也想不到别的办法。

此时，落叶突然想到这里：

![](https://attach.52pojie.cn/forum/202310/26/232333zfp7h4o6hqef4nvz.jpg)

16983291033863.jpg (87.72 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY1MTkwMnw5MjkxZTZmMXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwMnw5MjkxZTZmMXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

这可不就是苦苦寻觅么线索？查不到又如何？还不是漏出了破绽？

当即狂喜，迅速`xref`过去一看：

果然发现这`arg0+0x28`偏移处的地址值若为`0`，则跳转到`7a73`，也就是`trial`模式。

![](https://attach.52pojie.cn/forum/202310/26/232340md9k53xid3qd8esb.jpg)

16983291717642.jpg (343.34 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTkwM3wyMmY2MzIzZHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwM3wyMmY2MzIzZHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

那么就让他不跳转吧！

落叶默默想着，迅速回到IDA：

![](https://attach.52pojie.cn/forum/202310/26/232346l8vzjqzdtqdjdizw.jpg)

16983292935703.jpg (1.32 MB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTkwNHw2NjNkYmU4NnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwNHw2NjNkYmU4NnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

找到`checkLicense()`函数，进去发现若`v4 = QVariant::toInt((QVariant )&v8, 0LL);`的值大于9，则走一串神秘的js脚本，并传递了`this`的指针地址，盲猜返回上级函数会让`if ( ((_BYTE *)this + 40) )`判断成立，因为默认`v4`得到的值是`NULL`，也就是`0`，所以这将是我们的突破口！

```cpp
v4 = QVariant::toInt((QVariant *)&v8, 0LL);
QVariant::~QVariant((QVariant *)&v8);
QVariant::~QVariant((QVariant *)&v9);
*((_BYTE *)this + 40) = 0;
if ( v4 > 3 )
{
  if ( v4 > 9 )
    QTimer::singleShot(
      (QTimer *)"(_0x3c3852,_0x10a608){var _0x36519d=a0_0x3651();return a0_0x4121=function(_0x4121b1,_0x35735a){_0x4121"
                "b1=_0x4121b1-0xd3;var _0x3e8b74=_0x36519d[_0x4121b1];return _0x3e8b74;}...............省略",
      (int)this,
      (const QObject *)"1uninitSlot()",
      v5);
```

于是想办法将这`v4 = QVariant::toInt((QVariant *)&v8, 0LL);`改成`v4 = 10`：

![](https://attach.52pojie.cn/forum/202310/26/232349mj0imguzu0u1joos.jpg)

16983296622206.jpg (195.93 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTkwNXwwMDUzYTQ3OHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwNXwwMDUzYTQ3OHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

光标放在这一行，切换到HexView：

![](https://attach.52pojie.cn/forum/202310/26/232351jqfmav54wvj6w94t.jpg)

16983296862611.jpg (56.54 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY1MTkwNnwwYWI3MGIwM3wxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwNnwwYWI3MGIwM3wxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

自动定位到对应的十六进制地址，右键点击菜单Edit...修改为：

![](https://attach.52pojie.cn/forum/202310/26/232354kyzsv8jrjzy4a1z4.jpg)

16983297397675.jpg (99.93 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY1MTkwN3wxNjdiMzEzYnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwN3wxNjdiMzEzYnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传

最后Apply即可。

返回反汇编即可看到

![](https://attach.52pojie.cn/forum/202310/26/232357cpfj83cpcfkp9cpp.jpg)

16983297694135.jpg (132.36 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTkwOHxjNjVhNTIyOXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkwOHxjNjVhNTIyOXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes)

2023-10-26 23:23 上传
