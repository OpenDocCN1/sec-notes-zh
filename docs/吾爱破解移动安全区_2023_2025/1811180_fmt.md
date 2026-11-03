
# [Android 原创] 《MT》如何进行log注入

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1811180
- 抓取时间: 2026-11-01T21:41:32

[最近在学习安卓的log注入，原本MT自己是本身就带有这个功能的，但是是网络上和官网的相关信息比较少，这边我就决定自己去摸索属于自己的一套log方法（本人是新手，希望各位喷的时候手下留情）

## 先说思路

因为安卓有个包就是负责打印log日志的，想办法写入到app中去，然后如果我们有个软件可以显示打印log日志(说的正是DDMS)，那么或许我们可以将他们联系起来。

于是可以写一个java类是有关log打印的，然后转换成smali类型再传入到目标软件的smali里面，最后再在目标函数中调用我们写好的smali就行了.(或许你们也好奇为什么不直接在目标函数那里直接使用安卓提供log日志的方法，这边我没试过)

这边我直接叫gpt帮忙写了个然后转换成smali

![Snipaste_2023-07-21_12-01-56.png](https://attach.52pojie.cn/forum/202307/21/164549wb1f0zii4qzj0juf.png)

在我们得到这些之后我们需要进行梳理，这个类叫`Mylog`，只有一个`print()`方法，里面传入个字符串然后进行`log.i("Mylog",要传入的字符串)`实现打印

因为MT的特殊，当我们传入smali的时候 .class部分的直接省去路径，也就是直接`.class Mylog`

如下图

![Snipaste_2023-07-21_12-11-34.png](https://attach.52pojie.cn/forum/202307/21/164616xve5n37qsd6ne6us.png)

## 安卓方面的操作

首先是先已经进入了app安装包内部然后找到dex进行点击

选择 Dex编辑器++

在进入后，会进入dex内部，里面会转换成smali

![Screenshot_2023-07-21-16-49-57-48.jpg](https://attach.52pojie.cn/forum/202307/21/171745ee65wjz716zljgv1.jpg)

这时候点击右上角创建工程名称，并点击确定

![Screenshot_2023-07-21-16-50-37-00.jpg](https://attach.52pojie.cn/forum/202307/21/171747r38gyaptky0wtptq.jpg)

这时候再退出到进入dex的位置，并且左滑可以看到

![Snipaste_2023-07-21_17-29-34.png](https://attach.52pojie.cn/forum/202307/21/173132xvq5ccrfq545rtu2.png)

然后找到自己的工程并且右滑可以看到灰色的定位标志

![Snipaste_2023-07-21_17-29-49.png](https://attach.52pojie.cn/forum/202307/21/173136zllu9u6ypua2mccq.png)

这时候左边的文件列表就是自己的工程，然后我们可以认为 左边文件列表要定位到传送smali的入口，右边文件列表是要定位到要传送smali的文件

![Screenshot_2023-07-21-16-51-16-95.jpg](https://attach.52pojie.cn/forum/202307/21/171753phxep83r020d4cr8.jpg)

左边的文件列表依次点击 `dex-smali-classes`(注意这里有多少个dex就有多少个classes)的里面就是传送smali的入口，这时候也把右边要传送的smali给确定好就是如下图

![Screenshot_2023-07-21-17-01-24-25.jpg](https://attach.52pojie.cn/forum/202307/21/171755hy7z7oqzuiou704o.jpg)

然后右边的直接点击移动即可。

随后我们直接右滑点击我们自己的工程，然后可以看到有个`Mylog`

![Screenshot_2023-07-21-17-02-44-93.jpg](https://attach.52pojie.cn/forum/202307/21/171757k6gaw1d125pgpq6b.jpg)

我自己就随机找个位置进行注入。如下图

![Screenshot_2023-07-21-17-04-38-33.jpg](https://attach.52pojie.cn/forum/202307/21/171800c20yvnw8ybd7vbyy.jpg)

在成功写入后直接退出保存然后右上角点击锤子，这时候就会进行编译然后完成后进行定位。

这时候左边的文件列表是已经修改好的dex，右边的文件列表是要替换的dex

![Screenshot_2023-07-21-17-06-14-78.jpg](https://attach.52pojie.cn/forum/202307/21/171738dpq4ug40qccqp2cf.jpg)

然后把要注入的app进行解压然后打开文件夹，这时候进行替换即可，然后直接右上角点击全选，然后点击压缩，以apk为后缀，之后这时候还要进行apk签名，最后才能安装app

![Screenshot_2023-07-21-17-07-59-31.jpg](https://attach.52pojie.cn/forum/202307/21/171740llllornrlnkndlx0.jpg)

得出的结果如下图

![Snipaste_2023-07-20_22-18-45.png](https://attach.52pojie.cn/forum/202307/21/171525h68g76plfi6trifv.png)
