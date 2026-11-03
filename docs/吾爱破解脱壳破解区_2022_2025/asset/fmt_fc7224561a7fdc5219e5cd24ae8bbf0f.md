
# [Web逆向] 某蜂窝 `w_tsfp` 参数分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1916130
- 抓取时间：2026-11-01T22:32:56

## 声明

本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关。

本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责，若有侵权，请联系作者立即删除。

## 目标地址

```
aHR0cHM6Ly93d3cubWFmZW5nd28uY24v
```

## 参数来源分析

新建新的无痕窗口，打开 `devtools` 并浏览目标地址，浏览器停在了一处动态加载的 `debugger` 代码中，该段 `js` 代码可以分析出由 `Function.constructor` 动态构造得到，所以直接在 `console` 中 hook 掉 `debugger` 的构造函数，使其失效：

![](https://attach.52pojie.cn/forum/202404/21/182722p8dd3dd8d2kwduda.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（65.69 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDkxNnwyMjNhZTk5M3wxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-21 18:27 上传

![](https://attach.52pojie.cn/forum/202404/21/182751jlz4jzrijxybhqly.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（123.64 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDkxN3xmMjZjOTJlMHwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-21 18:27 上传

```
Function.prototype.original_constructor= Function.prototype.constructor;

Function.prototype.constructor=function(){

    if (arguments && typeof arguments[0]==="string"){

        if (arguments[0]==="debugger")

            return;

    }

        return Function.prototype.original_constructor.apply(this, arguments);

};
```

![](https://attach.52pojie.cn/forum/202404/22/101040cejtc0hen3ncv1t1.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（63.26 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE2NXwyYjVkODljZHwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:10 上传

点击继续：

![](https://attach.52pojie.cn/forum/202404/22/101114l8ithkht0i8fpeia.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（80.55 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE2NnwzYzg2YzViNnwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:11 上传

![](https://attach.52pojie.cn/forum/202404/22/101151k26ccis2c5t03csx.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（198.9 KB，下载次数：4）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE3MXw2ODQ4MjIwZHwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:11 上传

查看网络请求可以知道，请求了两次目标地址，其中第一次请求只返回了一个空的 `x-waf-captcha-referer` 参数

![](https://attach.52pojie.cn/forum/202404/22/101231mzjwue450owouwe0.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（52.78 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE3NXw4YmMyMmIwZXwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:12 上传

两次目标地址请求间加载了一个 `probe.js` 文件，之后发起第二次请求，此时携带了一个 `w_tsfp` 参数并且成功返回网页内容。

![](https://attach.52pojie.cn/forum/202404/22/101301h64l9l4ok458zozm.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（245.52 KB，下载次数：1）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE3N3wxZmYxMDg4NnwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:13 上传

![](https://attach.52pojie.cn/forum/202404/22/101400o39irvfr3y43h009.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（185.42 KB，下载次数：4）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4MXw5NDk0ZTljMXwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:14 上传

由此可以基本断定 `w_tsfp` 参数在 `probe.js` 中生成。

## 解混淆

打开 `probe.js`，在 `js` 文件加载入口函数处打上断点，然后对其进行分析。

![](https://attach.52pojie.cn/forum/202404/22/101431tuohu36m6ltoax3h.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（100.89 KB，下载次数：2）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4M3w4NjJkMzZiMXwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:14 上传

可以知道该 `js` 文件进行了一些常规混淆，例如大量使用了字符串函数（指调用了某个函数返回特定字符串的函数，该函数目的是为了隐藏字符串），控制流平坦化等。

![](https://attach.52pojie.cn/forum/202404/22/101457mf9f9d9vdj2wmm9f.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（173.1 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4NHxlYjFlYmMxNXwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:14 上传

![](https://attach.52pojie.cn/forum/202404/22/101515b7uzuibbztf9b3zu.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（165.83 KB，下载次数：1）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4NXxlMTliZjY5MHwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:15 上传

### 1.1 还原字符串函数

通过分析可以发现，`probe.js` 任意一个字符串函数都源自 `a1i` 根字符串函数并且参数都是透传的，没有加上任何偏移：

![](https://attach.52pojie.cn/forum/202404/22/101542lfjvyzjimvff90vg.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（74.2 KB，下载次数：2）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4Nnw0MWUyZDEzN3wxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:15 上传

![](https://attach.52pojie.cn/forum/202404/22/101618tdmif3qqdwchiywl.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（54.85 KB，下载次数：2）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4N3xiZDU5ZTI1Y3wxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:16 上传

![](https://attach.52pojie.cn/forum/202404/22/101643pmzx0rc0sqxc9km2.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（144.21 KB，下载次数：4）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4OHw0NTBhMjlkZnwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:16 上传

因此可以先使用 `babel` 遍历所有字符串函数，得到所有参数变化列表，之后在 `console` 中计算得到所有字符串值；根据这些字符串值再将原 `js` 中的字符串函数调用替换为对应字符串。

首先需要找到字符串函数调用的特征，通过观察结构可以知道任何一个字符串函数调用都带有两个参数，其中第一个参数是一个十六进制数字，第二个参数是一个字符串，并且函数名长度很短，总是 2（除了 `a1i` 函数）。

![](https://attach.52pojie.cn/forum/202404/22/101710gdmdpzizp58mf8hj.png)
`1712048977504-3d661aa5-60de-4ffc-be9e-70641b238ff7.png`（212.37 KB，下载次数：3）
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTE4OXw2YTQwNzNiMXwxNzkzNTcyMzczfDE2ODA4NnwxOTE2MTMw&nothumb=yes)｜2024-4-22 10:17 上传

因此可以根据这些特征找到所有字符串函数调用：
