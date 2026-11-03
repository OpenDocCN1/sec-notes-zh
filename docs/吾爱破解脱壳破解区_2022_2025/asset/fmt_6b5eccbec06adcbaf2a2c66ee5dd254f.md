
# [Web逆向] 一时兴起分析并绕过某网站验证码验证

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2071222
- 抓取时间: 2026-11-01T21:59:32

## 一、前言

先赞一个：前段时间偶然刷到这个网站 [xiaolincoding.com](https://xiaolincoding.com) 里面有很多优质内容，我只看过图解系统系列的部分章节，个人觉得讲解得非常好，一图胜千言，用图解的方式讲解知识深得我的喜欢。

![](https://attach.52pojie.cn/forum/202511/09/154851rfxrqfe88ff8xr77.png)

[`1_AgAABTMkRx53Lf_LvZNB-bCUO_0MHqb4.png` (334.05 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU2Nnw4M2E5ZjhkNHwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

1

2025-11-9 15:48 上传

昨天晚上又随手点开这个网站，随便看看，结果发现阅读全文又需要输入验证码，看来又过期了。其实我早就已经关注过公众号了，验证码可以在公众号里免费获取。但是此时我不想打开公众号去重新获取，因为内心感觉这个验证应该挺简单的，于是了绕过它的冲动。

![](https://attach.52pojie.cn/forum/202511/09/160754sp6lhhl01x7f5i5p.gif)

[`2_gif_small.gif` (2.25 MB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU4Mnw0NmNlZDM1MnwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

2

2025-11-9 16:07 上传

## 二、分析

由于我只有html、css、js的基础语法知识（很久没复习也忘得差不多了），平时也没做过web相关的逆向分析，连开发也没做过，工作也没用到过这些知识技能，所以一时兴起的我只能连猜带蒙，现学现卖，记录下此次的探索过程。

在网页中按F12这个我倒是知道的，查看请求和响应，在响应中可以看到，全文内容其实已经拿到了，只是没有显示出来：

![](https://attach.52pojie.cn/forum/202511/09/154853h7skxj76v47jb1z7.png)

[`3_AgAABTMkRx70JQh4fB1HVLuV_ovE19FL.png` (440.42 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU2N3w3M2U4NTEyYnwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

2025-11-9 15:48 上传

那么首先想的就是能否直接去除遮挡，当场搜索并写了一下代码，在控制台中尝试运行：

```
// 修改CSS显示属性

// 显示所有隐藏的元素
document.querySelectorAll('[style*="display: none"]').forEach(el => {
    el.style.display = 'block';
});

// 移除hidden类
document.querySelectorAll('.hidden').forEach(el => {
    el.classList.remove('hidden');
});

// 更暴力的方法：显示所有元素
document.querySelectorAll('*').forEach(el => {
    el.style.display = 'block';
    el.style.visibility = 'visible';
});
```

运行之后，看到网页有变化，稍微兴奋了一下，但是并没有显示出全文内容，而是把一些隐藏的元素显示出来了：

![](https://attach.52pojie.cn/forum/202511/09/160757uyou81rfu8c5fz2u.gif)

[`4_gif_small.gif` (2.64 MB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU4M3xlZWFiOWNkYXwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

4

2025-11-9 16:07 上传

接着猜想，应该是js代码控制的显示逻辑的，那就去看看js代码。

先随便输入一个验证码，会弹出错误提示：

![](https://attach.52pojie.cn/forum/202511/09/154855ehm0muxiojjxorxx.png)

[`5_AgAABTMkRx6yXQa7ArNMR65i4toOxbQo.png` (95.22 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU2OHxhOTM0MzgyZnwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

2025-11-9 15:48 上传

这个框不就是`alert`函数弹出来的么，这点知识我还是有的。

在源代码这里查看js代码，只在`app`这个js中搜到了`alert`相关的代码，全部打上断点。但是运行发现，点击【提交】按钮后并没有中断：

![](https://attach.52pojie.cn/forum/202511/09/154858a81m16mr3lr198m6.png)

[`6_AgAABTMkRx7zQM4xcb1DCK2etwhfYYAc.png` (398.13 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU2OXwwMmZmNTVmZnwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

6

2025-11-9 15:48 上传

【注】：这是第一次与成功失之交臂，由于当时没有仔细观察，上图左下角提示弹窗验证是由TechGrow提供，那么在源代码中就应该去qiniu.techgrow.co里看js代码，而不是xiaolincoding.com。

那么只能再转换思路了，找验证码的【提交】按钮在哪里，经过了几次尝试没找到，后来才发现，可以直接在【提交】按钮这里右键检查，直接就能定位。。。

![](https://attach.52pojie.cn/forum/202511/09/154900j8ssg8pzksb40ekn.png)

[`7_AgAABTMkRx4DCL-pKUVBBaPI5ES9EWZm.png` (334.65 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3MHw3YWZiNmM2Y3wxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

7

2025-11-9 15:49 上传

接下来就是找【提交】按钮对应的处理函数，但是浏览上图代码，并没有在按钮附近找到调用了什么函数。

那么又得现学了，经过搜索学习，原来在F12窗口中有【事件监听器】这个东西，在`click`里面找到了【提交】按钮以及对应的js代码为`readmore.js`：

![](https://attach.52pojie.cn/forum/202511/09/154903d43v1vql4wr1acy9.png)

[`8_AgAABTMkRx5Cv_xuGitJBJg3PSpJWo0W.png` (448.87 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3MXxjYWFiOTQ5M3wxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

8

2025-11-9 15:49 上传

点开`readmore.js`，好家伙，原来在qiniu.techgrow.cn里，刚刚看错了，不应该看xiaolincoding.com：

![](https://attach.52pojie.cn/forum/202511/09/154907qc2vs22j1e72cdec.png)

[`9_AgAABTMkRx7My96Mr9NMObQVNe305lnn.png` (155.27 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3MnxiMGIzM2Q0MXwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

9

2025-11-9 15:49 上传

大致浏览了一下`readmore.js`代码，很明显加了混淆：

![](https://attach.52pojie.cn/forum/202511/09/154910uarggtzuldgqqiyz.png)

[`10_AgAABTMkRx7epgfs7TtJCZkZOIF2ggHq.png` (325.98 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3M3xiZjMxMjVlNnwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

10

2025-11-9 15:49 上传

先搜索错误弹窗的文本和`alert`吧，"验证码无效"没有搜到，而`alert`有，又又又开始兴奋了。

![](https://attach.52pojie.cn/forum/202511/09/154914lwwxfzvzo09rse95.png)

[`11_AgAABTMkRx6vOXUCCChKlpbnSUCJZoVl.png` (21 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3NHw3OWZkZmUwN3wxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

11

2025-11-9 15:49 上传

![](https://attach.52pojie.cn/forum/202511/09/154916b3x4n1y3ct13nnxx.png)

[`12_AgAABTMkRx750yhR2NhLm45jfBOY6P24.png` (18.3 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3NXxmNDA5NTliNXwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

12

2025-11-9 15:49 上传

给`alert`所在行全部打上断点，然后随便输入验证码，点击按钮后并没有断下，咦？咋回事？

![](https://attach.52pojie.cn/forum/202511/09/154918hsfofggfynaryany.gif)

[`13_gif.gif` (461.83 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3NnwyMGFkMTQyMHwxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

13

2025-11-9 15:49 上传

【注】：这是第二次与成功失之交臂，弹窗确实是由`alert`触发的，如果在第一个断点的上一行再下断，就能触发中断，可惜当时没有想到。

难道不是走的`alert`吗？于是我又开始搜索学习了，搜索弹窗的方式有哪些，但是都不对：

```
● confirm()：带 “确定” 和 “取消” 按钮的弹窗。

  不是这个，因为网页的弹窗没有"取消"按钮

● prompt()：带输入框的弹窗，可让用户输入内容，返回输入的字符串（取消则返回null）。

  不是这个，网页只是单纯弹窗，没有让输入什么内容

● 自定义弹窗：实际开发中常使用自定义 HTML+CSS 弹窗，可控制位置（如顶部）、样式和动画。

  难道是这个？这么高级？算了，不去分析是不是自定义弹窗了，麻烦。
```

没有找到弹窗的地方，那就只有去分析`readmore.js`代码的逻辑了。由于加了混淆，所以直接搜索在线的去混淆工具，用了好几个，只有 [obf-io.deobfuscate.io](https://obf-io.deobfuscate.io/) 效果最好，直接解出了`"验证码无效"`等明文：

![](https://attach.52pojie.cn/forum/202511/09/154920mcucycpyyycjuhyp.png)

[`14_AgAABTMkRx7qs35p-n1JOqAIppoc8xkT.png` (405.91 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMjU3N3wwOTAzZDUyM3wxNzkzNTcwMzcwfDE2ODA4NnwyMDcxMjIy&nothumb=yes)

14

2025-11-9 15:49 上传

OK开始分析源码，`"验证码无效"`直接定位到了关键函数，分析如下：
