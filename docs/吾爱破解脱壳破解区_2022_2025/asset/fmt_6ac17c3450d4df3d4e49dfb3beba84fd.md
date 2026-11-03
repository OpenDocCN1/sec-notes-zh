
# [Web逆向] 某贝漫画逆向分析

- 原帖：`https://www.52pojie.cn/forum.php?mod=viewthread&tid=2023585`
- 抓取时间：2026-11-01T22:10:49

## 某贝漫画逆向分析

目标网址：

反调试讲解：`'aHR0cHM6Ly93d3cubWFuZ2Fjb3B5LmNvbS8='`

请求章节响应解密讲解：上面那个网址随便点进去一个漫画的详情页（这里你可以尝试我上面讲解的过反调试的手段哦）

本文仅供学习交流，因使用本文内容而产生的任何风险及后果，作者不承担任何责任，一起学习吧

学学就好，别攻击别人！！！！！！！！！

### 文章摘要 and 作者低语

- 如何分析反调试手段
- 常规逆向的基本流程和手段
- 算法还原

### 反调试讲解

进入当前页面，我们会发现，我们的调试工具被断住了，如下图1所示：

![](https://attach.52pojie.cn/forum/202504/12/132955dytb6dmfd6edfd6b.png)

`1.png` (128.87 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQwOHw1ZmRhMWIyZXwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:29 上传

一旦我们放开断点，当前页面就会变成空白页面：

![](https://attach.52pojie.cn/forum/202504/12/132959na2bz8qwaayooz36.png)

`2.png` (21.49 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQwOXxkZmU1ZDIyNHwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:29 上传

通过图一我们看到，当前堆栈处于`loop`，那我们不妨把目光看向这个函数。（在当前空白页回退上一页就可以找到了）如下图3：

![](https://attach.52pojie.cn/forum/202504/12/133003r8trwr5zet330u3t.png)

`3.png` (129.35 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxMHxjYTU3YWMwNnwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

这里你可以会疑问，欸？utf8(作者)为什么你图一是堆栈的源代码是空白的，而图三不是呢。

如果你注意到了这点，那么恭喜你你有一个善于观察的眼睛。这里是因为我们打开开发者工具的时机导致的，我们一开始是先打开网址后打开开发者工具，这导致当前网址的部分文件我们没有缓存，也就是没有被开发者工具捕获。我们在空白页回退，让这个页面资源重新加载，开发者工具又重新捕获到了资源文件，所有就有了，这是一个常见的坑

言归正传，让我们来逐行详细分析这个函数：

```
// 立即执行函数，创建一个私有作用域防止变量泄漏到全局
((function () {
    // 存放检测到调试行为后要调用的回调函数
    var callbacks = [],

        // 设置调试中断允许的最大时间，单位为毫秒
        timeLimit = 50,

        // 标记是否已经触发过一次调试检测
        open = false;

    // 每 1 毫秒执行一次 loop 函数，用于反调试检测
    setInterval(loop, 1);

    // 返回一个监听器注册接口
    return {
        // 提供注册回调的功能，外部可以添加要在调试时执行的函数
        addListener: function (fn) {
            callbacks.push(fn);
        },

        // 提供移除回调的功能
        cancleListenr: function (fn) {
            callbacks = callbacks.filter(function (v) {
                return v !== fn;
            });
        }
    };

    // 反调试核心检测函数
    function loop() {
        // 获取当前时间戳
        var startTime = new Date();

        // 如果调试工具开启，debugger 会中断此处执行
        debugger;

        // 判断执行完 debugger 后消耗的时间是否超过限制
        if (new Date() - startTime > timeLimit) {

            // 如果是第一次检测到调试器
            if (!open) {
                // 执行所有注册的反调试回调函数
                callbacks.forEach(function (fn) {
                    fn.call(null);
                });
            }

            // 设置标志为 true，防止回调多次调用
            open = true;

            // 通常是跳转页面到 about:blank，使页面内容彻底消失
            aboutBlank();

        } else {
            // 如果没有检测到调试器，重置标志
            open = false;
        }
    }

})())
// 注册一个监听器函数：在检测到调试器时执行 aboutBlank()
.addListener(function () {
    aboutBlank(); // 页面跳转到 about:blank
});
```

知己知彼，百战不殆。`loop`是干扰我们调试的关键函数，`setInterval`又是反复监控。我们首先想到的方法就是直接替换文件，我们将`loop`函数和`setInterval`函数给注释掉不就行了，我们尝试一下，发现该方法完全可行。

到这里我们就通过了反调试（其实这个通过手段有特别多，你还可以直接`hook`函数，关键点在于你对反调试代码的理解，我上面对这个代码分析的很详细，相信你可以也可以想出其他的方法）。

### 3. 请求章节响应解密讲解

目标接口：`comicdetail`

定位接口之后，简单查看，如下图：

![](https://attach.52pojie.cn/forum/202504/12/133005eriisq8gi1yurrru.png)

`4.png` (45.4 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxMXw3NTI4YzYzYnwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

直接查看请求调用堆栈，如下图：

![](https://attach.52pojie.cn/forum/202504/12/133008lhf8fzsugzg046hz.png)

`5.png` (76.66 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxMnxlZjgwZTA4YnwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

看到了我们的老朋友，`eval`，这个是逆向加密常用的手段。小萌新可以先记一下小本本。

为了从头展示逆向过程，我们直接进入第一个堆栈，打上断点（`xhr`断点同理），如下图：

![](https://attach.52pojie.cn/forum/202504/12/133012v30duuus6udaua4u.png)

`6.png` (80.53 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxM3xkZWI0ODcwYnwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

欸，这代码不是人看的，那么恭喜你找到加解密的入口了（代码越抽象，越是在掩饰什么），这一看就是个`ob`混淆，给大家讲一下这种混淆的特点：

- 基于数组映射字符串表的字符串混淆（Array mapping obfuscation）
- 字符串集中存储：所有字符串（变量名、类名、接口路径、HTML代码等）统一存储在一个数组中（如 `var _0xabc = [...]`），避免明文出现。
- 索引访问替代明文：所有原始字符串通过一个映射函数（如 `_0xfunc(index)`）来访问，实现对原始代码的“抽象化引用”。
- 配合数组洗牌：通过`shift()`/`push()`组合“旋转”数组位置，使索引顺序与原始定义不一致，进一步提升混淆性。

如果你会`ast`那么你可以先还原这个代码，后续调试就会简单方便很多。我这里不展示`ast`方法直接分析（绝对不是我不会）。

### 4. 详细逆向过程

逆向，首先我们要找准我们的切入点是什么？--> 这个是核心思路，这个也是后续你快速分析，快速定位的方法

本次案例我们不讲快速定位，老老实实，采用笨方法，一步步跟，主要是掌握跟栈，技巧是你基本功熟练之后自我总结的，我现在直接告诉你快速定位你是不利于你学习的。关注核心在于入参口

最初断点，断下信息如下：

![](https://attach.52pojie.cn/forum/202504/12/133026guppw5oloo5fp4oy.png)

`7.png` (530.13 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxNHw2YmVhYTRkY3wxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

要记住一点，响应体解密必然是在请求发出之后才开始（很合理）。所以，在这个断点下，我们老老实实一步步跟（`F9`）。欸，突然我们发现一个关键点！如下：

![](https://attach.52pojie.cn/forum/202504/12/133028m74gndt0dkzux7go.png)

`8.png` (52.19 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxNXxmZGU1MzQzOHwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

这个`key`，有可能是是解密的关键一步，也可能不是，这个时候你可以选择保存一下这个常量，万一后面遇到了呢，复制一下（你可以把调试过程中所有你认为的关键参数记下来），接着老老实实的跟（`F9`）。

功夫不负有心人啊，终于我们又进入这个混淆文件：

![](https://attach.52pojie.cn/forum/202504/12/133032c0077gyjf7jp8syp.png)

`9.png` (310.23 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxNnw0NjVkOWZmZXwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

但是还是没有看到入参，不急，接着（`F9`），欸，好像看到了希望

![](https://attach.52pojie.cn/forum/202504/12/133037tzy8y5p66yn5eep0.png)

`10.png` (322.25 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxN3w2NWJhMzVmZHwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

在这里打上断点记录一下，我们接着`F9`，看到入参进入函数：

![](https://attach.52pojie.cn/forum/202504/12/133040r7e606879x87bnz6.png)

`11.png` (168.31 KB, 下载次数：0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MDQxOHw1NmY4YWFkNXwxNzkzNTcxMDQ3fDE2ODA4NnwyMDIzNTg1&nothumb=yes)

2025-4-12 13:30 上传

跟进去，我们的响应参数进入到这个函数，代码如下：

```

function _0x468bc8(_0x5d7669) {

                        0xc8 !== _0x5d7669['code'] ? (_0x48fc84['innerText'] = '加載失敗，點擊重新加載',

                        '加載失敗，點擊重新加載' === _0x48fc84[_0x4e8a('0x54')] ? _0x48fc84['onclick'] = function() {

                            _0x45ef84()

                        }

                        : _0x48fc84[_0x4e8a('0x2')] = function() {

                            return null

                        }

                        ) : function(_0x55a4d4) {

                            var _0x2e033c = _0x522e0c(0x6)

                              , _0x5e1413 = _0x55a4d4
```
