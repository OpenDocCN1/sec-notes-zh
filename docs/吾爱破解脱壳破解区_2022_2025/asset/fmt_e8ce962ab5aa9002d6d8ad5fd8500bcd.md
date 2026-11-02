
```javascript
// ext值原始函数
'ext': _0x4b1c93(_0x5054ec(this[_0x487f4d(0x105)][_0x487f4d(0x272)][_0x487f4d(0x7a4)], this[_0x487f4d(0x8d6)] + ',' + this[_0x487f4d(0x4b4)][_0x487f4d(0xab4)]))
```

简化一下就是下面这样：

```javascript
win_0x4b1c93(win_0x1767c8(token, 1 + ',' + pathStr.length))
```

将上面简化之后的结果进行测试。

![](https://img-blog.csdnimg.cn/2ac112af28894b1e875cf94743f5899c.png)

## `cb` 参数加密函数

在 `core.v2.19.1.min_copy.js` 文件下搜索 `referer` 关键字，然后下断点，可以看到这个 `cb` 跟最后发送请求的是一样的。

![](https://img-blog.csdnimg.cn/9b8618ef9179479fafeb4d52d27e07cb.png)

我们向上往回走一步。

![](https://img-blog.csdnimg.cn/6ea81ce482d6443bbee0e6f451bb7925.png)

跟进去，会进到函数 `_0x285d22()`。这个函数会返回 `_0xe25de6(_0x45082c)`。

![](https://img-blog.csdnimg.cn/829c7e7c9978480a9dff93f76a059c71.png)

在这里插入图片描述。

然后再跟进去看下返回的 `_0xe25de6(_0x45082c)`，可以看到它调用了我们之前全局定义的 `win_0x4b1c93`。

![](https://img-blog.csdnimg.cn/d640b130f4f64e918decfac690d84734.png)

这里我们直接修改一下源码，将 `_0x285d22()` 也暴露出来。

![](https://img-blog.csdnimg.cn/f57c6ae2993845cfb79f7d40e92ec0fd.png)

这里的 `_0x2acb72[0x20](https://www.52pojie.cn/0x20)` 要说明一下，进到这个函数里面，它会有一个随机数，所以它是随机的。

![](https://img-blog.csdnimg.cn/264d73943dfd4d42993a8587d4e72fce.png)

![](https://img-blog.csdnimg.cn/00f544accf3c4ee3ae44168a6de1e661.png)

那么 `cb` 参数的调用就是：

```javascript
win_0x285d22()
```

![](https://img-blog.csdnimg.cn/2182a651d21e48b785931b5159d50e86.png)

## `callback` 参数加密函数

最后还剩下一个 `callback` 参数。

因为 `callback` 参数我们从调试开始到现在是一直没有遇到的，我们直接从发送请求的调用堆栈去找找，直接从最后往前找。

![](https://img-blog.csdnimg.cn/fd6ebc728ff043c3ab0a423e17804f37.png)

进到里面之后，我们先下断点，然后滑动完成，在这个位置断下来了，可以看到，会生成一个最后实际请求要发送的 `callback` 参数。

![](https://img-blog.csdnimg.cn/06327489e07a46258a5618c10044d1fa.png)

将鼠标往后面拖动，会发现有一个 `__JSONP` 字符，我们先复制下来。

![](https://img-blog.csdnimg.cn/9b761180f1744bfab2431d1984e8c3c4.png)

将逗号表达式改为正常赋值，然后替换到源码上，方便查看调试。

```javascript
var _0x541624
var _0x21d7a8
var _0x12fd7e = Math[_0x3daaf1(0x9a7)]()[_0x3daaf1(0x327)](0x24)[_0x3daaf1(0x9d6)](0x2, 0x9)
var _0x421a5c = _0x9795d6[_0x3daaf1(0x290)] || '__JSONP'
var _0x2582fe = _0x9795d6[_0x3daaf1(0x8ef)] || _0x421a5c + ('_' + _0x12fd7e) + ('_' + _0x12f742++)
var _0x46c3ff = _0x9795d6[_0x3daaf1(0x74c)] || _0x3daaf1(0x643)
var _0x5babb5 = _0x9795d6[_0x3daaf1(0x2c7)] || 0x1770
var _0x5c6aa5 = window[_0x3daaf1(0x1d9)]
var _0x4b24f9 = document['getElementsByTagName'](_0x3daaf1(0x58f))[0x0] || document[_0x3daaf1(0x2e2)]
```

![](https://img-blog.csdnimg.cn/605c21d23c1545e1b0e536b22d0586a8.png)

通过查看，在这一行生成了最后的 `callback` 参数。

![](https://img-blog.csdnimg.cn/ec0f86dcdf984d2ab9f999e7a5fe2cab.png)

```javascript
// 原始代码

var _0x12fd7e = Math[_0x3daaf1(0x9a7)]()[_0x3daaf1(0x327)](0x24)[_0x3daaf1(0x9d6)](0x2, 0x9)

var _0x421a5c = _0x9795d6[_0x3daaf1(0x290)] || '__JSONP'

var _0x2582fe = _0x9795d6[_0x3daaf1(0x8ef)] || _0x421a5c + ('_' + _0x12fd7e) + ('_' + _0x12f742++)
```

简化后就是这样：

```javascript
var _0x12fd7e = Math.random().toString(0x24).slice(0x2, 0x9)

var _0x421a5c = _0x9795d6['prefix'] || '__JSONP'

// var _0x2582fe = _0x9795d6['name'] || _0x421a5c + ('_' + _0x12fd7e) + ('_' + _0x12f742++)

var _0x2582fe = _0x9795d6['name'] || _0x421a5c + ('_' + _0x12fd7e) + ('_' + 1)
```

参数 `_0x12f742` 是表示次数，你每发送一次，只要没有成功，它就会叠加，所以这里我们直接写成固定值。

直接调用测试。

![](https://img-blog.csdnimg.cn/7bbf173f3bbe486eb9a61b4e55977e22.png)

## 验证测试

到这里，我们所有需要的参数都找到了，下一步就是模拟调用进行验证。我已经录制好了所有出现的路径（一共是 `7+6+5+4+3+2+1 = 28` 条），当然你愿意的话也可以去录制 `7*7` 的路径条数。目前先手动填写需要交换的顺序，自动填写的话，需要参考我之前发的文章 [7060430919081361421](https://juejin.cn/post/7060430919081361421)，将这个项目做成一个服务去调用，识别计算拼图结果。

进到调用堆栈，看它最后的请求。

![](https://img-blog.csdnimg.cn/03c3a684a09a438386f7128eef27c2ce.png)

![](https://img-blog.csdnimg.cn/e2795245f2a84f47ad2a914df39442a6.png)

这段代码修复一下，下面是修复的代码，可以看到，它会创建一个 `javaScript` 标签，然后通过 `javaScript` 标签去请求。

```javascript
let _0x5f84aa = "https://c.dun.163.com/api/v3/check"
_0x5f84aa += (~_0x5f84aa.indexOf('?') ? '&' : '?') + _0x31e743(senObj) + '&' + 'callback' + '=' + callback
_0x5f84aa = _0x5f84aa['replace']('?&', '?')
var _0x4b24f9 = document['getElementsByTagName']('script')[0x0] || document['head'];
_0x5f84aa['replace']('?&', '?')
let _0x541624 = document.createElement('script')
_0x541624['type'] = 'text/javascript'
_0x541624['src'] = _0x5f84aa
_0x4b24f9['parentNode']['insertBefore'](_0x541624, _0x4b24f9)
_0x30c635;
```

这里需要注意一个，发送的 `get` 请求如果你只模拟这一段的话，最后会因为请求链接有空格，它会给你报链接检测错误的信息。

我们直接按照它的处理方式去处理：

```javascript
_0x5f84aa += (~_0x5f84aa.indexOf('?') ? '&' : '?') + _0x31e743(senObj) + '&' + 'callback' + '=' + callback
```

`_0x31e743(senObj)` 这个方法会帮你处理好请求链接的空格问题。这里不再赘述，详细的可以去看具体细节。这里直接贴上完整模拟代码：

```javascript
// 前端模拟提交，这里也可以使用py去提交，我是不想去折腾了，py提交的时候记得带上cookies
window[callback] = function (_0x203a88) {
    console.log('okokokok');
}
function _0x31e743(_0x295923) {
    var _0x50ff1e = []
    for (var _0x5b5df6 in _0x295923)
        _0x295923['hasOwnProperty'](_0x5b5df6) && _0x50ff1e['push'](encodeURIComponent(_0x5b5df6) + '=' + encodeURIComponent(_0x295923[_0x5b5df6]));
    return _0x50ff1e['join']('&');
}
function _0x30c635() {
    window[_0x2582fe]()
}
let _0x5f84aa = "https://c.dun.163.com/api/v3/check"
_0x5f84aa += (~_0x5f84aa.indexOf('?') ? '&' : '?') + _0x31e743(senObj) + '&' + 'callback' + '=' + callback
_0x5f84aa = _0x5f84aa['replace']('?&', '?')
var _0x4b24f9 = document['getElementsByTagName']('script')[0x0] || document['head'];
_0x5f84aa['replace']('?&', '?')
let _0x541624 = document.createElement('script')
_0x541624['type'] = 'text/javascript'
_0x541624['src'] = _0x5f84aa
_0x4b24f9['parentNode']['insertBefore'](_0x541624, _0x4b24f9)
_0x30c635;
```

## 完整代码

最后一步，将我们之前找出来的东西封装成函数来调用，代码如下：

```javascript
function createUrl(obj) {

    // 已经录制好的路径字典

    let movPath = {

        '0,1': [

            {

                "_0x3c8f13": 1,

                "_0x46347e": 0,

                "diffTime": 77

            },

            {

                "_0x3c8f13": 2,

                "_0x46347e": 0,

                "diffTime": 99

            },

            {

                "_0x3c8f13": 3,

                "_0x46347e": 0,

                "diffTime": 113

            },

            {

                "_0x3c8f13": 5,

                "_0x46347e": 0,

                "diffTime": 120

            },

            {

                "_0x3c8f13": 7,

                "_0x46347e": 1,

                "diffTime": 129

            },

            {

                "_0x3c8f13": 10,

                "_0x46347e": 1,

                "diffTime": 137

            },

            {

                "_0x3c8f13": 14,

                "_0x46347e": 1,

                "diffTime": 145

            },

            {

                "_0x3c8f13": 20,

                "_0x46347e": 2,

                "diffTime": 152

            },

        .................此处省略，因为实在太长了，6000多行，具体的可以去看最后的项目地址，里面有完整代码

        ]

    }

    function randomNum(minNum, maxNum) {

        switch (arguments.length) {

            case 1:

                return parseInt(Math.random() * minNum + 1, 10);

                break;

            case 2:

                return parseInt(Math.random() * (maxNum - minNum + 1) + minNum, 10);

                break;

            default:

                return 0;

                break;

        }

    }

    let pathKey = obj.pos[0] + ',' + obj.pos[1]

    let pathList = movPath[pathKey]

    let pathStr = []

    for (let i = 0; i
```

调用测试示例：

此处不作 `py` 测试，不想折腾了，道理其实是一样的。

## 测试方法

第一步：抓包替换文件。

第二步：找到 `token`，这里我们直接手动找一个，在 `get` 中会返回一个 `token`。

![](https://img-blog.csdnimg.cn/5770333466824a3f8715c98f988bd48f.png)

第三步，如下：

![](https://img-blog.csdnimg.cn/32edefa424054c7aa567d4cfc54889f4.png)

如果成功的话，就会返回这样的一个很长的结果。

![](https://img-blog.csdnimg.cn/92655de6be2b493791ef8c4d26f2a3e0.png)

如果失败，就会像下面这样。

![](https://img-blog.csdnimg.cn/eddc70f4c59148699cde3ae8502f4d60.png)

## 项目代码地址

[项目代码地址](https://gitcode.net/weixin_45307278/wyyd)

## 演示
