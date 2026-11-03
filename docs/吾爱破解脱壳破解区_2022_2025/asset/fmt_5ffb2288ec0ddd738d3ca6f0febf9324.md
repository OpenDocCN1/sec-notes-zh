
# 浏览器特征收集代码

```javascript
// body 不存在时，入栈字符串 "undefined"
_0x6f820c['push']("undefined");
}
// 检测是否支持 WebSQL（window.openDatabase），用 _0x2ba902 处理并放入数组
_0x6f820c["push"](_0x2ba902(window["openDatabase"]));
// 读取浏览器的 CPU 类型（IE 特有属性 cpuClass），放入数组
_0x6f820c["push"](_0x301a50["cpuClass"]);
// 读取浏览器平台信息（如 Win32、MacIntel 等），放入数组
_0x6f820c['push'](_0x301a50["platform"]);
// 定义变量，稍后用来存放 canvas 支持状态
var _0x49c4ff;
// 判断标志 _0xdd8e4c['l'] 是否为真，表示是否进行 canvas 特征检测
if (_0x49c4ff = _0xdd8e4c['l'])
    try {
        // 创建一个 canvas 元素
        var _0x3f3574 = _0x27711e["createElement"]("canvas");
        // 检测 canvas 是否支持 getContext 且支持 2d 渲染上下文
        _0x49c4ff = !(!_0x3f3574["getContext"] || !_0x3f3574["getContext"]("2d"));
    } catch (_0x47141f) {
        // 如果创建或检测过程中抛出异常，则表示不支持
        _0x49c4ff = !1;
    }
// 如果 canvas 支持
if (_0x49c4ff)
    try {
        // 调用函数 _0x2688e1() 获取 canvas 指纹数据，放入数组
        _0x6f820c["push"](_0x2688e1());
        // 如果标志 _0xdd8e4c['b'] 为真，调用 _0x52ab0f() 获取更多 canvas 相关数据并入栈
        _0xdd8e4c['b'] && _0x6f820c["push"](_0x52ab0f());
    } catch (_0x3b58f7) {
        // 如果获取 canvas 指纹异常，标记为 "canvas exception"
        _0x6f820c["push"]("canvas exception");
    }
// 调用函数 _0x19c64d()，可能是收集字体、插件等信息，加入第一个数组
_0x6f820c["push"](_0x19c64d());
// 如果标志 _0xdd8e4c['a'] 为真，调用 _0x50a0aa() 收集额外特征，放入第二个数组
_0xdd8e4c['a'] && _0x1cb4a4['push'](_0x50a0aa());
// 把浏览器 userAgent 字符串加入第二个数组
_0x1cb4a4['push'](_0x301a50["userAgent"]);
// 把浏览器语言设置加入第二个数组
_0x1cb4a4["push"](_0x301a50["language"]);
// 把屏幕颜色深度加入第二个数组
_0x1cb4a4["push"](window["screen"]["colorDepth"]);
// 如果标志 _0xdd8e4c['o'] 为真，收集屏幕分辨率（高度 x 宽度）
if (_0xdd8e4c['o']) {
    // 如果 window.screen 存在，取高宽组成数组，否则默认[0,0]
    _0x49c4ff = window["screen"] ? [window["screen"]["height"], window["screen"]["width"]] : [0, 0];
    // 判断 _0x49c4ff 是否未定义，并通过 _0x2ba902 进行处理，结果不是 "undefined" 即加入第二个数组
    if (('undefined' == typeof _0x49c4ff ? "undefined" : _0x2ba902(_0x49c4ff)) !== "undefined")
        _0x1cb4a4["push"](_0x49c4ff['join']("x"));  // 格式化成 "heightxwidth" 字符串入栈
}
// 加入当前时区偏移（分钟），例如中国是 -480
_0x1cb4a4['push'](new Date()["getTimezoneOffset"]());
// 加入 doNotTrack 设置，表示用户是否启用了“请勿跟踪”
_0x1cb4a4['push'](_0x301a50["doNotTrack"]);
// 调用 _0x43c5b9() 收集其他环境信息，加入第二个数组
_0x1cb4a4["push"](_0x43c5b9());
}
// 最终准备输出的数组，重用变量名
_0x49c4ff = [];
// 根据标志 _0xdd8e4c['g'] 是否存在，决定用哪个函数对收集的特征进行哈希或编码处理
if (_0xdd8e4c['g']) {
    // 使用 _0xdd8e4c['g'] 函数对第一个数组拼接字符串进行处理，再加入结果数组
    _0x49c4ff['push'](_0xdd8e4c['g'](_0x6f820c["join"]("###")));
    // 使用 _0xdd8e4c['g'] 函数对第二个数组拼接字符串进行处理，再加入结果数组
    _0x49c4ff['push'](_0xdd8e4c['g'](_0x1cb4a4["join"]("###")));
} else {
    // 如果没有指定处理函数，使用默认的 _0x354f80 函数对拼接字符串进行处理，加入数组
    _0x49c4ff['push'](_0x354f80(_0x6f820c['join']("###")));
    _0x49c4ff["push"](_0x354f80(_0x1cb4a4['join']("###")));
}
// 返回包含两个处理结果的数组
return _0x49c4ff;
}
```

## canvas绘图结果

![](https://attach.52pojie.cn/forum/202511/15/201246yd5szdbmodsddd1e.png)

2025-11-15-17-48-20-真实.png (12.9 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgxNDYwM3wyOTRkOGU0YXwxNzkzNTcwMzU5fDE2ODA4NnwyMDczMTI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNDYwM3wyOTRkOGU0YXwxNzkzNTcwMzU5fDE2ODA4NnwyMDczMTI0&nothumb=yes)

2025-11-15 20:12 上传

## 2> 结尾

每天进步一点点
