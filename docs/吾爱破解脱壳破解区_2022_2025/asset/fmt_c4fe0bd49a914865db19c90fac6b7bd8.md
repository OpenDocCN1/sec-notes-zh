
```javascript
_0x3834dc[_0x11c544(0x42c, '\x5b\x32\x33\x28')] = function(        _0x121c70,
        _0x10dcdd,
      ) {
        const _0x74b07a = _0x11c544;
        const _0x58b550 = {
          uKQaL: _0x74b07a(0x289, '\x6e\x74\x43\x4d'),
          LRljw(_0x52e5da, _0x58ed7b) {
            return _0x52e5da === _0x58ed7b;
          },
        };
        function _0x1c18e2(_0x2a7547, _0x2daf7e) {
          const _0x208f76 = _0x74b07a;
          if (_0x2a7547[_0x58b550[_0x208f76(0x36a, '\x4c\x41\x6f\x36')]])
            return _0x2a7547[_0x58b550[_0x208f76(0x36a, '\x4c\x41\x6f\x36')]](
              _0x2daf7e,
            );
          for (
            let _0x282371 = 0x0,
              _0xca62fe = _0x2a7547[_0x208f76(0x3ec, '\x66\x40\x31\x4c')];
            _0x282371
```

还是一段混淆的代码，老规矩，直接处理掉混淆内容

[JavaScript] 纯文本查看 复制代码

```javascript
_0x3834dc = function (_0x121c70) {
    function _0x1c18e2(_0x2a7547, _0x2daf7e) {
        if (_0x2a7547["includes"]) {
            return _0x2a7547["includes"](_0x2daf7e);
        }
        for (let _0x282371 = 0, _0xca62fe = _0x2a7547.length; _0x282371
```

处理完成后我们可以简单测试一下结果是不是符合。

![](https://attach.52pojie.cn/forum/202507/02/080748aknma2o193tlo9nm.png)

image.png (67.69 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY3MHwzMGMxNWY5M3wxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY3MHwzMGMxNWY5M3wxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

数组逻辑5

2025-7-2 08:07 上传

很好，非常符合，我们再用`python`简单实现一下同样的逻辑

[Python] 纯文本查看 复制代码

```python
def canvas_list(input_string):    def list_contains(lst, value):
        return value in lst

    result = []
    for i in range(32):
        if i >= len(input_string):
            break
        char_code = ord(input_string[i])
        while list_contains(result, char_code % 32):
            char_code += 1
        result.append(char_code % 32)
    return result
```

![](https://attach.52pojie.cn/forum/202507/02/080931qdu72bcvsezln64i.png)

image.png (87.59 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY3MXwzM2Q3NDJlZnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY3MXwzM2Q3NDJlZnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

数组逻辑6

2025-7-2 08:09 上传

ok，搞定了，

## 三、总结

轨迹算法和图像还原的内容我就不献丑，纯应付处理代码，论坛里大佬有很多这样的算法，可以去===借鉴借鉴===，整个加密核心就是`rsa`的处理，更多的需要去看文章。

![](https://static.52pojie.cn/static/image/smiley/default/52.gif)
