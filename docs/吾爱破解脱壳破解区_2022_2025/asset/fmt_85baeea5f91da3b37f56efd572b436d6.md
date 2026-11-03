
```javascript
(function () {
    var _0x6afdb7 = function () {
        var _0x35ea72;
        try {
            _0x35ea72 = Function('return\x20(function()\x20' + '{}.constructor(\x22return\x20this\x22)(\x20)' + ');')();
        } catch (_0x3fbb7b) {
            _0x35ea72 = window;
        }
        return _0x35ea72;
    };
    var _0x53b64a = _0x6afdb7();
    var _0x100cf5 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
    _0x53b64a['atob'] || (_0x53b64a['atob'] = function (_0x59ebb5) {
        var _0x3c892f = String(_0x59ebb5)['replace'](/=+$/, '');
        var _0x3dc83d = '';
        for (var _0x3f7515 = 0x0, _0x494b6e, _0x1eb0e6, _0x48a044 = 0x0; _0x1eb0e6 = _0x3c892f['charAt'](_0x48a044++); ~_0x1eb0e6 && (_0x494b6e = _0x3f7515 % 0x4 ? _0x494b6e * 0x40 + _0x1eb0e6 : _0x1eb0e6, _0x3f7515++ % 0x4) ? _0x3dc83d += String['fromCharCode'](0xff & _0x494b6e >> (-0x2 * _0x3f7515 & 0x6)) : 0x0) {
            _0x1eb0e6 = _0x100cf5['indexOf'](_0x1eb0e6);
        }
        return _0x3dc83d;
    });
})();
var _0x3a5e7e = function (_0x5f34f2, _0x28e2e7) {
    var _0x5c2c82 = [], _0x3520f5 = 0x0, _0x1308fb, _0x31aa5f = '', _0x18a4ce = '';
    _0x5f34f2 = atob(_0x5f34f2);
    for (var _0x1e08c = 0x0, _0x4ec2ba = _0x5f34f2['length']; _0x1e08c  {
                        // 在return处才知道函数是操作符类型还是函数调用类型
                        if (body.isReturnStatement()) {
                            if (body.get('argument').isBinaryExpression()) {
                                // 操作符还原
                                let operator = body.get('argument.operator').node
                                let left = path.get('arguments.0')
                                let right = path.get('arguments.1')
                                // console.log(path.toString(), '-->', left.toString(), operator, right.toString())
                                path.replaceWith(types.binaryExpression(operator, left.node, right.node))
                            } else if (body.get('argument').isCallExpression()) {
                                // 函数调用还原
                                let origin_args = path.get('arguments')
                                origin_args = Array.from(origin_args)
                                let args
                                if (origin_args.length === 1) {
                                    args = []  // 没有参数
                                } else {
                                    args = origin_args.slice(1).map(arg => arg.node)
                                }
                                let old_path_string = path.toString()
                                path.replaceWith(types.callExpression(origin_args[0].node, args))
                                // console.log(old_path_string, '-->', path.toString())
                            } else if (body.get('argument').isLogicalExpression()) {
                                // 操作符还原
                                let operator = body.get('argument.operator').node
                                let left = path.get('arguments.0')
                                let right = path.get('arguments.1')
                                // console.log(path.toString(), '-->', left.toString(), operator, right.toString())
                                path.replaceWith(types.logicalExpression(operator, left.node, right.node))
                            }
                        }
                    }
                }
            }
        }
    }
})
```

```javascript
// 语法数转JS代码
let { code } = generator(ast, { compact: false });
console.log(code);
// 保存
fs.writeFile('./AST/jsl/output.js', code, (err) => {
});
```

### 3.2. js分析

解混淆后代码如下：
