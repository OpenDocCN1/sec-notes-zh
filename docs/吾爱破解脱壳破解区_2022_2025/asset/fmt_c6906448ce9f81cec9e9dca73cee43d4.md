
然后稍微整理一下一些跟字符串相关的看上去很费解的结构, 例如多个字符串相加或者 `MemberExpression` 的 `property` 是一个 `sequenceExpression` 但是实际有用的只有最后的字符串.

![](https://attach.52pojie.cn/forum/202404/19/160645rzqiie0264e420d2.png)

*1713079300044-76884725-b928-461d-ad95-6a637fd0f7a7.png (119.16 KB, 下载次数: 2) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ1OHw0MzczM2ZkNnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:06 上传*

![](https://attach.52pojie.cn/forum/202404/19/160653tigszblll1x0zxq2.png)

*1713079403699-dc63275f-e662-4b37-bb4a-b1c79292e817.png (355.86 KB, 下载次数: 2) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ1OXwyYjNkNjIzZnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:06 上传*

```
function handleBinaryExpressionString(path) {
// 去除多个字符串相加或常量相加的情况, 直接使用path.evaluate()即可. babel会计算好结果.
  const { confident, value } = path.evaluate();
    if (confident) {
        path.replaceInline(types.valueToNode(value));
    }
}

// 去除MemberExpression的property是一个sequenceExpression的情况
function removeSequenceExpressionForString(path) {
    const { node } = path;
    if (!types.isSequenceExpression(node.property)) {
        return;
    }
    const expressions = node.property.expressions;
    if (!types.isStringLiteral(expressions[expressions.length - 1])) {
        return;
    }
    path.replaceWith(types.memberExpression(node.object, expressions[expressions.length - 1], true));
}
```

![](https://attach.52pojie.cn/forum/202404/19/160704styyzv89nevytn9e.png)

*1713079742785-ec779723-1237-4180-8b28-843dab4d1fc3.png (148.12 KB, 下载次数: 2) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2MHwxN2IyYWE2NXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

以下是替换之后的效果:

![](https://attach.52pojie.cn/forum/202404/19/160711tr5e5y5xefme3xrf.png)

*1713079785670-dbf31992-9332-4ebb-bfd0-f0c4c0470749.png (142.59 KB, 下载次数: 1) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2MXxkNDc3NzQ4NHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

## 1.2 还原控制流

继续分析字符串函数还原后的代码可以知道, 代码中有很多如下这种格式的控制流结构, `for-switch` 的形式, `switch` 的是一个字符数组, 根据匹配的字符顺序进行还原即可.

![](https://attach.52pojie.cn/forum/202404/19/160721b7tgcrtcg53rzzr7.png)

*1713080108672-a206b85f-bf85-4ebf-9107-2b2732424402.png (297.25 KB, 下载次数: 0) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2MnxlOTNhN2RjZHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

首先找出用于 `switch` 的字符串数组:

```
function handleControlFlow(path) {

    const { node } = path;

    if (!types.isBlockStatement(node.body) || node.body.body.length 
```

拿到字符数组之后只需要按照字符数组的顺序重新编排代码块顺序即可, 然后替换整个 `for` 循环:

```
function handleControlFlow(path) {
    // ...上面的代码

    const cases = {};
    for (const c of switchStatement.cases) {
        cases[c.test.value] = c;
    }

    const blocks = [];

    for (const step of steps) {
        const switchCase = cases[step];
        let consequent = switchCase.consequent;
        const lastNode = consequent[consequent.length - 1];
        if (types.isContinueStatement(lastNode) || types.isBreakStatement(lastNode)) {
            consequent = consequent.slice(0, consequent.length - 1);
        }
        blocks.push(...consequent);
    }

    path.replaceWithMultiple(blocks);

}

// ...
traverse(ast, {
    ForStatement: {
        exit: [
            handleControlFlow
        ]
    }
});
```

前后代码对比:

![](https://attach.52pojie.cn/forum/202404/19/160731vlwmsj9qsmsg0lg9.png)

*1713080864025-e467ae24-559f-4d08-a860-bd3627d3e7dc.png (264.71 KB, 下载次数: 1) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2M3xkZjQxMmVkYnwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

## 1.3 还原仅有一句表达式代码的函数

![](https://attach.52pojie.cn/forum/202404/19/160741kzfogf9a8z9bmrpg.png)

*1713084145098-d9ebbd91-be09-4d2e-bbdf-91c5e5c2681c.png (13.52 KB, 下载次数: 1) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2NHwwYWJhMGY0MHwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

![](https://attach.52pojie.cn/forum/202404/19/160749vvq7jkvxgujrvfpi.png)

*1713084160758-ac5ef13d-81c6-4422-9a49-3d92dff28444.png (38.41 KB, 下载次数: 2) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2NXwxNTdhMTFmY3wxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

![](https://attach.52pojie.cn/forum/202404/19/160756bpplfco8oiicwl0z.png)

*1713084820425-6f03e966-cfef-4738-9229-39811857f836.png (15.49 KB, 下载次数: 2) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2NnxiNjlmMWIwOXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:07 上传*

![](https://attach.52pojie.cn/forum/202404/19/160804elxpilfsfspfgsg2.png)

*1713084842806-bd587db6-a3d1-4025-99df-810481db3eee.png (33.99 KB, 下载次数: 1) | [下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDQ2N3w1ODU2MDNlNXwxNzkzNTcyMzkwfDE2ODA4NnwxOTEzNTUz&nothumb=yes) | 2024-4-19 16:08 上传*

全文 `js` 代码中有大量像上图 `xtLZB` 这样的函数, 这些函数作用仅仅是对传入的参数做某一种操作并且只有一行 `return` 代码, 它们的目的仅仅是用于增加代码的阅读难度, 所以可以将这些函数调用的节点替换为 `return` 中表达式的节点.

例如:

```
var ut = {

        xtLZB: function (t, r, n, e) {

                return t(r, n, e);

              }

};

// ...

Xt = ut.xtLZB(Fg, Jt, null, 2)
```

替换为:

```
Xt = Fg(Jt, null, 2);
```

去除 `xtLZB` 函数调用后, 代码看上去会简洁很多.

首先先筛选出符合上述特征的函数调用: 函数是只有一行代码且是 `return` 语句或者除了 `return` 语句外, 还有一行是无用的 `var` 语句.

```
function handleSimpleCallExpression(path) {

    const {node} = path;

    if (!types.isMemberExpression(node.callee) || !types.isIdentifier(node.callee.object)) {

        return;

    }

    const calleeBinding = path.scope.getBinding(node.callee.object.name);

    if (!calleeBinding) {

        return;

    }

    let func = null;

    let funcScope = null;

    function tryGetFunction(v, scope, targetName, binding) {

        if (types.isIdentifier(v)) {

            binding = scope.getBinding(v.name);

            if (binding.kind === "param") {

                return;

            }

            return tryGetFunction(binding.path.node.init, binding.scope, targetName, binding);

        }

        if (types.isObjectExpression(v)) {

            const properties = v.properties;

            for (const pro of properties) {

                if (pro.key.name === targetName) {

                    return [pro.value, scope];

                }

            }

            if (!binding) {

                return ;

            }

            // 从赋值语句中查找

            for (const ref of binding.referencePaths) {

                if (!types.isAssignmentExpression(ref.parentPath.parent)) {

                    continue;

                }

                const assign = ref.parentPath.parent;

                if (!types.isMemberExpression(assign.left)) {

                    continue;

                }
```
