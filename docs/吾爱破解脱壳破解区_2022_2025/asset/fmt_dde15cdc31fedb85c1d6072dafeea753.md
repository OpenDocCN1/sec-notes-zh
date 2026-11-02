
# [.NET逆向] 一次Unity引擎gal游戏TextMesh Pro字库汉化记录

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1588559
- 抓取时间: 2026-11-01T23:36:13

## 一、原由

受别人邀请接手了一个Gal的汉化，发现是Unity引擎的，并不是原版厂商常用的引擎。因为之前也没有接触过Unity，好一番折腾，总算是成了，记录一下曲折过程，顺便分享一下经验。

个人水平有限，写得不好还请多多谅解。

## 二、初步分析

从群友那里拿到游戏文件，双击直接可运行，不用[//www.52pojie.cn" target="_blank" class="relatedlink"](https://www.52pojie.cn:href="https)，lucky~

进步一观察游戏资源，发现为Unity引擎：

![](https://attach.52pojie.cn/forum/202202/16/085623rjj4v0xvzj444q4c.png)

QQ图片20220216085550.png (32.52 KB, 下载次数: 226)

[href="forum.php?mod=attachment&aid=MjQ5NTA3N3xlYjRlYTc5OXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA3N3xlYjRlYTc5OXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 08:56 上传

进入`Managed`目录，发现是原版Mono打包的，没有用`IL2CPP`加密，lucky~，对我这种新手来说太好了。

![](https://attach.52pojie.cn/forum/202202/16/085815yhpdk8judhf9jrtk.png)

QQ图片20220216085751.png (62.79 KB, 下载次数: 223)

[href="forum.php?mod=attachment&aid=MjQ5NTA3OHwyNzQzZmNlMHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA3OHwyNzQzZmNlMHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 08:58 上传

网上搜索了一下，Unity引擎的脚步存放在`Assembly-CSharp.dll`中，没加密就是C#的，`ILSpy`安排上。

完美反编译，美滋滋~，代码保存成C#工程，计划通~

![](https://attach.52pojie.cn/forum/202202/16/090211ebh36kpkl3hb3f9k.png)

QQ图片003.png (21.98 KB, 下载次数: 219)

[href="forum.php?mod=attachment&aid=MjQ5NTA5NXw2MzQ5ZTUyNHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5NXw2MzQ5ZTUyNHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:02 上传

进一步分析游戏的资源的构成，发现`StreamingAssets`目录下有我们感兴趣的脚本封包

![](https://attach.52pojie.cn/forum/202202/16/090511d3hh3m65eil04u4m.png)

QQ图片004.png (46.83 KB, 下载次数: 221)

[href="forum.php?mod=attachment&aid=MjQ5NTA5OXxiMjQ4Njk1OHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTA5OXxiMjQ4Njk1OHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:05 上传

在保存的C#反编译项目中搜索 "script 找到如下代码：

![](https://attach.52pojie.cn/forum/202202/16/090845chfxxi3hizaw6khs.png)

QQ图片005.png (85.11 KB, 下载次数: 228)

[href="forum.php?mod=attachment&aid=MjQ5NTEwNXw3NTEyZmY3OHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTEwNXw3NTEyZmY3OHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:08 上传

在这个类里继续往下翻，发现获取脚本的函数：

![](https://attach.52pojie.cn/forum/202202/16/091212g0cw6tlr169vw4bm.png)

QQ图片006.png (24.35 KB, 下载次数: 225)

[href="forum.php?mod=attachment&aid=MjQ5NTExM3w3Y2JkNjk3NXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTExM3w3Y2JkNjk3NXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:12 上传

顺腾摸瓜，看看哪里调用了这个函数，发现是`AssetManager.cs`中的`GetScriptData()`函数调用了它，继续回溯：

![](https://attach.52pojie.cn/forum/202202/16/091647hjhxc6x22zcbqjcx.png)

QQ图片007.png (96.94 KB, 下载次数: 220)

[href="forum.php?mod=attachment&aid=MjQ5NTExNHwzOTZiZTNjZXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTExNHwzOTZiZTNjZXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:16 上传

方框里就是脚本的读取了，包括`string`列表，这样脚本就解密完了，感觉很顺利呢，一天可以搞掂的样子~

## 三、曲折

写解包工具，提取了日文版的文本（这个版本是英文版的），测试顶一下，修改源代码，编译，一气呵成~

![](https://attach.52pojie.cn/forum/202202/16/092128k9qqqqc0co00s040.png)

QQ图片008.png (15.93 KB, 下载次数: 230)

[href="forum.php?mod=attachment&aid=MjQ5NTExNnwxMDlmYzIzZXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTExNnwxMDlmYzIzZXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:21 上传

发现有很多框框，马上意识到是字库有问题，找找文字输出的函数吧，找了一会找到一个`DisplayText()`函数，看起来像是 （人名，对话）

![](https://attach.52pojie.cn/forum/202202/16/092518c4maabd4amvrbooa.png)

QQ图片009.png (54.15 KB, 下载次数: 224)

[href="forum.php?mod=attachment&aid=MjQ5NTExN3w2NGY0NGVmMHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTExN3w2NGY0NGVmMHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:25 上传

转到函数，发现`NameArea`和`TextArea`

![](https://attach.52pojie.cn/forum/202202/16/092757yrnt2l5pwffypw77.png)

QQ图片010.png (27.04 KB, 下载次数: 228)

[href="forum.php?mod=attachment&aid=MjQ5NTExOHw0ZTgwODVlYXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTExOHw0ZTgwODVlYXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:27 上传

转到定义，来到了class `CStringEntity` : `MonoBehaviour`，发现`TextMeshPro` `TextObject`，

![](https://attach.52pojie.cn/forum/202202/16/093020e6cbkblggf1bbhgh.png)

QQ图片011.png (59.58 KB, 下载次数: 227)

[href="forum.php?mod=attachment&aid=MjQ5NTExOXwwNmIzYWRiM3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTExOXwwNmIzYWRiM3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:30 上传

进一步分析发现此时代码已经转到`Assembly-CSharp-firstpass.dll`中了，我意识到这可能是系统组件，网上搜索一番

![](https://attach.52pojie.cn/forum/202202/16/093247z3kru1a93zez1une.png)

QQ图片012.png (128.05 KB, 下载次数: 223)

[href="forum.php?mod=attachment&aid=MjQ5NTEyMHwyZTIwMjhmMHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTEyMHwyZTIwMjhmMHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:32 上传

果然，下面就是汉化字库就完成了，看起来还挺简单的，网上搜搜，看看有解决方案没有，还真找到了：

https://www.bilibili.com/video/BV1D5411V7MK

嗯，对着操作一番，首先下载unity，看起来要对应的版本才行，OK，主程序上属性一下，发现是`Unity5.5`版本的：

一般正规的软件厂商都会带版本信息，还是比较可靠的，下载，安装，一气呵成~，我，今天，就！要！搞！掂！它！

![](https://attach.52pojie.cn/forum/202202/16/093619ibm374414zsrpmg8.png)

QQ图片013.png (18.24 KB, 下载次数: 226)

[href="forum.php?mod=attachment&aid=MjQ5NTEyM3w0M2IyZTg0NXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTEyM3w0M2IyZTg0NXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 09:36 上传

好的，安装`TextMeshPro` 插件，嗯，按网上说的进AssetStore搜索，嗯？怎么会没有？网友留了地址，点进去，嗯？？？怎么是404？？？

what fuck？？

上管网搜索，好嘛，该软件已下架，我陷入了深深的思考。

没关系，面向百度编程嘛，找到一个 `Text Mesh Pro U5.unitypackage`的包，导入，安装，然后报错，修改，然后报错，看来是版本不对，再搜。

CSDN CSDN CSDN，好嘛，全是CSDN，没分，没钱。

偶然间找到一篇文章介绍 `TextMeshPro`的，了解到该字库最开始是由个人开发的收费软件，后来Unity官方觉得不错，收购了，从unity2018版起集成到了Unity引擎中，但是官方app依然提供下载。

俗话说得好，凡事都怕但是，看来2022年官方觉得这么多年过去了，是时候淘汰老版本了。

好嘛，没关系，梯--子打开，面向google编程，翻了两天英文网页，我终于找到了1.0.56 b2 1.0.56 b3 1.2.2版本for unity5.5 unity5.6 unity2017.1 unity2017.2 unity2017.3。

查了一下游戏发行时间2017年，1.0.56 b2安排上。

字库制作中，略过不表，网上教程很多，大同小异。

这里有一个选择，按照bilibili的教程，应该生成字库后提取，然后替换原来的字库，但是我看评论区有为朋友提到：

`Font Asset Creator`生成好字体（类型为`TMP_FontAsset`）后，做一个包含`TMP_FontAsset`的`AssetBundle`。

之后在代码中加载`AssetBundle`，获取你做好的`TMP_FontAsset`，直接替换掉游戏原本的字体。

我心动了，这种才是完美解决之道，才是程序应该做的，直接替换破坏了原来的数据，不好不好。

面向Unity编程，打包打包。

[C#] 纯文本查看 复制代码
