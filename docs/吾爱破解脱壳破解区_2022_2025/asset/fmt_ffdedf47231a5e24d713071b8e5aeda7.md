
## AST 还原处理

### 操作符与函数调用还原

```javascript
let obj_property = node_path.get('key.value').node

if (
    obj_property === property &&
    node_path.get('value').isFunctionExpression()
) {
    let func_bodys = node_path.get('value.body.body')
    func_bodys = Array.from(func_bodys)
    func_bodys.forEach(body => {
        // 在return处才知道函数是操作符类型还是函数调用类型
        if (body.isReturnStatement()) {
            if (body.get('argument').isBinaryExpression()) {
                // 操作符还原
                let operator = body.get('argument.operator').node
                let left = path.get('arguments.0')
                let right = path.get('arguments.1')
                console.log(path.toString(), '-->', left.toString(), operator, right.toString())
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
                console.log(old_path_string, '-->', path.toString())
            } else if (body.get('argument').isLogicalExpression()) {
                // 操作符还原
                let operator = body.get('argument.operator').node
                let left = path.get('arguments.0')
                let right = path.get('arguments.1')
                console.log(path.toString(), '-->', left.toString(), operator, right.toString())
                path.replaceWith(types.logicalExpression(operator, left.node, right.node))
            }
        }
    })
})

}

}

}

})
```

### 控制流还原

然后，我们对这样的控制流进行还原。

![](https://attach.52pojie.cn/forum/202412/07/033255ulzqf7izfdqr7ifo.png)

`9.png` (68 KB, 下载次数: 3)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjA2N3w3NzlhMDA1OHwxNzkzNTcxNTE1fDE2ODA4NnwxOTg5MDUw&nothumb=yes)

2024-12-7 03:32 上传

思路：拿到控制器和 `case` 节点，然后根据控制器的顺序对 `case` 节点进行排序。

#### AST 代码

```javascript
let controler_code = {}
let controler = {}
traverse(ast, {
    WhileStatement: {
        exit: function (path) {
            if (
                path.get('test').isUnaryExpression() || (path.get('test').isArrayExpression() && path.get('test').toString() === '[]')
            ) {
                if (path.get('body.body').length === 0) return  // while循环体为空，直接返回
                if (path.get('body.body.0').isTryStatement()) return
                console.log(path.toString())
                let switch_condition
                try {
                    switch_condition = path.get('body.body.0.discriminant.object.name').node  // 控制器名称
                } catch (e) {
                    return
                }
                controler_code[switch_condition] = {}  // 整体代码有多个控制流，需要分开

                if (!path.scope.getAllBindings()[switch_condition].path.get('init.callee.object').isStringLiteral()) return
                // 取控制器，var _0x41a9c6 = "1|4|3|0|2"["split"]('|')
                eval(`controler['${switch_condition}'] = ` + path.scope.getAllBindings()[switch_condition].path.get('init').toString())
                let cases_path = path.get('body.body.0.cases')  // 拿到所有case节点，数组类型
                for (var i = 0; i  {
                        if (!c.isContinueStatement()) {
                            // 剔除case中的continue
                            controler_code[switch_condition][case_num].push(c)
                        }
                    })
                }
                let code_node = []
                for (var i = 0; i  {
                        code_node.push(n.node)
                    })
                    // code_node.push(controler_code[switch_condition][index][0].node)
                }
                path.replaceWithMultiple(code_node)
            }
        }
    }
})
```

最后，再处理一下。

### 解编码

```javascript
const transform_literal = {
    NumericLiteral({node}){
        if (node.extra && /^0[obx]/i.test(node.extra.raw)){
            node.extra = undefined;
        }
    },
    StringLiteral({node}){
        if (node.extra && /\\[ux]/gi.test(node.extra.raw)){
            node.extra = undefined;
        }
    }
}
traverse(ast, transform_literal)
```

### 表达式计算

```javascript
traverse(ast, {
    BinaryExpression: {
        exit(path){
            let {confident,value} = path.evaluate();
            if(!confident)return
            path.replaceInline({type:"NumericLiteral", value: value})
        }
    }
})
```

### 移除无用对象

```javascript
ast = parse(generator(ast, { compact: true }).code)

traverse(ast, {
    VariableDeclarator: {
        exit(path) {
            let { init, id } = path.node;
            if (!types.isObjectExpression(init) && !types.isIdentifier(id)) return;
            let { scope } = path;
            let binding = scope.getBinding(id.name);
            if (binding.referencePaths.length !== 0) return;
            path.remove();
        }
    }
})
```

成功将四千多行的代码还原到七百多行，而且逻辑也清晰多了（图不贴了）。

然后，我们再验证一下这代码能不能用，按道理来说，应该一步一验证的，但是我已经踩过坑了，所以直接一次性讲完。

具体验证方法就是去浏览器替换看能不能用，记得一定一定一定要压缩！！！保存的时候`let { code } = generator(ast, {compact: true});`将`compact`修改为`true`即可。

可以看到替换后也能成功拿到数据。

![](https://attach.52pojie.cn/forum/202412/07/033257m9pilpnlo1l0ok0c.png)

`10.png` (37.62 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjA2OHw4MzRmOTFjYnwxNzkzNTcxNTE1fDE2ODA4NnwxOTg5MDUw&nothumb=yes)

2024-12-7 03:32 上传

## 逆向分析

我们需要的逆向的参数是`type__1017`，可以搜索`type__`，可以看到数组`Jt`有，那我们就可以大胆猜测下面的`Ju`应该是我们要的值了。

![](https://attach.52pojie.cn/forum/202412/07/033259ayopqqasa4yadp32.png)

`11.png` (34.23 KB, 下载次数: 3)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjA2OXw3Y2Y5NjBhMXwxNzkzNTcxNTE1fDE2ODA4NnwxOTg5MDUw&nothumb=yes)

2024-12-7 03:32 上传

`Ju`确实是我们要的值，这个值非常好跟，抠代码靠自己了。

![](https://attach.52pojie.cn/forum/202412/07/033301brwc444r4xzcrrc6.png)

`12.png` (21.07 KB, 下载次数: 3)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjA3MHwwZDczNWQxZXwxNzkzNTcxNTE1fDE2ODA4NnwxOTg5MDUw&nothumb=yes)

2024-12-7 03:33 上传

然后我们看数据解密，老样子，我们尝试`hook` `JSON.parse`。

`hook`到响应数据的明文。

![](https://attach.52pojie.cn/forum/202412/07/033303s1i1aszu16mllsdc.png)

`13.png` (14.35 KB, 下载次数: 4)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjA3MXxjOTAwNzNjOHwxNzkzNTcxNTE1fDE2ODA4NnwxOTg5MDUw&nothumb=yes)
