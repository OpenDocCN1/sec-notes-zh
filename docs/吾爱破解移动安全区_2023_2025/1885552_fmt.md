
# [Android 原创] autojs简介与对抗

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1885552
- 抓取时间: 2026-11-01T21:30:21

## 前言

最近发现现在大多安卓工具很多都是autojs和按键精灵等这些非常易于开发的语言来做黑灰产自动化工具，此帖也是由此做为出发点来展开说说这个autojs，我们如何去做对抗。由于笔者对这块也是刚了解学习不久而且这篇帖子也会说的很简单，有很多不足的地方或者解释不充分还请大佬们指出。

## autojs介绍与版本

### autojs介绍

我们知道使用js来编写一个apk是多么的爽，ui部分不使用xml，代码部分不使用Java，全程使用js来编写对刚入门想要开发一款安卓应用来说是一件非常方便易学的事，所以就有了autojs这么一款语言。

使用官方的一句话就是

![](https://attach.52pojie.cn/forum/202401/28/154139m4rq1tfkqejjv16j.png)

pic1.png (21.08 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY3Mjk3MnxhNzlkODE0OHwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3Mjk3MnxhNzlkODE0OHwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 15:41 上传

是的，我也觉得autojs这款工具对于老人难以进行复杂的操作和子女进行微信视频提供了很大的帮助，具体有没有人这样做我也不知道。

### autojs版本

autojs的版本还是分得比较多，现在常用版本分为：autojs4.1、 autojs pro7、autojs pro 8、autoxjs（此帖子也是围绕这个版本来的）但是本身autojs的作者已经不更新了，并且这个开源项目也已经删除，原因是这款语言被太多的黑灰产利用了。还有一些其他的版本，但是主流还是上面那4个版本用的比较多，其中pro7和pro8都是收费版但是现在都被破解了。现在还在更新和维护的是autoxjs。

## autojs原理

1.这个软件的ui界面并不是由js写的。是这个软件提供了一个可以编写界面的js环境。这个软件本身的界面是由Java和Android XML编写的。

2.这是利用了`AccessibilityService`的API。参见`AccessibilityService`的`getRootInActivieWindow()`函数。

`common`模块提供了其他各个模块的公用类、工具等，例如一些数据结构、`View`工具类等。是其他各个模块的依赖。

`automator`模块实现了自动操作的大部分内容。包括选择器的实现、简单操作的实现、控件节点的封装等。是`autojs`模块的依赖。

`autojs`模块是Auto.js的JavaScript运行环境，包括脚本引擎的封装，核心运行库的实现，对JavaScript层暴露的API，JavaScript和Java的交互。同时提供了管理运行的JavaScript脚本的服务。

`app`模块是界面、业务逻辑。依赖`autojs`模块。

项目主要需要Android基础，和uiautomator基础没有太大关系

这里是转载来自：[//www.jianshu.com/p/1ff3f2c8e540" target="_blank" rel="noopener noreferrer nofollow"](https://www.jianshu.com/p/1ff3f2c8e540:rel="nofollow noopener" href="https)

以下为个人理解的autojs较为核心的模块，因本人也没有深入到源码去了解，可能会有出入欢迎指出。

![](https://attach.52pojie.cn/forum/202401/28/180646f7hjhq314yhzjwud.png)

pic2.png (98.04 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MzAwOXxjYzJlYzY3MnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAwOXxjYzJlYzY3MnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

### 执行流程

由于现在的autojs版本没有做太多混淆处理，而因为他本身就是一款模拟点击的app，我们去做分析的话可能不会有太多价值，比较源码里面也都是模拟点击的api，后续我们对抗肯定也不会从模拟点击的api入手，所以这个执行流程我简化为：读js代码-->有加密则执行解密-->先转换ui部分代码为xml代码再转换js代码为art字节码-->对界面做展示。

### 文件结构

怎么去判断一个apk是autojs开发的只需要找`assets/project`有js文件和一个json文件就可以判断，其次就是看`libc`库，里面有

`libjackpal-androidterm5.so`,

`libjackpal-termexec2.so`

这两个也可以确定是autojs开发的app

![](https://attach.52pojie.cn/forum/202401/28/180648kzrsx6i0sf0qc6xf.png)

pic3.png (168.52 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MzAxMHxlMzRmMzI1M3wxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxMHxlMzRmMzI1M3wxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

## 解密js

如果加密过的js肯定会执行解密，这里我们可以从两个地方进行hook拿到解密后的js

![](https://attach.52pojie.cn/forum/202401/28/180650lspjs8tlw4esltet.png)

pic4.png (38.02 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY3MzAxMXwyYzg0OGE3MXwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxMXwyYzg0OGE3MXwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

其实这两个点都可以做一个hook，我自己做的一个小工具也是把这两个点做了hook，首先就是第一个函数`StringScriptSource`这个点，他会传入两个`String`参数，一个是解密的文件名一个是解密后的js，可以把解密后的文件名和js做保存。

![](https://attach.52pojie.cn/forum/202401/28/180652e1lrsywsr1g70rw1.png)

pic5.png (20.91 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY3MzAxMnxkMzZiYTY1YnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxMnxkMzZiYTY1YnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

第二个就是解密的函数，如果hook解密的函数需要对返回值做转换然后保存即可，直接在这个函数做hook后面的逻辑我们也不用管。

![](https://attach.52pojie.cn/forum/202401/28/180654c1uum3mm7n14ams3.png)

pic6.png (20.25 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MzAxM3w0YjhlNDhkMHwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxM3w0YjhlNDhkMHwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

这两个点就已经可以拿到源码了，如果做了加固，再换以下`classloader`即可，这边贴出使用xposed做的hook

```
public void de_autojs(ClassLoader classLoader) {
    XposedHelpers.findAndHookConstructor("com.stardust.autojs.script.StringScriptSource", classLoader, java.lang.String.class, java.lang.String.class, new XC_MethodHook() {
        @Override
        protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
            super.beforeHookedMethod(param);
            XposedBridge.log("xiaoc----StringScriptSource" + param.args[0].toString());
            //XposedBridge.log("xiaoc----StringScriptSource"+param.args[1].toString());

            mainhook.main(param.args[1].toString(), param.args[0].toString());
        }

        @Override
        protected void afterHookedMethod(MethodHookParam param) throws Throwable {
            super.afterHookedMethod(param);
        }
    });
    XposedHelpers.findAndHookMethod("com.stardust.autojs.engine.encryption.ScriptEncryption", classLoader, "decrypt", byte[].class,
            int.class,
            int.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                }

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    super.afterHookedMethod(param);
                    String str = mainhook.bytesToString((byte[]) param.getResult());
                    XposedBridge.log("xiaoc_ScriptEncryption" + str);
                    mainhook.main(str, "de");
                }
            });
}
```

## 对抗思路

虽然我们上面拿到了源码，但我一开始也说了我们即使拿到源码也看不出东西全是autojs模拟点击的方法，如果要做对抗我们应把重点放到无障碍服务与风控上。

这也是我的主要对抗思路，下面介绍几种思路。

### 1. 无障碍列表可疑应用

获取已开启无障碍的应用，通过入口函数来查看，包含autojs字样或者stardust字样，直接闪退即可，但是使用这种方法也得考虑自己的用户场景，有误杀的可能，最好结合多种来进行检测。也可进行动态检测，如果发现此款app后端发现是黑灰产工具，也可以直接闪退(笔者有点不成熟，最好不不要使用哈哈，会有更好的检测手段)

`ResolveInfo{ea35d2a org.autojs.autoxjs.v6/com.stardust.autojs.core.accessibility.AccessibilityService m=0x108000}`

### 2. 主动抛出异常

上面的介绍图中说到，`AccessibilityService`这个是无障碍服务，如果没有这个服务我是实现不了模拟点击的，我们可以通过主动抛出异常，来检测控件是否被`Accessibility`控制按下或者滑动。

![](https://attach.52pojie.cn/forum/202401/28/180656kzzo5to1niu58mom.png)

pic7.png (170.6 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MzAxNHw2NGE5MDIzM3wxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxNHw2NGE5MDIzM3wxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

### 3. 轨迹变化特征

![](https://attach.52pojie.cn/forum/202401/28/180659py11qjljikggkaf2.png)

pic8.png (53.24 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY3MzAxNXw3YTcyYmE2NnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxNXw3YTcyYmE2NnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:06 上传

![](https://attach.52pojie.cn/forum/202401/28/180701ip54e5u7kaugu14u.png)

pic9.png (15.93 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY3MzAxNnxhODExNDRiZnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MzAxNnxhODExNDRiZnwxNzkzNTY4NjE5fDE2ODA4NnwxODg1NTUy&nothumb=yes)

2024-1-28 18:07 上传

通过以上两张图，我们已经可以很形象的看出来，通过轨迹特征来判断一个app被自动化脚本操作检测起来是多么简单事

### 4. 群控、按部就班

如果被群控了，那么肯定是多个用户在一个比较集中的时间内开始作业，而他们的作业路径非常统一都是按部就班的执行一切操作，此时我们可以先把这批账号拉入监控名单，一旦触发可直接封

### 总结

其实现在大厂的检测很多都是做的风控检测，也就是后面所说的两种，但肯定是远远不止这两种，主要还是围绕着来做，说起来简单做起来难，现针对模拟点击也没有一个很好的针对客户端来做检测，包括按键精灵、触动精灵等脚本等工具。总结下来还是做好风控。

## 笔者废话

整篇看下来很简单，内容也不多，都被简化了而且也挺多没提到，就好比无障碍这个服务是如何去调用并实现点击功能的整条链路，或许如果能去学习下无障碍这个服务怎么实现的一个点击或者说阅读autojs的源码会有更好的对抗思路，文章很简单，笔者也是做一个记录，后面如果有更好的客户端检测手法也会给读者们贡献出来，让这个圈子更卷
