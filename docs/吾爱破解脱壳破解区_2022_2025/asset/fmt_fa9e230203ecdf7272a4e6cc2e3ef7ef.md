
# Babel AST 遍历与 `kA` 函数解析

```js
const traverse = require("@babel/traverse").default;

const input = "js_security_v3_0.1.6.js";

const content = fs.readFileSync(input, {encoding: "utf-8"});

const ast = parser.parse(content);

function handleStringFunction(path) {

    const { node } = path;

  // 找到kA函数

  if (node.id.name !== "kA" || node.params.length !== 2) {

        return;

    }

}

traverse(ast, {

    FunctionDeclaration: {

        enter: [

            handleStringFunction

        ]

    }

});
```

然后把 `kA` 函数得到的字符串数组复制过来, 定义为 `kAWords` 数组.

![](https://attach.52pojie.cn/forum/202404/19/160416cg319xbefuh9lax8.png)

1712759175664-f5540819-73df-42f8-89ef-f765df7047f5.png (168.84 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MDQ0NXxhNjNiOTVhZnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ0NXxhNjNiOTVhZnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:04 上传

接着使用 `path.scope` 查找所有调用 `kA` 的地方并将其放到一个 `to_visit` 的队列中, 便于遍历所有情况, 如果调用 `kA` 传递的参数不是具体数值还需要将当前调用者所处的函数的所有调用记录加到 `to_visit` 中, 如果是数值则可以直接得到结果并替换调用节点为字符串节点.

![](https://attach.52pojie.cn/forum/202404/19/160426cruy2133ygprj329.png)

1713020679649-cf3b7b9a-8bdb-41c7-8d1a-c21c280373b9.png (58.78 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MDQ0NnwzNDY1MTZjN3wxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ0NnwzNDY1MTZjN3wxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:04 上传

## 传入 `kA` 的参数分析

### (1) 全是数字

例如:

```
kA(100, 200)
```

在 `astexplorer.net` 中分析, 此时可以清楚地知道它的 `arguments` 中的第一个参数是 `NumericLiteral` 类型 (对于 `kA` 函数来说, 它的返回值只跟第一个参数有关, 所以只用看第一个参数就行).

![](https://attach.52pojie.cn/forum/202404/19/160438xnsl6rvmqpunavub.png)

1713018430330-7a0ad4ba-b6ca-41b5-ba52-d288cb0a59e6.png (302.15 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY5MDQ0N3wzMDcyYWE5Y3wxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ0N3wzMDcyYWE5Y3wxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:04 上传

因此可以写如下函数来获取值:

![](https://attach.52pojie.cn/forum/202404/19/160449hk2g88ddkg8hygga.png)

1713019764125-e1bd6552-ed46-4424-8f7f-6dbe1fffc9c1.png (49.63 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MDQ0OHw3ZTQ3OTA3ZXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ0OHw3ZTQ3OTA3ZXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:04 上传

### (2) 传入参数是变量加偏移

分析 `kA` 函数的调用列表可知道, 有很多这种情况.

![](https://attach.52pojie.cn/forum/202404/19/160502frlafdf861ma1ar4.png)

1713019148635-b15fcde0-10ed-4f55-8cc0-9fd4e1dc44fb.png (245.86 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MDQ0OXxhNWMzOTY4OHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ0OXxhNWMzOTY4OHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:05 上传

抓住其中一个调用分析它的 `ast` 结构, 着重看第一个参数的 `ast` 结构.

![](https://attach.52pojie.cn/forum/202404/19/160513oa7zyu90s39y924u.png)

1713019416594-6473c4c8-4295-4a36-86e6-ecccd6b81414.png (122.56 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MDQ1MHwwNDU2NDZlOXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ1MHwwNDU2NDZlOXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:05 上传

![](https://attach.52pojie.cn/forum/202404/19/160520tvz0c3g72983g558.png)

1713020135765-61e4d14f-e15f-4e6b-bba3-036f361af46e.png (312.45 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MDQ1MXw0MjhjM2EyNnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ1MXw0MjhjM2EyNnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:05 上传

![](https://attach.52pojie.cn/forum/202404/19/160529u999s9bnzhgt9st9.png)

1713020260091-0e809455-d2bf-400b-8113-f3112b03117f.png (522.46 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MDQ1MnwyM2Q3MzJmMHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ1MnwyM2Q3MzJmMHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes)

2024-4-19 16:05 上传

可以看出来, 第一个参数是一个 `BinaryExpression` 表达式, 这个表达式的 `left` 是一个 `Identifier`, `right` 是 `NumericLiteral` 或 `UnaryExpression`, `operator` 可以是 `"-"`, `"+"` 之类. 现在我们只需要找到这个 `left` 和 `right` 对应的值是多少就计算出该表达式的具体数值. 使用 `path.scope` 可以获取到作用域内变量的定义和初始值/是否常量等信息, 可以找到初始值, 找一个 `Identifier` 的初始值的方法类似如下代码:

```
function getIdentifierNumber(identifier, scope) {
    // scope: 作用域
    // 得到binding
    const binding = scope.getBinding(identifier.name);
    if (!binding) {
        return;
    }
    // 当前变量是常量且初始值是数值类型则可以得到数值
    if (binding.constant && types.isNumericLiteral(binding.path.node.init)) {
        return binding.path.node.init.value;
    }
      // 初始值是null且只有一个constantViolations且是数值
    if (binding.path.node.init === null && binding.constantViolations.length === 1) {
        const node = binding.constantViolations[0].node;
        if (!types.isAssignmentExpression(node)) {
            return;
        }
        return tryGetArgNumber(node.right, binding.constantViolations[0].scope);
    }
}
```

`BinaryExpression` 表达式可以分别计算 `left` 和 `right` 的值, 然后根据 `operator` 计算数值; `isUnaryExpression` 可以计算 `argument` 的值并计算数值.

```
function tryGetArgNumber(arg, scope) {
    // 数值类型直接取value即可
    if (types.isNumericLiteral(arg)) {
        return arg.value;
    }
    // 变量类型查找所在变量的binding, 并获取初始值
    if (types.isIdentifier(arg)) {
        return getIdentifierNumber(arg, scope);
    }
    // UnaryExpression类型只需要获取UnaryExpression的argument并进行operator操作就可以得到数值
    if (types.isUnaryExpression(arg)) {
        const argumentValue = tryGetArgNumber(arg.argument, scope);
        if (typeof argumentValue !== "number") {
            return;
        }
        return eval(`${arg.operator} ${argumentValue}`)
    }
          // 计算left和right的值
    if (types.isBinaryExpression(arg)) {
        let leftValue = tryGetArgNumber(arg.left, scope);
        const rightValue = tryGetArgNumber(arg.right, scope);
        if (typeof leftValue !== "number" || typeof rightValue !== "number") {
            return ;
        }
        return eval(`${leftValue} ${arg.operator} ${rightValue}`);
    }
}
```

对于传入参数是 `param` 类型即传入的是当前函数的参数, 可以获取当前函数的所有调用记录并加上一个偏移值, 然后继续加入到 `to_visit` 数组中遍历.

举个例子, 有如下函数定义:

```
function ob(e, t) {
  return kA(e + 100, t);
}

const result = ob(100, 0);
```

那么 `result` = `ob`(100, 0) = `kA`(200, 0) = "逆向研究点滴"

示例解析代码如下:
