
# [Web逆向] 一个技术网站的反反爬和模拟镜像网站搭建方法

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990765](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990765)
- 抓取时间：2026-11-01T22:18:07

无意中在搜索技术资料的时候发现一个看似简陋但是内涵丰富的小网站，对于我经常查的东西非常有帮助。但是在线的方式总是会受到网速的影响，要想做的随时随地的快速查阅。随即想到这个网站既然这么装修简单，应该很容易就能把它的主要内容下载下来，存到手机或者电脑上，用 NodeJs 搭建一个本地的网站去提升查阅的速度和效率。但是事实证明，这个网站采用了非常让人费解技术，让我走了不少弯路，这些弯路在大佬们看起来应该都是非常简单技术，但是实在是坑有些多，这些小坑叠加起来也非常考验耐心。下面看一下这个网站的主页和里面的主要内容（为了不违规，网站的地址在帖子中均做脱敏处理）。

这是一个 3GPP 常用技术规范的查询网站：

![image.png](https://attach.52pojie.cn/forum/202412/12/213850cud8f8f8djxv1xfy.png)

进去看了一下，整理的还挺不错：

![image.png](https://attach.52pojie.cn/forum/202412/12/214242z8ozkavnn3nvbd9m.png)

![image.png](https://attach.52pojie.cn/forum/202412/12/214424iorujr1y6sygnjtj.png)

里面的内容还是非常干货多多，符合我的需求。随即尝试开干。

## 第一招：用 `wget` 进行递归的爬

按照以往的经验，简单的网站用这个方法，能把大部分网页和脚本给爬下来。

采用如下命令：

```
wget --mirror --user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:91.0) Gecko/20100101 Firefox/91.0" --page-requisites --no-parent --no-if-modified-since 这里跟网站的主页地址
```

爬完后的内容主目录如下：

![image.png](https://attach.52pojie.cn/forum/202412/12/214803jo64kqf7r9997ger.png)

双击 `index.html`，欣喜了一阵，主页出来啦。

![image.png](https://attach.52pojie.cn/forum/202412/12/214852t880eeehuu0jxuc0.png)

但是随即点击任何一个链接，都让人一下子跌入谷底，每个页面都是直接光显示下面一行字。

![image.png](https://attach.52pojie.cn/forum/202412/12/215031uvivz14oxm124c18.png)

最开始我以为这个网站挑浏览器，换了 `firefox` 还是不行。而且这无法解释为什么同一个浏览器我直接打开就能看内容，爬下来就看不了内容。

## 检视爬取内容

于是我开始检视自己爬下来的内容，终于知道是咋回事了，原来爬下来的网页就是一些垃圾网页，大部分都是 1KB 的，只有少量的网页是由内容的。

![image.png](https://attach.52pojie.cn/forum/202412/12/215252qhuz86wjhapeb27z.png)

随便打开一个 1KB 的网页，看看里面的内容

![image.png](https://attach.52pojie.cn/forum/202412/12/215431j76fsviedyndzi0q.png)

就这么几行内容，换啥浏览器都不可能有有用的内容。尝试了几种其他简单的方法，无效，除非用浏览器手动打开，保存后的网页是有内容的。难道真要我开发一个浏览器去完全模拟这个过程么，代价太大，后来想想先手动搞搞，看看规律。

## 手动保存后的网页

如下是手动保存后的网页，打开是正常的。

![image.png](https://attach.52pojie.cn/forum/202412/12/225516m8983wt93s98ssss.png)

![image.png](https://attach.52pojie.cn/forum/202412/12/225256tfak4k9qtcg99ag2.png)

![image.png](https://attach.52pojie.cn/forum/202412/12/225632xavbz0bhjsge61bi.png)

有点遗憾的是上面的按钮功能是无用的了。在原来的网页上点击这个按钮可以显示或者隐藏详细内容，效果如下：

![image.png](https://attach.52pojie.cn/forum/202412/12/225759t5tkyxrh25kqktqx.png)

## 隐藏内容的处理

对于我来说，我希望的是一直显示出详细的内容，那么就相当于每次都要点一下这个按钮再手动保存。有些费事。后来突发奇想，感觉肯定是这个页面做了什么隐藏技术。是内容被隐藏了。

用文本编辑器分别打开隐藏了详细内容和显示详细内容的 Html 文件，发现了如果需要显示详情信息，只需要额外定义 就可以了。那这样就好说了，就可以先以默认的页面保存，然后写一个 `python` 脚本进行添加相应的定义就好了。

![image.png](https://attach.52pojie.cn/forum/202412/12/230355nnki1vfifvtrbrfq.png)

## 自动化保存脚本

最开始的时候是手动打开保存，但是半小时过后，手已经疼了。这样不是事儿，需要引入自动化才行。于是写了如下脚本来模拟这个过程：

![image.png](https://attach.52pojie.cn/forum/202412/12/231100jrj4wbwnbj1jezft.png)

本来最开始的时候我基本没有添加 `sleep` 的时间，想加快这个速度，结果欲速则不达，保存下来的网页大部分都是还是空白的加之前的提示，说明网页还没有生成成功就被我保存了，白白浪费了时间。后来经过我仔细微调才得到上面脚本中的参数，速度是降低了一些，但是起码是机器在做，我可以休息，慢慢等。

## 保存后的资源文件夹问题

随后经过很长时间，网页保存完毕之后，就得到了一个很多 `html` 的文件夹。但是也存在一个问题，就是保存网页的时候，存了好多与网页同名的文件夹，文件夹里的内容都是相同的，都是 Html 引用的 js 脚本和资源，如果保存了 200 个网页就会出现 200 个这样的文件夹，而且资源尺寸挺大的，占用了非常大的硬盘空间，其实只需要一份就可以了。我想删掉这些文件夹，

![image.png](https://attach.52pojie.cn/forum/202412/12/231651rx2emuujm2uxz20u.png)

![image.png](https://attach.52pojie.cn/forum/202412/12/231729doi3k3omz8ck683t.png)

![image.png](https://attach.52pojie.cn/forum/202412/12/232056o110sbv8b1h1z1sb.png)
