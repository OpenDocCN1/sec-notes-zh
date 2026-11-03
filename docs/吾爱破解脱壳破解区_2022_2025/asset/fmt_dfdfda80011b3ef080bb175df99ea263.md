
# [Web逆向] 武海笔院书籍的 PDF 下载

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1938457
- 抓取时间: 2026-11-01T22:28:25

文武双全天灵根。

泉海碧波映佳人。

书笔怎写羞羞事。

局院深处躺平时。

== 欸~道友留步呀，这么着急走作甚？你还没说这 武海笔院 是何处呢？还有，这什么打油诗？

=> 这，＞︿＜ 哎呀，真是的 ，每次都要这样吗？……（四顾无人，施展好不容易修到大圆满境界的隔音术，再悄悄说道）你把每行诗句的开头，对，就开头第一个字连起来看看。

== 啊？哦~~哦！原来如此！(≧∇≦)ﾉ 道友着实有心了，不过下次可以不要取奇怪的标题吗？别人看不懂也搜索不到这里呀。

=> 咳咳，我也是没办法，所以只能看谁有缘了。

== 好吧，那这 “文……

=> 住口！不要说下去了，千万不要说出那四个字！！！

== ？？(＃°Д°)

=> （待到平复心中的惊慌，再运起大圆满境界的传音术）唉……我也不想如此呀，此四字为一灵兽的真名，传闻此灵兽道法通天，在此间（注：此间指逆向星域的web 大陆）不可直呼其名，否则冥冥之中感应到我的存在，仅投下一道视线，我必灰飞烟灭、不得入轮回！

此次也不过是远远瞧见了它的一道分身飞过天边，借用留影石保存画面罢了。

==（⊙ｏ⊙） ……嘶~（倒吸一口凉气）原来如此！多谢道友告知，那我一睹其英姿就走！

=> 别急，给，这是本文结构。
- 简述下载书籍的方案（留影之术）
- 源代码的获取、程序在本地运行的环境配置
- 程序的使用方法
- 具体的分析流程（断因果）。此部分现在可以忽略，等到将来网站变动时可供参考

## 缘起

由于一些不可言说的理由（不能说，就算一直盯着我也不能说），我凑齐五十下品灵石前往 武海笔院 购买书籍。

武海笔院 的书籍有两种阅读方式：在网页端看、下载电子书用它的软件看，都不能带出去，所以我想要将书籍下载下来，选择的方式自然也是：分析网页端，爬取它的图片并生成 PDF。所以此种方式下载的 PDF 是无法复制文字的。

> 

想要下载完整的书籍需要先用灵石购买它，也就是获得该书籍的阅读权限。

我先去散修城的藏经阁寻找方案（指搜索文章、github），前人几乎都是基于 web端爬取图片、合并成 PDF。仔细分析了他们的实现方案、并结合现在灵兽分身的踪迹（指网站的工作方式），发现这些方案容易和该灵兽分身（浏览器）有牵连，会吸引其本体的注意从而被灭（如封 IP、封账号），就像这样：

![](https://attach.52pojie.cn/forum/202406/26/174430q5mxmjmmmpc5fc50.png)

image-20240625213526732.png (68.28 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwNDAxNHxmMDE4MmE1MXwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNDAxNHxmMDE4MmE1MXwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:44 上传

最后，我于高山之上闭关 1 天另寻他法，且看后文分析。

## 留影之术

下面让我们先探讨现有的、下载图书的方式，最后说明我的解决方案 —— 留影之术。

```
# ======= 回忆长廊 =======
        启动
```

# 当前进度：查看书籍的每一页是怎样的结构

查看网站 HTML 结构，发现每一页的内容由 6 张小图片构成，如下：

![](https://attach.52pojie.cn/forum/202406/26/174322x5sj7iahn00hxsi2.png)

image-20240623121839423.png (270.78 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcwMzk5MXw3ZWRjY2RjZHwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMzk5MXw3ZWRjY2RjZHwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:43 上传

通过浏览器的抓包分析，找到这 6 张小图片的请求。

![](https://attach.52pojie.cn/forum/202406/26/174320qh2sb9dp6ho6rv6k.png)

image-20240623121422143.png (278.16 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwMzk5MHwwNDMxYzE4YnwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMzk5MHwwNDMxYzE4YnwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:43 上传

显而易见地：我们需要爬取这些小图片、合成每一页的图片，最后合并所有的页，得到一个 PDF 文件。

现在让我们仔细探讨现有的爬取方式。

### 模拟请求

不推荐此方式。

经过测试，发现在鼠标滚轮滚动时会触发一个 `save` 请求（滚动一次就触发一次）。

![](https://attach.52pojie.cn/forum/202406/26/174334sgmesu7qcrkss28e.png)

image-20240623220649132.png (118.42 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwMzk5NXxmMzMzMTQyMHwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMzk5NXxmMzMzMTQyMHwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:43 上传

这是在做什么呢？这是在记录我的阅读时长，或者说阅读习惯，具体说明如下：

![](https://attach.52pojie.cn/forum/202406/26/174337tye4komooz6m28m0.png)

image-20240623221416276.png (279.62 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwMzk5NnxmMjZmNzE4Y3wxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMzk5NnxmMjZmNzE4Y3wxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:43 上传

模拟请求的方式有以下缺点：

- 虽然可以模拟上述的 `save` 请求，但这需要大量测试是否可行，我的帐号只有一个，所以不能轻易尝试。
- 考虑账号的安全性，模拟请求所耗费的时间、精力（分析请求、分析参数）以及可能承担的后果都使得我不推荐此方式。

### 自动化工具

不推荐此方式。

在部分解决方案中（注：此处原本想放一个链接，却发现其牵扯到灵兽的另一分身，不得不用我大圆满境界的打码术去除了）使用的是自动化工具 `selenium`，并且提到了“那 6 张小图片不能二次访问”，其中一个解决方案是用代码控制鼠标右键点击图片来保存。

自动化方案有以下缺点：

- 自动化工具是可以被检测的，账号只有一个，我要选稳妥的方式。
- 保存图片的时候用 `pyautogui` 控制鼠标、键盘，这样就不能在电脑上做其它的事情了。

### 我的方案：代{过}{滤}理捕获，留影之术

既然小图片只能访问一次，我的想法是通过拦截响应来获取图片。这可以写成浏览器插件的形式，不过考虑到要捕获请求了，最好和浏览器分离开来，所以使用代{过}{滤}理的形式！

没错！完全和浏览器隔开，不沾染因果，此乃留影之术的本质。

整个流程如下：

![](https://attach.52pojie.cn/forum/202406/26/174340lt0ssuvsf7lqmfii.png)

image-20240623223921856.png (198.8 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwMzk5N3w0YTU3YWFkZnwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMzk5N3w0YTU3YWFkZnwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:43 上传

当然它的缺点很明显：非常占用资源，因为浏览器会一直发出图片请求、并解析、渲染图片啦。

== 可是，(。・・)ノ 要怎么自动翻页呢？？！

=> 经过前文的分析，我们要稳妥一点，所以不要自动翻页。

== 啊？？那我该怎么办？w(ﾟДﾟ)w

=> 手动翻页呗，难不成自己翻页太快也要封账号？！大家可是都会量子波动速读法的！—— 此法术只有在发动的时候才能记住文字，一旦停止施法，读过的内容就全部忘记了，这也许就是该法术的代价吧。

== 什么鸡肋术法，我就是想自动翻页、自动下载！(○´･д･)ﾉ

=> ……这样，你给我 100 下品灵石，我帮你翻页，这样对你来说也算是自动的。

== 啊！突然记起来传法殿中是有这么个量子什么阅读法的 ╥﹏╥... 可是我宗门贡献分不够兑换呀

=> 真拿你没办法，既然有缘，再多说道一二。现在你可以用任何方式、只要能让浏览器翻页就行 —— 前端就是这样的，只需要好好翻页就行了，可代{过}{滤}理端要考虑的事情就多了。

== 嗯？这个句式好像有点眼熟 (´･ω･`)?

=> 咳咳……我认为比较稳妥的是：编写 `JS` 脚本实现滚动翻页（是让页面慢慢滚动从而翻页，而不是一页一页地、跳跃式翻页），不使用自动化工具来操作浏览器（不要和分身有任何牵连）。如此，自动翻页既安全、也可以做其他的事情嘛。

== 嗯嗯，然后呢？然后呢？

=> 放心，项目里有自动翻页的脚本啦。不过在我的测试中，自动翻页超过一定次数，会弹出错误信息（把我吓出一身冷汗，我甚至感觉到其本体的视线已经跨越时间与空间，将我钉在了此处，它似乎正看我的过去、我的来历，毕竟在他的眼中，小小练气修士是不敢招惹它的，如果它知道我没有强悍的背景，恐怕……额，好像可以无视警告继续翻页欸，咳咳，稳妥起见，可以暂停一会）。

![](https://attach.52pojie.cn/forum/202406/26/174434xyd0drjn1crcd0n7.png)

image-20240625231840243.png (12.62 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcwNDAxNnw4Y2MwM2IzY3wxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNDAxNnw4Y2MwM2IzY3wxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:44 上传

![](https://attach.52pojie.cn/forum/202406/26/174437azk0z0kcbu4838ug.png)

image-20240625231945960.png (117.76 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwNDAxN3w0NDExZTczNXwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNDAxN3w0NDExZTczNXwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:44 上传

### 重要声明

我并没有测试 “一个账号一天翻完了多本书会不会封账号”（这还要花钱买书测试呢！我也不推荐大家尝试），不过仔细想想，怎么说也是手动翻页的，不应该封号才对呀，除非它强烈抗议用户使用量子波动速读法。

== 可是，它都明确说了禁止下载……

=> 咳！这个嘛，尽管放心，我也只是自己用啦，我也不推荐道友将这个拿出去乱搞哈。

最后，重要事情说三遍。

建议一天下载一本书，账号只有一个，要用最稳妥的方式！

建议一天下载一本书，账号只有一个，要用最稳妥的方式！

建议一天下载一本书，账号只有一个，要用最稳妥的方式！

## 环境配置

此部分是大家拿到源代码、在本地运行之前的配置。

从 `https://github.com/Hosinoharu/WuHaiBiYuan_downloader` 获取源代码。

### 配置 Python 环境

首先需要有 Python 3.11 及以上版本（因为我写代码时用的这个版本，我也没有用低版本的 Python 进行测试）。

```
# 首先进入到项目所在的目录 WuHaiBiYuan_downloader
# 这里使用的是内置的 venv 模块来创建虚拟环境
# 该虚拟环境保存在当前目录（项目所在目录）下的 .venv 目录中
python -m venv .venv
# 然后启动虚拟环境
.\.venv\Scripts\activate
# 修改该虚拟环境 pip 的下载源，大家也可以用自己常用的那个
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/ --site
# 执行以下命令安装依赖包
pip install -r requirements.txt
# 测试核心的代{过}{滤}理软件是否安装正常
mitmdump.exe --version
# 执行上行命令之后会输出以下类似的信息
'''
PS > mitmdump.exe --version
Mitmproxy: 10.3.1
Python:    3.11.4
OpenSSL:   OpenSSL 3.2.2 4 Jun 2024
Platform:  Windows-10-10.0.19045-SP0
'''
# 然后启动代{过}{滤}理程序，开始配置证书
mitmdump.exe
```

### 配置浏览器的代{过}{滤}理

不同浏览器的代{过}{滤}理配置方式不同（有些浏览器无法单独配置代{过}{滤}理，只有配置到操作系统上），这个大家自己搜索吧。这里以 Firefox 为例，设置方式如下：

![](https://attach.52pojie.cn/forum/202406/26/174402sbe7e37mtm9m7e8m.png)

image-20240625161214176.png (133.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcwNDAwM3xiNmJjYWFjNnwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNDAwM3xiNmJjYWFjNnwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:44 上传

![](https://attach.52pojie.cn/forum/202406/26/174259nlf59f4l2171f2m7.png)

image-20240621141131373.png (93.86 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwMzk4NXwwZWRjOWE0ZHwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMzk4NXwwZWRjOWE0ZHwxNzkzNTcyMTAyfDE2ODA4NnwxOTM4NDU3&nothumb=yes)

2024-6-26 17:42 上传

然后点击 OK 完整代{过}{滤}理设置。

### 配置代{过}{滤}理证书
