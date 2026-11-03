
# [Web逆向] 【js逆向】码上爬-题16 补环境，第一次补环境成功记录一下！

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2059718
- 抓取时间: 2026-11-01T22:03:06

## 前言：

前几道都是补到一半，补不下去就去搜的详细教程才完成的。但是这个16题我找了半天硬是没看一个补完整的教程。所以只能自己硬着补完，被各种报错给折磨了一晚上。最终还是补成功了记录一下。

## 直接开始:

首先来到网址：`aHR0cHM6Ly93d3cubWFzaGFuZ3BhLmNvbS9wcm9ibGVtLWRldGFpbC8xNi8=` 
直接看数据接口：
![](https://attach.52pojie.cn/forum/202509/12/092037u223hi1tu2ut17iu.png)
6fc5cb94-aaf5-4c93-8253-1c9c1eb8d87e.png (110.85 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI1NnwwMmUwNTQ3YnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI1NnwwMmUwNTQ3YnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
数据接口data
2025-9-12 09:20 上传
其中接口的负载数据就是一个时间戳和一个加密值`h5`（然后其余位置都没有看见明显加密位置，所以这一个`h5`值就是我们要逆向的点）
![](https://attach.52pojie.cn/forum/202509/12/092124fohuwoq4i9z9n1h8.png)
image.png (67.21 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI1N3xmYmIxYzRmOHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI1N3xmYmIxYzRmOHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
接口负载数据
2025-9-12 09:21 上传
知道是这个数据加密了，我们就直接开始跟栈找加密位置,使用`xhr`或者启动器来找都可以，我这里就直接使用启动器来找了（找的到加密位置的过程很简单，这里就不一一写出了）
最后我们会定位到这么一个函数，在这里时我们单步步过后，加密值就生成，所以就可以判断这个值肯定就是在这里生成的了，我们可以直接步进观察函数内部细节逻辑。
![](https://attach.52pojie.cn/forum/202509/12/092922w55tbbetivz00e56.png)
image.png (26.36 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI2MHw2MDgwMTVlMXwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2MHw2MDgwMTVlMXwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
定位函数
2025-9-12 09:29 上传
步进之后我们就能看到这函数，其实这里就是真正的加密位置的函数了，`_$lR`传入的是一个翻页的页数和一个时间戳信息，然后加密值就通过`btoa(_$lw.h5st + String(_$lw.t))`函数返回给`_$lw`值里。
![](https://attach.52pojie.cn/forum/202509/12/093256snqb51axqe2ht00e.png)
image.png (54.01 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI2MXxmZWRhYjVmZHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2MXxmZWRhYjVmZHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
真加密位置
2025-9-12 09:32 上传
![](https://attach.52pojie.cn/forum/202509/12/093502evjuzh5thtj0vavg.png)
image.png (37.2 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI2Mnw2ZjE5ZTlkNnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2Mnw2ZjE5ZTlkNnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
成功加密的返回
2025-9-12 09:35 上传
这里我们就直接开始把代码全部扣下来，在`node`里去补环境。（上环境代理，那里报了补那里）。
![](https://attach.52pojie.cn/forum/202509/12/094812xehim8ui8ppb6uea.png)
image.png (29.22 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjgwMjI2NHxlMTc4ZmQwMnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2NHxlMTc4ZmQwMnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
第一步步环境
2025-9-12 09:48 上传
![](https://attach.52pojie.cn/forum/202509/12/094855wzvqg3ave73214vp.png)
image.png (43.08 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI2NXxkNmY3OThlNnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2NXxkNmY3OThlNnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
一点点
2025-9-12 09:48 上传
才补了一点点，我们的代码就没有报错了，然后我们就需要去模拟浏览器调用我们执行加密的方法。通过观察也是很不容易的找到了他的调用逻辑，有一个大的`Sign`方法，在这个大的`Sign`里面有一个`_$lk`方法，我们的加密函数就只是在`_$lk.prototype.sign`中，然后调用就是最下面的一个`window.PcSign = new window.Sign()`这个方法，我们可以直接使用这个`window.PcSign.sign({page:2,t:12345648479})`，就可以直接调用我们的加密函数。
![](https://attach.52pojie.cn/forum/202509/12/095033b100qjejjz1kjxxl.png)
image.png (15.47 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI2Nnw4MjZlYmYwOHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2Nnw4MjZlYmYwOHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
调用逻辑
2025-9-12 09:50 上传
![](https://attach.52pojie.cn/forum/202509/12/095314zncu9yaj9c8u3y2k.png)
image.png (130.86 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgwMjI2OXw5Mjk5N2ViYnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI2OXw5Mjk5N2ViYnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
sign中的`_$lk`
2025-9-12 09:53 上传
在调用这个方法后，我们其实最好是不要改变代码的逻辑，所以我们还需要再加密函数里面去定义一个自己的变量去接受我们加密的结果比如：`window.YYYYY = _$lw`

## 调用函数的构造：

```
function get_res(page_,t_){
    window.PcSign = new window.Sign({page:page_,t:t_})
    console.log(window.YYYYY)
    return window.YYYYY
}
get_res(3,123456789)
```

然后运行发现没有值，很正常，因为环境肯定是不正常的，我们还需要通过我们的代理去补环境，这里我主要就写我在里面遇到的第一个问题，就是在补到一半时，他一直报一个错误 `load rac js fail!`
![](https://attach.52pojie.cn/forum/202509/12/100802mf1nqlkl10wnul3j.png)
image.png (20.09 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwMjI3NHwxZjIyOGZjZnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI3NHwxZjIyOGZjZnwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
fail
2025-9-12 10:08 上传
这个就是因为没有去和浏览器的环境做对比，我一直以为是我自己补的环境出问题，我还把各种AI都拷打了一遍死活没有找到正确的解法，后来只能放弃去补其他的环境。（其实后面补着补着才发现浏览器一样也是报的这个`load rac js fail!`错误，所以。。。确实有些无语，在这个地方卡了很久）
![](https://attach.52pojie.cn/forum/202509/12/110947euvcmd5env4tms01.png)
image.png (17.31 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgwMjI5OXwyYTY2NGIzZHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMjI5OXwyYTY2NGIzZHwxNzkzNTcwNTg0fDE2ODA4NnwyMDU5NzE4&nothumb=yes)
浏览器报错fail
2025-9-12 11:09 上传
最后就直接上我的成品补环境的代码了，我也不知道为什么他们都不发出来，哎！估计确实是补的很多吧。其中的`canvas`的代码是直接是前面有道题需要的环境（`canvas`的代码AI生成的，我写不来，代码中的`cookie`，`localStorage`的值在自己的浏览器上`copy`就可以了）

```
window = global

window.setInterval = function setInterval(selector) {

    console.log('setInterval的值:',selector)

}

window.setTimeout = function setTimeout(selector) {

    console.log('setTimeout的值:',selector)

}

// 创建一个Canvas元素模拟类

class Canvas {

  constructor(width, height) {

    this.width = width || 300;

    this.height = height || 150;

    this._context = null;

  }

  getContext(contextType) {

    if (contextType === '2d') {

      if (!this._context) {

        this._context = new CanvasRenderingContext2D(this);

      }

      return this._context;

    }

    return null;

  }

  toDataURL() {

    return 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8z8BQDwAEhQGAhKmMIQAAAABJRU5ErkJggg==';

  }

  // 添加其他Canvas方法

  addEventListener() {}

  removeEventListener() {}

  getBoundingClientRect() {

    return {

      left: 0,

      top: 0,

      right: this.width,

      bottom: this.height,

      width: this.width,

      height: this.height

    };

  }

}

// 创建CanvasRenderingContext2D模拟类

class CanvasRenderingContext2D {

  constructor(canvas) {

    this.canvas = canvas;

    this._fillStyle = '#000000';

    this._strokeStyle = '#000000';

    this._lineWidth = 1;

    this._font = '10px sans-serif';

    this._textAlign = 'start';

    this._textBaseline = 'alphabetic';

    this._globalAlpha = 1.0;

    this._globalCompositeOperation = 'source-over';

    this._lineCap = 'butt';

    this._lineJoin = 'miter';

    this._miterLimit = 10;

    this._lineDashOffset = 0;

    this._shadowBlur = 0;

    this._shadowColor = 'rgba(0, 0, 0, 0)';

    this._shadowOffsetX = 0;

    this._shadowOffsetY = 0;

    // 初始化状态栈

    this._stateStack = [];

  }
```
