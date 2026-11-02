
```
        public static TMP_FontAsset chsFont;
        public static void LoadChsFont() {

            string chsfont = Path.Combine(Application.streamingAssetsPath, "CHS/jissdf");
            LogPrint(chsfont);
            AssetBundle assetBundle = AssetBundle.LoadFromFile(chsfont);
            if (assetBundle != null) {
                foreach (string tt in assetBundle.GetAllAssetNames()) {
                    LogPrint(tt);
                }

                chsFont = assetBundle.LoadAsset("jisSDF");
                if (chsFont != null) LogPrint("asset sucess! " + chsFont.GetType().ToString());
                else LogPrint("faild,faild");
                assetBundle.Unload(false);
            }
            else LogPrint("load AssetBundle faild");
        }
```
运行，faild，修改，运行，faild faild faild faild，情况不对啊，在自己的Unity中运行，成功，嗯？？？

我悟了，看来是Text Mesh Pro的版本不对，我顿时有了一个大胆的想法，干嘛不引用新的Text Mesh Pro版本，把原来的Text Mesh Pro忽略掉，说干就干。

修改反编译的C#工程代码，引用 `TextMeshPro-5.5-1.0.56-Runtime.dll`，命名空间引用别名，解决命名冲突，编译，一气呵成~

load，成功！！！，yes~，进游戏测试，嗯？怎么文字全不显示了？继续改改，改改，还是不显示。好嘛，看来要多学一点Unity的知识，面向bilibili学习中。。。

## 四、解决

经过一番学习之后，我终于发现：Unity的场景工作模式是这样的，你要先再场景中放置一个对象，然后给对象绑定一个脚本，在脚本里编写你需要的代码。

回到游戏中，使用 `AssetStudio` 查看场景文件，发现场景中的对象是绑定了具体哪个 `dll` 程序集的，而且很多相关对象都是绑定了同一个程序集，像之前那种移到另一个程序集，行不通，或者要改太多地方。

