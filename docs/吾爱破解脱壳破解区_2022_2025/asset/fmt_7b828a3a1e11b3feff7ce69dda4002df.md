
# [原创] 纯爆破流强制去水印、对比走位去时间、授权的流程、强制干翻`ReportForm`异常、文档为只读

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1725952
- 抓取时间：2026-11-01T23:03:44

## 前言

本帖不能让你一次性成为[大侠](https://www.52pojie.cn)。

但是我们通过各种实验来推演各种复杂的变化、落点，制造不同的结局。

就像玩五子棋一样，等跳转的运用、落子的时机、顺序、棋型、走位，最终影响胜率的不同变数。

最终我们可以总结出各种公式化、实例化，具有某种特性和规律的东西，从中享受到破解带来的奇趣艺术体验。无论学思路，还是编程，都能从中得到最大的启示。

- 强制去水印输出、随机字符串乱码、随机禁用超链接
- 大力暴力改
- 二力暴力改（前两种方式简单暴力，但是！经不起时间的考验）
- 轻微暴力改（慢慢细跟，根据输出慢慢修改；亦可针对16进制文件直接下刀）
- 对比走位法去时间（利用`x64dbg`的插件）——扩展分支：编程实现文件夹、文件、注册表键值的监控新技能和升级《法王版`x64dbg`走火入魔2.0版》
- 授权的流程——扩展分支：编程实现《对比走位三度比较器报告器2.0版》
- 启动过程中授权注册的奥秘
- 授权文件藏身地点与真假授权文件
- 注册始终无法成功的4个原因与干扰因素
- 强制干翻`ReportForm`异常（利用软件的`bug`）
- 解除文档为只读（编程抽码法）
- 去恶心提醒`label`（让软件崩溃）
- 意外获得的宝藏（伯乐识宝马篇）
- 额外的启迪与思考

> `CR = width`，best wishes from crack
>
> 对破解者最好的祝福

里边竟有这句话，可见该软件收关注程度还是比较高的，功能还是可圈可点的。

每天晚上继续更新传奇故事：

这是老毛子PJ网站上极其牛逼的软件；作为为数极其少的所见所得的CHM编辑软件，它是唯一的一款完整版的。

另外我知道的那款叫`CHMEditor`，不少坑爹网站都用Demo版修改而成的。而今天咱们说的这个主角，则是完全版的哟~~

另外再给大家科普下，CHM编辑类软件无非分为3种：

- 第1种是解包后编辑，诸如国内知名的`SuperCHM`、`EasyCHM`……太多了。
- 第2种就是咱们上面提到的，像`EasyCHM`、`HelpAndManual8`、`WinCHM Pro`。
- 第3种编辑转码或另存输出的，像`My Notes Keeper`这类的。

但是前面说的该网站上面给出的不少爆破品不少是坑爹的，是经不起时间的考验的。

我下载的算是比较新的版本吧，但是安装输出一个`chm`之后就会发现注册成功是假的；因为输出有乱码字符+随机水印。

于是又找零下七度表哥要了一个比较老的绿色注册版的6.0的测试；结果发现一旦输出同样有上述问题，所以，还是自己动手的最靠谱。

毕竟又有乐子，还有成就感，那自己做主的感觉就是好啊，想怎么修改就怎么修改。下面截图请出今天的主角来吧。

这个软件比较拖沓地研究了几个月时间吧，算是一个总结和交待吧，几天是不太可能全部交待明白的。所以边编辑、边修改、边总结、边说明吧。

## 第1章 功能限制汇总与`Delphi`编程实现的遐想

![image.png](https://attach.52pojie.cn/forum/202212/13/192846vqgq0zayzzy7htyj.png)

`image.png`（119.21 KB，下载次数：1）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAxNHw4YmE5NDZjMnwxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes)

2022-12-13 19:28 上传

粉圈1、2、3：授权`key`、试用过期、未注册评估字样（其中圈3又与“关于”中调用的内容相同）。

![image.png](https://attach.52pojie.cn/forum/202212/13/193126hgr1qs5s32j6wz11.png)

`image.png`（96.12 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAxNnw3NWZmOTg5ZHwxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes)

2022-12-13 19:31 上传

未注册发布的页面，圈4。

图圈5（以发布`*.chm`文件为例）。

发布后的文档。

输出后的随机水印+随机字符串限制。

![image.png](https://attach.52pojie.cn/forum/202212/13/202618xdwzpzpupbzpnwq8.png)

`image.png`（235.88 KB，下载次数：1）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAyN3w0ZWMyOTVkOXwxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes)

2022-12-13 20:26 上传

打开`RegSPY`一眼我就知道过期的标志是哪一个，楼下的你能说出为什么吗？

![image.png](https://attach.52pojie.cn/forum/202212/13/202755jc1aka3zn2rra1er.png)

`image.png`（105.41 KB，下载次数：0）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAyOHw1OTFkZGMzNnwxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes)

2022-12-13 20:27 上传

上来还是四处搜集信息，这些你都能说出个所以然来，后面的就更易理解。

## 第2章 输出后的随机水印+随机字符串限制

![image.png](https://attach.52pojie.cn/forum/202212/14/210722zgg6pge5gpe5kzep.png)

`image.png`（697.04 KB，下载次数：1）

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTI0NnxmNDBjMDlkOHwxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes)

2022-12-14 21:07 上传

先来解决时间过期问题吧。

### 干掉时间限制

- 能想到的传统方式：`inc`变`dec`
- 找到源头永远重置为空
- 使用大白补丁注入或`HOOK`
- 论坛`bester`姥爷的玩法：`HOOK GetLocalTime`补丁（内嵌补丁）
- 找到过期的地方修改
- 法王姥爷最爱的方式，把注册表写记号的动作干掉

### 可疑的几个`API`

- `FileTimeToLocalFileTime`
- `FileTimeToSystemTime`
- `GetLocalTime`

利用插件、脚本、跟踪3种方法来对比走势和最终的分水岭。

都需要用到1个开始的`VA`和结束的`VA`。

猜测哪种方式更少、更快、更好？

- 开始`VA`、结束`VA`+跟踪窗对比
- 开始`VA`、结束`VA`+插件对比
- 开始`VA`、结束`VA`+脚本对比

先是`GetLocalTime`。

接着：

```text
01C8F255 | E8 9A9F78FE | call
```

接着：

```text
01C8F280 | E8 6F9F78FE | call
```

后面就是下面的流程了：

过期原因：`HKEY_CURRENT_USER\Software\Classes\RichOleLink.TOleLink.1\CLSID`检测此记号。
