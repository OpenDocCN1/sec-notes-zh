
[href="forum.php?mod=attachment&aid=MjgwNTA3N3wyOGM1NThmNXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3N3wyOGM1NThmNXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

## 就是这个时间戳导致的

校验这个时间戳发现超时了。

修改为取当前时间即可：

```
this["st"] = Math.floor(Date.now() / 1000);
```

重新再来一遍，终于可以断点到 `Promise` 了。

![](https://attach.52pojie.cn/forum/202509/24/200407rzev8jj8n9nzqhay.png)

`29.png` (14.92 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA3OHw3NjZhOTcwN3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3OHw3NjZhOTcwN3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

`Promise` 中 `return` 了一个 `interrogate` 执行函数。

传入包含 `aih` 的一些参数和两个函数，跟着往下走。

![](https://attach.52pojie.cn/forum/202509/24/200411j3cyh4lvcvfexzay.png)

`30.png` (30.64 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA4MHw5MjhkYzc5M3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4MHw5MjhkYzc5M3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

眼熟吗，这不就是刚才改时间戳位置的 `st` 吗？

再往下走，就进入了最主要的加密逻辑方法里了。

![](https://attach.52pojie.cn/forum/202509/24/200413jf98dbkx6x92424k.png)

`31.png` (29.16 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwNTA4MXw4MTk1NzlmNnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4MXw4MTk1NzlmNnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

创建一个隐藏的 `iframe` 元素，并为其绑定一个 `load` 事件监听器。

当 `iframe` 加载完成后，会执行这个回调函数。

分析函数结构。

![](https://attach.52pojie.cn/forum/202509/24/200415bbm5a9jway1f911y.png)

`32.png` (32.58 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA4MnxhYTk5Nzk2NXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4MnxhYTk5Nzk2NXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

向 `re` 数组里 `push` 了 22 个函数（不同站点可能不一样）。

最后执行 `QN` 方法去遍历执行数组里的每个方法。

在 `push` 的最后一个方法里能看到 `p` 值生成。

![](https://attach.52pojie.cn/forum/202509/24/200418rgir729f9h99qik8.png)

`33.png` (17.5 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA4M3w1NDA2MjY5ZXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4M3w1NDA2MjY5ZXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

再往上可以看到 `p` 值是由 `nt` 这个数组加密而成的。

![](https://attach.52pojie.cn/forum/202509/24/200420rza35r23tbz4me6w.png)

`34.png` (45 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA4NHwwOWQ1YThkYnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4NHwwOWQ1YThkYnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

而 `nt` 里面所有的参数都是那 22 个函数生成存进来的。

## 补环境

补环境有两种方式。

第一种：只拿第一部分代码来补，因为核心加密方法都在第一部分。只需要如下构造 `interrogate` 方法和包含 `aih` 的参数即可。注意！这是一个异步函数！

![](https://attach.52pojie.cn/forum/202509/24/200422vrnf3a0ii9n077nl.png)

`35.png` (69.62 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwNTA4NXxkZjBmNzBiZnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4NXxkZjBmNzBiZnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

第二种：一二三部分代码全要。不需要构造 `interrogate` 方法，只要成功 `hook` 到 `fetch` 请求做处理即可。

![](https://attach.52pojie.cn/forum/202509/24/200424wglnbbll9afuk1uf.png)

`37.png` (43.84 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA4Nnw3ODkwMDQ3YnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4Nnw3ODkwMDQ3YnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

两种方式要补的环境差不多，第二种方式稍微多一点，主要还是那 22 个函数里的环境。

下面列举一下这 22 个函数都大概校验了哪些环境：

- 函数一：通过 `document.addEventListener` 添加很多的监听事件。
- 函数二：通过 `window.OfflineAudioContext` 的 Web Audio API 离线音频处理方法，渲染计算一些值。
- 函数三：通过 `document.addEventListener` 添加很多的监听事件，通过 `document.__selenium_evaluate` 检测一些自动化环境。
- 函数四：随机数组生成字符串。
- 函数五：检验 `navigator`、`screen`、`window` 窗口的一些参数，生成四个 `canvas` 链接。
- 函数六：拿上一步的一个 `canvas` 链接做处理。
- 函数七：检测 `window.WebAssembly`，计算出一些值。
- 函数八：再拿函数五生成的一个 `canvas` 链接做处理。
- 函数九：再拿函数五生成的一个 `canvas` 链接做处理。
- 函数十：创建一个 `canvas`，获取 `WebGL` 上下文。
- 函数十一：拿上一步的 `WebGL` 生成一些值，检测了大量的 `WebGL` 扩展列表属性值和着色器数值，还生成了两个 `canvas` 链接。
- 函数十二：拿函数十一生成的一个 `canvas` 链接生成 hash 值。
- 函数十三：随机数组生成字符串。
- 函数十四：再拿一个 `canvas` 链接处理。
- 函数十五：检测了 `window.WebGLRenderingContext`、`navigator` 的一些属性，`document.createEvent` 主动报错，`window.ontouchstart`、`document.createElement('video')`、`document.createElement('audio')`、`window.chrome`、`history.length`，检测了一些自动化环境，`window.PERSISTENT`、`window.TEMPORARY`，检测了 `PerformanceObserver`，创建一个 `canvas` 并获取其 2D 渲染上下文，以及 `document.documentElement.children`、`document.head.children`、`document.body.children` 和 `document` 中的一些函数。
- 函数十六：随机数组生成字符串。
- 函数十七：向 `nt` 数组 `push` 一个 `value` 为 `true` 的参数。
- 函数十八：检测 `WebAssembly`、`winodw.BigInt`。
- 函数十九：`document.createElement`、`navigator` 的一些属性、`new window.Audio()`。
- 函数二十：检测了一些原型链，如 `Function.prototype.toString`、`Function.prototype.call`、`Function.prototype.apply`、`Function.prototype.bind`，还有一些 `window` 和 `navigator` 等环境。
- 函数二十一：检测了 `window.performance.now()`、`window.Object.getPrototypeOf()` 校验一些属性。
- 函数二十二：没啥校验了，生成 `p` 值。

二十二个函数的大概环境说完，再说一些注意点。

`canvas` 相关：创建了很多次的链接需要注意顺序。

![](https://attach.52pojie.cn/forum/202509/24/200426n7xqlahqrlltqrhr.png)

`38.png` (37.2 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwNTA4N3wwZGE3ODRhNHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4N3wwZGE3ODRhNHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

`window.OfflineAudioContext`：调用它的方法模拟生成一些数值。

![](https://attach.52pojie.cn/forum/202509/24/200429fpxii0w00xii3f3j.png)

`39.png` (14.84 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA4OHxiMTk4Mzg2ZHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA4OHxiMTk4Mzg2ZHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

`document` 补的东西最多，主要是 `canvas` 和 `WebGL` 相关。

![](https://attach.52pojie.cn/forum/202509/24/200433iwwrizeiw9w9awwr.png)

`40.png` (40.35 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwNTA5MHw0YWMxNzgzM3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA5MHw0YWMxNzgzM3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

其它都还简单，最后注意 `toString` 保护函数就行。

## 结果验证

补了差不多 1 千行代码。最后将环境单独打包成一个文件，每次和请求的 JS 代码合并一下执行就行。

因为 JS 是异步执行的，所以我是用 `python` 执行 JS 文件后将生成的参数再写入本地文件读取。纯算法的话最近比较忙，等我抽空看有时间再写吧。

动态链接的验证：

![](https://attach.52pojie.cn/forum/202509/24/200435znc52j6zc35pp8tn.png)

`41.png` (67.13 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwNTA5MXw2MWJiOTlmY3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA5MXw2MWJiOTlmY3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传

固定连接的验证：

![](https://attach.52pojie.cn/forum/202509/24/200437skzzjoe9d1epye4d.png)

`42.png` (49.55 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwNTA5MnxiZTNhNzJlMHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA5MnxiZTNhNzJlMHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)

2025-9-24 20:04 上传
