
```js
// /home/scz/src/js/webpack/hello/browser2node_1/run_webpack_code_1.js
global.self     = global;

require( './hello.bundle_1.js' );
require( './808.chunk.js' );

let __webpack_require__
                = globalThis.__exposedWebpackRequire;
let module_245  = __webpack_require__( 245 );
let module_808  = __webpack_require__( 808 );
let module_54   = __webpack_require__( 54 );

let sth         = "Webpack Test 6912";
console.log( sth );

let ret;
ret             = module_245.func_foo_2( sth );
console.log( ret );
ret             = module_808.func_bar_2( sth );
console.log( ret );
ret             = module_54.func_foo_3( sth );
console.log( ret );
```

`$ node run_webpack_code_1.js`

Webpack Test 6912

1c4a039a5443cdee3fc425220a46a576

1c4a039a5443cdee3fc425220a46a576

f00ac1bc3e8d90718658228e0c5657c24f55537583fee7527a5e10933657ff27

## 3.3) 半自动收集245模块所有依赖模块

假设`245`模块内部依赖很多其他模块，分散在不同chunk文件中，想收集这些被依赖的模块，本小节介绍如何达此目的。

在`245`模块入口、出口分别设断点。待`245`模块入口断点命中后，Console中查看第三形参（本例为`o`），此即`__webpack_require__`函数；点击跳转，即`hello.bundle_1.js`中`"function s(t) {"`所在。

```js
//
// 此即__webpack_require__函数，只是本例中名为s，t即moduleId
//
function s(t) {
    //
    // o[t]实际是__webpack_module_cache__[moduleId]
    //
    var e = o[t];
    if (void 0 !== e)
        return e.exports;
    var r = o[t] = {
        exports: {}
    };
    //
    // 可在此设日志断点
    //
    // 'Need module ' + moduleId
    // 'Need module ' + t
    //
    // window.module_array += moduleId + ':' + __webpack_modules__[moduleId] + ',\n'
    // window.module_array += t + ':' + n[t] + ',\n'
    //
    // n[t]实际是__webpack_modules__[moduleId]
    //
    return n[t].call(r.exports, r, r.exports, s),
    r.exports
}
```

在`__webpack_require()`入口设断点，F8继续，断点命中后立即删除该断点。先在Console中清空`webpack_module_cache__`，如下

```js
__webpack_module_cache__ = {}   // 原理
o = {}                          // 本例
```

这步是为了后面的`__webpack_modules[moduleId].call`（本例对应`n[t].call`）触发更多的依赖。考虑这种情况，`245`模块依赖`633`模块，`633`模块在`245`模块之前因为其他原因已加载过，`webpack_module_cache`中有`633`模块对应项；`633`模块依赖某个假想中的`xxx`模块；现在加载`245`模块，`n[t].call`不会再次触发对`633`、`xxx`模块的加载，很可能导致收集不到`xxx`模块信息。预清空`webpack_module_cache__`，可收集`245`模块更多的依赖。

即使这样，也不能保证一次性收集`245`模块所有依赖。比如加载`245`模块时触发的依赖已收集到位，但调用`245`模块中某函数时触发的依赖在加载`245`模块时收集不到，只能等报错后用同样技术继续补充模块，此时可能出现模块重复；现实世界遭遇过。

假设正断在`__webpack_require__()`入口，立即在Console中定义一个全局变量

```js
window.module_array = ''
```

后面靠它收集结果。

`webpack_require()`中必有`webpack_modules[moduleId].call()`。即使`strip`过，也能找到，只要找唯一的`call()`即可，本例中是`n[t].call()`，此`n[t]`即`__webpack_modules__[moduleId]`。

在`call`所在行设日志断点，比如

```js
window.module_array += moduleId + ':' + __webpack_modules__[moduleId] + ',\n'
window.module_array += '"' + moduleId + '":' + __webpack_modules__[moduleId] + ',\n'
```

本例应为

```js
window.module_array += t + ':' + n[t] + ',\n'
```

日志断点中作为`key`出现的`moduleId`两侧加双引号与否，视当前`case`而定，两种情况都在现实世界遭遇过。

为什么在`call`所在行设日志断点，而非在`__webpack_require()`入口设日志断点？即使前面已经预清空`webpack_module_cache`，在加载`245`模块过程中，仍可能多次命中`webpack_module_cache__`，`cache`的目的就是这个。若在入口设前述日志断点，可能导致`module_array`重复收集模块，体积不必要地增大。

测试观察到意外现象，`dict`的`key`可以重复，不会报错。若真碰上模块重复，可不消重，有洁癖时再说。

F8继续执行，会断在`245`模块出口断点，Console中查看`window.module_array`，即`245`模块所有依赖模块。右键"Copy string contents"，用美化网站查看，比如：

```
https://beautifier.io/
```

理论上可将`window.module_array`的内容全部放入`one_bundle.js`，本例过于简单，看不出效果。此法在超级复杂的现实`webpack`中半自动抠取成功，呈深度优先遍历。

为什么在`245`模块入口、出口同时设断点？因为只想收集`245`模块的依赖模块，出口断点可避免污染`window.module_array`。

前述操作初看较繁琐，熟悉后并不复杂。亦可考虑`Local Overrides`，而非日志断点。

## ☆ 实战讨论

### 1) 识别webpack

特征比较多，看几个现实世界示例，F12调试时注意到

```js
(window.webpackJsonp = window.webpackJsonp || []).push([[242], {
    2368: function(t, e, n) {
        ...
            n(42),
            n(43),
            n(7),
            n(4176),
            n(4177),
            n(34),
            n(54);
```

```js
!function(e) {
    ...
}({
    ...
    "7d92": function(e, t, n) {
```

上面的`2368`、`7d92`就是`moduleId`，后面的三参数函数就是模块函数。模块函数的第三形参，别管它名字是什么，都对应`__webpack_require__`函数。`moduleId`与模块函数呈`"key:value"`形式。

### 2) webpack_require/webpack_module_cache/webpack_modules

看几个现实世界示例

```js
//
// 此即__webpack_require__函数
//
function c(e) {
    //
    // l[e]即__webpack_module_cache__[moduleId]
    //
    if (l[e])
        return l[e].exports;
    var d = l[e] = {
        i: e,
        l: !1,
        exports: {}
    };
    //
    // o[e]即__webpack_modules__[moduleId]
    //
    return o[e].call(d.exports, d, d.exports, c),
    d.l = !0,
    d.exports
}
```

```js
//
// 此即__webpack_require__函数
//
function o(t) {
    //
    // n[t]即__webpack_module_cache__[moduleId]
    //
    if (n[t])
        return n[t].exports;
    var i = n[t] = {
        i: t,
        l: !1,
        exports: {}
    };
    //
    // e[t]即__webpack_modules__[moduleId]
    //
    return e[t].call(i.exports, i, i.exports, o),
    i.l = !0,
    i.exports
}
```

`webpack`半自动抠取时需要找到这三个对象

```
__webpack_require__/__webpack_module_cache__/__webpack_modules__
```

`strip`过的，都是短名字，框架结构不变。

### 3) webpack运行时

`__webpack_require__()`所在`js`含有完整的`webpack`运行时，现实世界中，运行时可能单独位于某个`runtime.js`中，可能位于某个`chunk`文件中。初学者建议将`webpack`运行时完整抠取，减少麻烦，熟悉之后另说。

### 4) webpack抠取结果的集成形式

我习惯于编写`run_webpack_code.js`，框架如下

```js
//
// 补环境
//
global.self         = global;
global.window       = global;
window.navigator    = {
    userAgent: '...',
};
...
//////////////////////////////////////////////////////////////////////////
//
// webpack运行时，含改名过的__webpack_require__()
// 
require( './runtime.js' );
//
// 目标模块所在chunk，已补完所有依赖模块
//
require( './one_bundle.js' );
//////////////////////////////////////////////////////////////////////////
let __webpack_require__
                = globalThis.__exposedWebpackRequire;
let module_xxx  = __webpack_require__( xxx );
//////////////////////////////////////////////////////////////////////////
let param       = ...;
let ret         = module_xxx.func_yyy( param );
console.log( ret );
```

用`require()`加载`chunk`，用`__webpack_require__()`进一步加载`module`。

`runtime.js`、`one_bundle.js`有可能分开，有可能在一个`js`中。无论哪种情况，都需全局导出`__webpack_require__`函数，以便`run_webpack_code.js`调用之。无论哪种情况，都需补全目标模块所有依赖模块。

看几个现实世界`one_bundle.js`示例

```js
(window.webpackJsonp = window.webpackJsonp || []).push([[242], {
    //
    // 2368模块入口，第三形参n即__webpack_require__
    //
    2368: function(t, e, n) {
        ...
    //
    // 2368模块出口
    //
    },
    //
    // For 2368
    //
    // 此部分用半自动抠取技术获取
    //
    120:...,
    ...
    2280:...,
}, [[...]]]);
```

```js
!function(e) {
    ...
    //
    // 此即__webpack_require__函数
    //
    function o(t) {
        if (n[t])
            return n[t].exports;
        var i = n[t] = {
            i: t,
            l: !1,
            exports: {}
        };
        return e[t].call(i.exports, i, i.exports, o),
        i.l = !0,
        i.exports
    }
    //
    // 在__webpack_require__函数体后增加如下代码，全局导出该函数
    //
    globalThis.__exposedWebpackRequire  = o;
    ...
}({
    //
    // 7d92模块入口，第三形参n即__webpack_require__
    //
    "7d92": function(e, t, n) {
        ...
    //
    // 7d92模块出口
    //
    },
    //
    // For 7d92
    //
    // 此部分用半自动抠取技术获取
    //
    "b639":...,
    ...
    "21bf":...,
    //
    // 6c27模块入口，第三形参即__webpack_require__
    //
    "6c27": function(module, exports, __webpack_require__) {
        ...
    //
    // 6c27模块出口
    //
    },
    //
    // For 6c27
    //
    // 此部分用半自动抠取技术获取
    //
    "f28c":...,
    "3c35":...,
});
```

### 5) 补环境

逆向工程一般关注算法函数，其不涉及DOM操作。用半自动抠取技术后，一般补环境不会很复杂，主要是`self`、`window`、`navigator`、`document`这几个。没补全时，执行`run_webpack_code.js`会报错，根据错误提示再补。

若真碰上补`document`，可能比较复杂，现实世界让AI补过一次

```js
global.document     = {
    createEvent: function( type ) {
        return {
            timeStamp: Number( process.hrtime.bigint() / 1_000_000n ),
        };
    },
};
```

### 6) 导出module_xxx.func_yyy

若`module_xxx.func_yyy`已静态导出，这种易认别，模块函数中会有`module.exports`之类的代码。

现实世界碰上过动态导出

```js
(window.webpackJsonp = window.webpackJsonp || []).push([[242], {
    2368: function(t, e, n) {
        ...
        (function(t) {
            /*
             * 将来有个module_2368.a，对应h()，相当于导出h()
             */
            n.d(e, "a", (function() {
                return h
            }
            ));
```