![](https://attach.52pojie.cn/forum/202202/16/101210x9n284r8jzbr0nwo.png)

QQ图片014.png (43.04 KB, 下载次数: 233)

[href="forum.php?mod=attachment&aid=MjQ5NTE0NHxiNzY0ODdhY3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE0NHxiNzY0ODdhY3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:12 上传

好嘛，还是得替换文件，而且我还发现，游戏里的 `TMP` 版本，跟我手头上有的还不一样！因为在VS中有提示某些函数已经废弃，看来游戏里的 `TMP` 版本，比我的要旧，

网上继续搜索，没找到。。。CSDN找到一个1.0.52的，淘宝六毛八下载下来。。。

安装，引用，嗯？这次发现VS中没有提示错误了，看来是这个版本了，顿时放心下来，看来是时候了！

这里说一下 `TMP` 字库的构成，使用 `AssetBundleExtractor` 查看生成的字库文件 `*SDF.asset`，有三个文件

![](https://attach.52pojie.cn/forum/202202/16/102000q0npa77nztrzpa00.png)

QQ图片015.png (25.76 KB, 下载次数: 212)

[href="forum.php?mod=attachment&aid=MjQ5NTE0NXw0ZmI2NTQxM3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE0NXw0ZmI2NTQxM3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:20 上传

`MonoBehaviour` 保存了字库的基本信息，包括文字索引

![](https://attach.52pojie.cn/forum/202202/16/102159cpq1ro8oiiqtm7bm.png)

QQ图片016.png (33.74 KB, 下载次数: 209)

[href="forum.php?mod=attachment&aid=MjQ5NTE0N3xlYjcxMDI0M3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE0N3xlYjcxMDI0M3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:21 上传

`Material` 保存了文字的效果，比如描边啊，阴影啊，`Texture2D` 保存的是字库的图片，就是一张大大的图上写满了文字

![](https://attach.52pojie.cn/forum/202202/16/102808jonhs6y8z0iikcho.png)

QQ图片017.png (249.43 KB, 下载次数: 224)

[href="forum.php?mod=attachment&aid=MjQ5NTE0OHw2NjM0YTRlYnwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE0OHw2NjM0YTRlYnwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:28 上传

OK，了解清楚我们照着视频操作，使用 `AssetBundleExtractor` 替换，保存，这里有个小细节，替换 `MonoBehaviour` 时必须首先点击 `ViewData`，查看数据，并且弹射的第一个对话框选 “是”，后面的对话框根据有没有具体的 `dll` 选否，原理我们后面介绍。

替换完成，保存，运行，当然是成.....

崩溃了，崩溃了。

思考.jpg
![](https://static.52pojie.cn/static/image/smiley/default/27.gif)

![](https://static.52pojie.cn/static/image/smiley/default/27.gif)

why？

检查生成的封包，发现替换的 `MonoBehaviour` 变成无法读取了，读取不出数据了。花了一段时间毫无营养的百度机器互相抄袭的答案后，我悟了。

我突然想起在反编译时看到的字段特性：

![](https://attach.52pojie.cn/forum/202202/16/103455j99u9zs2sy9lckoh.png)

QQ图片018.png (40.24 KB, 下载次数: 229)

[href="forum.php?mod=attachment&aid=MjQ5NTE0OXxlNWM1NTEwMXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE0OXxlNWM1NTEwMXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:34 上传

注意这个 `[Serializable]` 标识，C# 拥有自动序列化的特性，只需要一个 `[Serializable]` 即可轻松序列化反序列化实例，再对照看看我们 `AssetBundleExtractor` 看到的信息，破案了兄弟们，

以及为什么 `AssetBundleExtractor` 查看信息之前要寻找 `dll`，就是通过反编译 `dll` 文件查找类属性，执行反序列化操作。自动序列化是一个好东西，但是这玩意是双刃剑，序列化是轻松了，

但是如果有任何字段的调整，比如添加新的序列化字段，之前序列化的东西就是无法再次在新的类上执行反序列化，表现出来的就是版本问题，版本不一样，你无法实现替换。

验证我的想法，`AssetBundleExtractor` 导出老的 `TMP` 信息，和我生成的 `TMP` 版本信息，一比较，果然：

![](https://attach.52pojie.cn/forum/202202/16/105036qn8kkz937d3zd7dx.png)

QQ图片019.png (38.17 KB, 下载次数: 223)

[href="forum.php?mod=attachment&aid=MjQ5NTE1Nnw4MWNmYWM0Y3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE1Nnw4MWNmYWM0Y3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:50 上传

![](https://attach.52pojie.cn/forum/202202/16/105146ulwaclbw9sw9kuwb.png)

QQ图片020.png (33.12 KB, 下载次数: 220)

[href="forum.php?mod=attachment&aid=MjQ5NTE1N3xkNWZkZmE0Y3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE1N3xkNWZkZmE0Y3wxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:51 上传

新生成的字库比旧的字库信息多出来几项，删掉它。

字库信息还比较接近，`Material` 几乎就是面目全非了，完全没有修改的价值，好在游戏里的文字特效比较简单，就一个描边和阴影

了解了一下 `TMP` 的文字特效设置，找到阴影设置项

![](https://attach.52pojie.cn/forum/202202/16/105524sq3q22bb33sbqy3y.png)

QQ图片021.png (23.24 KB, 下载次数: 203)

[href="forum.php?mod=attachment&aid=MjQ5NTE1OXwzNjc0NjEwNXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE1OXwzNjc0NjEwNXwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:55 上传

`_UnderlayOffsetX` 是阴影的X轴偏移，同理Y是Y轴偏移，`_UnderlaySoftness` 是阴影的虚化程度，这里我都改为0，因为已经有描边了，阴影效果不好，主要是我换了字体，原来的设置不适用。

修改完成后，导入，再测试，完美，收工收工，前前后后折腾了快一个礼拜了。

![](https://attach.52pojie.cn/forum/202202/16/105912jhhqqkwmihkgnz06.png)

QQ图片022.png (27.33 KB, 下载次数: 218)

[href="forum.php?mod=attachment&aid=MjQ5NTE2MnxkNWI0ZjYyOHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE2MnxkNWI0ZjYyOHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5&nothumb=yes)

2022-2-16 10:59 上传

## 五、总结：

unity引擎汉化要特别注意版本问题，实在找不到原版本，找个差不多的版本来改吧。

文中并没有介绍各种工具的操作方法，具体使用百度上有很多了。

最后给大家分享一个我收集的各版本的Text Mesh Pro，不妨收藏一下，万一哪天用上了呢。

![](https://static.52pojie.cn/static/image/smiley/default/lol.gif)

![](https://static.52pojie.cn/static/image/filetype/text.gif)

[href="forum.php?mod=attachment&aid=MjQ5NTE2N3wwYjlkMWE2MHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTE2N3wwYjlkMWE2MHwxNzkzNTc2MTcwfDE2ODA4NnwxNTg4NTU5)

(194 Bytes, 下载次数: 118)

2022-2-16 11:09 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB
