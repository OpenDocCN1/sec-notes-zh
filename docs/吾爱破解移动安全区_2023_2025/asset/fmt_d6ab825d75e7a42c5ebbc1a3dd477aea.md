
## 方法一

所以，就算 `onCreate` 方法我们修改不了，我们也可以主动添加一个 `onStart`，里面写 `doCopyAssets` 方法的调用，使软件在刚启动时将软件包 `assets` 文件夹内的 `.xml` 文件拷贝至 `shared_prefs`，从而达成变相去除弹窗的效果，这样操作的优先级也是足够的。

最后，我们只需要将修改后的 `.xml` 文件手动拷贝到软件包 `assets` 文件夹内，编译、签名、安装即可。

![](https://attach.52pojie.cn/forum/202508/02/161609b7az7l909a09v098.jpg)

到测试阶段，发现第一次打开软件仍然弹窗，但是杀后台后，第二次打开就不弹窗了。

![](https://attach.52pojie.cn/forum/202508/02/161510l3rb33nfanxe64x4.jpg)

之前提到，软件第一次启动会检测 `.xml` 文件存在与否，现在看来这个判断就存在于被抽取的 `onCreate` 方法，而 `onCreate` 的优先级高于 `onStart`，所以第一次判断时，我们在 `onStart` 中定义的复制方法还未被调用，这就导致第一次还会弹出窗口，而再次打开窗口消失。

一点点小瑕疵没什么关系，当然如果大家想在第一次启动时直接去除，存档注入方法仍然适用。我们只需要用添加启动图或类似云注入等工具，替换原软件的第一入口，再于替换后的入口 `onCreate` 中插入复制方法调用即可。只是由于篇幅和时间限制，我就不细说了，各位可以自己折腾(其实是懒

![](https://static.52pojie.cn/static/image/smiley/default/4.gif)

)

现在所有的思路都捋顺了，这个方法也顺利的结束。

![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)
![](https://static.52pojie.cn/static/image/hrline/line8.png)

## 方法二

### Hook 大法

既然弹窗都能被检测到，那么把它 Hook 掉也就没什么问题了。Hook 方法有的时候才是最实用的，因为有些软件会上非常恶心的加固。

我们再回算法助手，观察日志，可以看到窗口的文本竟然是可 Hook 的明文，这里我们可以直接使用算法助手或 `SimpleHook` 自带的屏蔽弹窗功能。但这样未免太偷懒，我就是想没事找事。

于是我决定自己手搓一个 `Xposed` 模块。我们用手机端的 `AIDE pro` 作为范例。

打开 `AIDE pro`，新建一个单应用 `Xposed App`，名称自定。

![](https://attach.52pojie.cn/forum/202508/02/161539zmfpc6m4ps4ls6ff.jpg)
![](https://attach.52pojie.cn/forum/202508/02/161541ziqzc3ve9f93c9te.jpg)

建立工程后，从侧栏定位到目录 `/storage/emulated/0/AideProjects/你的工程名/app/src/main/java`，即可看见 `xposed.java` 和 `xposedmain.java` 两个文件(模板不同可能文件名称、位置也不同)，我们这里只要修改 `xposed.java` 即可。

如果你不会写代码，可以直接丢给 AI，让 AI 帮你写(本来想偷偷懒，然后发现 AI 写的一坨屎

![](https://static.52pojie.cn/static/image/smiley/default/huffy.gif)

)，其实也不是很难，因为已经有大佬准备好模板了，只需替换即可。因本人略通 `JAVA`，准备自己写。

为防止屏蔽到软件内其他弹窗，切记不能全局 Hook，一定要对关键弹窗做判断。

![](https://attach.52pojie.cn/forum/202508/02/161544exnrhlhw6w6spxer.jpg)

半吊子还是半吊子，本来想通过关键词屏蔽弹窗，但怎么写也取不到关键词，Hook `Setext` 好像也没用，有没有大佬来教教

![](https://static.52pojie.cn/static/image/smiley/default/30.gif)

？

但是这里我想出一个很逆天的方法，由于软件启动后，要去除的弹窗第一个弹出，且没有其他弹窗附带，于是我写了一个计数器，计算屏蔽弹窗次数，只对第一个弹出的弹窗进行一次屏蔽，屏蔽完后就躺平，成功解决问题

![](https://static.52pojie.cn/static/image/smiley/default/victory.gif)

以下是完整代码的链接，需要的无脑复制粘贴即可：

ht删tps://share.we删iyun.com/CC删Qc4atC

编辑完文件后，戳右上角的编译键编译，然后安装模块，勾选模块作用域即可屏蔽弹窗。

但是这里考虑到有很多同学没有 root，所以我们再用 `LSPatch` 内置模块。

![](https://attach.52pojie.cn/forum/202508/02/161547hjl9slzq5ze5crn9.jpg)

在内置模块时，发生了一件很有意思的事。`LSPatch` 报错了。

![](https://attach.52pojie.cn/forum/202508/02/161549r4jep7zumweg92e3.jpg)

贴出详细内容：

```
Launch patch
, Processing /data/user/0/org.lsposed.lspatch/cache/apk/趣打印_12.15.04_anti.apk -> /data/user/0/org.lsposed.lspatch/cache/apk/趣打印_12.15.04_anti-434-lspatched.apk
, apk path: /data/user/0/org.lsposed.lspatch/cache/apk/趣打印_12.15.04_anti.apk
, Parsing original apk...
, Register apk signer with default keystore...
, Failed to parse AndroidManifest.xml
, D3.a: Failed to parse AndroidManifest.xml
        at u4.b.a(Unknown Source:1360)
        at c4.c.n(Unknown Source:23)
        at y3.a.l(Unknown Source:5)
        at O3.A.run(Unknown Source:115)
        at com.google.common.util.concurrent.g.run(Unknown Source:152)
        at V3.j.run(Unknown Source:2)
        at V3.a.run(Unknown Source:90)
```

阅读错误日志发现，`LSPatch` 在解析软件 `AndroidManifest.xml` 时出错了。一开始以为是同名文件夹干扰，但后来一想我已经反混淆了，不存在这个问题。

再转到 MT 管理器，发现资源文件正常打开。

![](https://attach.52pojie.cn/forum/202508/02/161551pl9sfnlf5uc2ydfg.jpg)

什么鬼？这是什么情况？马上就要成功了卡我一下

![](https://static.52pojie.cn/static/image/smiley/default/cry.gif)

但是我们不能放弃。我切到了 np 管理器，发现在打开 `AndroidManifest.xml` 时，np 也报错了:

![](https://attach.52pojie.cn/forum/202508/02/161553kea525cp2x5ptpvp.jpg)

就连俄罗斯大神写的 `Apktool` 反编译也显示错误。

这里我分析，可能是该软件中的 `AndroidManifest.xml` 格式损坏，导致大部分编译器无法识别。但为啥还能正常安装，我就不知道了，有没有大佬能够解释一下？第一次碰到这种问题。

这时我突然想到个方法，既然 MT 管理器能正常打开，那么我们就随便往里写点东西再保存，说不定损坏的地方就被自动修复了。

![](https://attach.52pojie.cn/forum/202508/02/161556avudyhvhdnyvnnnv.jpg)

保存成功，再拿到 `LSPatch` 里补丁，就可以正常修补了。

![](https://attach.52pojie.cn/forum/202508/02/161558n86od85o6ud39p13.jpg)

修补完测试时，软件第一次打开好像会闪退，重开几次才正常。我猜想可能是函数抽取和 `LSPatch` 补丁环境不兼容的原因，但是在测试中，用网友分享的魔改版 `ZPatch` 内置模块完美运行。

经过一波三折，弹窗被成功去除。

![](https://attach.52pojie.cn/forum/202508/02/161513nfso7g1qsbabzsw5.jpg)

至此，帖子的分析过程结束，能耐心看到这里的都是大佬。本人技术能力有限，各位勿喷，主要还是想为小白普及一点逆向知识。目前我只想到这两个方法，感觉改 `so` 层不太现实，毕竟函数被抽。如果大家有不同见解，欢迎分享评论，也希望有大佬能够解答帖子中的问题。

感谢大家的阅读！码字不易，点赞可有？

帖子内所有工具及原包都已放在下面，需要练手自取

ht删tps://jiguro.lanz删ouw.com/ivC143删2c13vi
