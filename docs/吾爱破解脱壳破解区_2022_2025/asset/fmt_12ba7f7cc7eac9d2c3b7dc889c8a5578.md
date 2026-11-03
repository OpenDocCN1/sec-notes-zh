
```javascript
var _0x488717 ={};
_0x488717['mouseData'] = [
    [
        0,
        12,
        0
    ]
];
var _0x13468f = _0x488717['mouseData'];
_0x58b708['hg'] = this['getEncryptContent'](_0x13468f, 'be221ccf');
```

## `th` 参数

他们走的都是同个加密函数，第二个参数照样还是字符串，先分析第一个传参`_0x5eae4a - _0x4dc2d8`

![](https://attach.52pojie.cn/forum/202510/31/195932lhi126a9a7bx16wa.png)

2025-10-31-13-51-28-image.png (21.18 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg4NHxkYjlmYTEwNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg4NHxkYjlmYTEwNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:59 上传

![](https://attach.52pojie.cn/forum/202510/31/195942p2qppwhm0mrwubbs.png)

2025-10-31-13-52-25-image.png (1.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg4NXwyNzAyN2JhN3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg4NXwyNzAyN2JhN3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:59 上传

`_0x5eae4a`是取的结束时间，而`_0x4dc2d8`是开始时间

![](https://attach.52pojie.cn/forum/202510/31/195951u1u4oo3cll22y8on.png)

2025-10-31-13-55-15-image.png (22.59 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg4Nnw5ZTdhYTFmNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg4Nnw5ZTdhYTFmNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:59 上传

![](https://attach.52pojie.cn/forum/202510/31/200010byysixytkfkuajst.png)

2025-10-31-13-55-33-image.png (6.32 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg4N3wyYzM0MmRmZnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg4N3wyYzM0MmRmZnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:00 上传

打印`_0x488717`发现里面的所有值都跟滑块有关，估计可以猜测`_0x5eae4a`是取的滑块松开结束时间，而`_0x4dc2d8`是滑块触摸开始时间

![](https://attach.52pojie.cn/forum/202510/31/200022dn77f3zdn2iz42n7.png)

2025-10-31-13-57-53-image.png (44.17 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg4OHw3NzQ3MDdlN3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg4OHw3NzQ3MDdlN3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:00 上传

到这里，验证码所有会变的参数都已经分析完毕。emmm,都分析到这里了，还是把所有参数都分析一遍看看里面的明文是啥。

## 其他参数

`qt`的第二个传参照样还是字符串，第一个传参是290，也就是渲染的宽。

![](https://attach.52pojie.cn/forum/202510/31/200049angp1yvwyr0n0z8g.png)

2025-10-31-14-02-09-image.png (16.21 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg4OXwyOTgzMGY4OHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg4OXwyOTgzMGY4OHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:00 上传

![](https://attach.52pojie.cn/forum/202510/31/200103tjkefddfbuebjkkj.png)

2025-10-31-14-03-41-image.png (1.42 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5MHw3NjhiOTI5N3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5MHw3NjhiOTI5N3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:01 上传

`lf`的第二个传参照样还是字符串，第一个传参是145，也就是渲染的高。

![](https://attach.52pojie.cn/forum/202510/31/200114wp1rsscpurupakcc.png)

2025-10-31-14-04-27-image.png (21.09 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5MXxlNDMxY2ZhMHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5MXxlNDMxY2ZhMHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:01 上传

![](https://attach.52pojie.cn/forum/202510/31/200207xs867gv5s67y69jr.png)

2025-10-31-14-05-34-image.png (1.44 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5Mnw3OWRkOTg2ZXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5Mnw3OWRkOTg2ZXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:02 上传

继续向下跟栈，就可以找到其他的参数

![](https://attach.52pojie.cn/forum/202510/31/200220vdo5gxpx2x859ixc.png)

2025-10-31-14-09-40-image.png (10.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5M3xjM2E2MTk1MXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5M3xjM2E2MTk1MXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:02 上传

### `fm` 参数

`fm`的明文是从某个对象里面取的，没去深究了，因为是固定的明文。

```javascript
_0x58b708['fm'] = this['getEncryptContent'](_0x1758fe['default']['__userConf']['console'], 'a571b7e5')
```

![](https://attach.52pojie.cn/forum/202510/31/200233qlledyiqsyeyinst.png)

2025-10-31-14-10-36-image.png (1.64 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5NHw0YjA4ZTdiN3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5NHw0YjA4ZTdiN3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:02 上传

### `sl` 参数

`sl`的第一个参数是执行`_0x1758fe['default']['runBotDetection']`的返回值，虽然是固定0的，不过可以去深究一下看看函数。

```javascript
_0x58b708['sl'] = this['getEncryptContent'](_0x1758fe['default']['runBotDetection'](), '146ca9d6')
```

![](https://attach.52pojie.cn/forum/202510/31/200247ot0tvhsvn9z397ux.png)

2025-10-31-14-16-06-image.png (10.66 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5NXwxN2RlN2JkM3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5NXwxN2RlN2JkM3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:02 上传

![](https://attach.52pojie.cn/forum/202510/31/200257tk099595lp3hyh5u.png)

2025-10-31-14-19-39-image.png (34.15 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg5NnxmZDgxOGI0YXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg5NnxmZDgxOGI0YXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 20:02 上传

网站都是混淆代码，扣到本地解混淆之后分析。

这段代码就是综合了多种检测方法检测是否使用了自动化插件，以识别是否有自动化工具在使用浏览器访问网页。如果检测到任何自动化工具的特征，函数将返回1 ，否则返回0 ，所有我们固定0就好了，没试过固定1能不能过滑块，感兴趣的笔者可以自己测试。

```javascript
function Azojs(_0x4ba685, _0x43ec98) {

                return _0x4ba685 != _0x43ec98;

            }

function runBotDetection() {

            //解密函数可以注释掉
            // var _0x3dfceb = _0x2820f1;

            try {

// __webdriver_evaluate: 这是一个用于检测 WebDriver 环境的标识符。在一些情况下，网站会检查这个标识符来判断是否通过 WebDriver 进行访问。
// __selenium_evaluate: 与 WebDriver 相似，这个标识符用于检测 Selenium 环境。
// __webdriver_script_function: 用于表示 WebDriver 脚本功能，可能与执行脚本相关。
// __webdriver_script_func: 可能是 __webdriver_script_function 的一个变体，功能相似。
// __webdriver_script_fn: 与前两个类似，可能是脚本执行的某种标识。
// __fxdriver_evaluate: 特定于 Firefox 的 WebDriver 的一个标识符，用于脚本执行。
// __driver_unwrapped: 指示当前是否有包装的 WebDriver 实例。
// __webdriver_unwrapped: 与 __driver_unwrapped 类似，通常用于检测 WebDriver 实例的状态。
// __driver_evaluate: 用于执行脚本或表达式的一个标识。
// __selenium_unwrapped: 检测当前 Selenium 环境的状态。
// __fxdriver_unwrapped: 与 Firefox WebDriver 相关，表示没有包装的状态。
// _phantom: 通常指 PhantomJS，一个无头浏览器，用于执行网页的脚本。
// __nightmare: Nightmare.js 是一个用于自动化网页的高层次无头浏览器工具，基于 Electron 构建。
// _selenium: 通常指 Selenium 自动化测试框架。
// callPhantom: 可能是 PhantomJS 中用于调用特定功能的函数。
// callSelenium: 用于 Selenium 自动化功能的调用。
// _Selenium_IDE_Recorder: 可能与 Selenium IDE 相关，这是一种用于录制用户在浏览器中的操作并生成测试脚本的工具。

var _0x488f33 = ['__webdriver_evaluate', '__selenium_evaluate', '__webdriver_script_function', '__webdriver_script_func', '__webdriver_script_fn','__fxdriver_evaluate', '__driver_unwrapped', '__webdriver_unwrapped','__driver_evaluate', '__selenium_unwrapped', '__fxdriver_unwrapped']

  , _0x3e3aa6 = ['_phantom', '__nightmare', '_selenium', 'callPhantom', 'callSelenium', '_Selenium_IDE_Recorder'];

// 遍历_0x3e3aa6数组，如果存在对应数据，则返回1，主要作用是检测当前浏览器环境中是否存在一些与自动化测试工具或无头浏览器相关的特征。如果检测到这些特征，代码将返回 1，这通常用于判断当前环境是否为自动化环境。

for (var _0x17026d in _0x3e3aa6) {

    var _0xee867d = _0x3e3aa6[_0x17026d];

    if (window[_0xee867d])

        return 1;

}
```
