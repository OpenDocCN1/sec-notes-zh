
```
return generatedSource;

}

const compiledCode = extractEvalSource(ast);

eval(compiledCode);

const stringLiteralOptimizer = {

  CallExpression(path) {

    const { node } = path;

    if (!node.callee.object || typeof node.callee.object.name !== 'string') return;

    if (!node.callee.property || node.callee.property.name !== 'substr') return;

    const sourceVar = node.callee.object.name;

    const startIndex = node.arguments[0].extra.rawValue;

    const lengthParam = node.arguments[1].extra.rawValue;

    const computedValue = eval(`${sourceVar}.substr(${startIndex}, ${lengthParam})`);

    path.replaceWith(types.StringLiteral(computedValue));

  }

};

traverse(ast, stringLiteralOptimizer);
```

看看处理后的效果对比

![](https://attach.52pojie.cn/forum/202509/24/200336l08a2h82mham0a82.png)
16.png (118.76 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjgwNTA2NHw5ZWRkOGI5MnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA2NHw5ZWRkOGI5MnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

## 接着处理二三部分ob混淆

- 先遍历所有 `VariableDeclarator`（变量声明节点）
- 检查变量是否由 `a1_0x89f8` 或其“别名变量”初始化
- 将符合条件的变量名加入 `trackedIdentifiers` 集合收集起来

```
const trackedIdentifiers = new Set();
const declarationVisitor = {
  VariableDeclarator(path) {
    const { node } = path;
    if (!node.id?.name || !node.init?.name) return;
    const declaredName = node.id.name;
    const initializerName = node.init.name;
    if (initializerName === 'a1_0x89f8' || trackedIdentifiers.has(initializerName)) {
      trackedIdentifiers.add(declaredName);
    }
  }
};
traverse(ast, declarationVisitor)
```

然后将第二部分代码执行，声明出解密环境

```
obEnv = `第二部分代码`

eval(obEnv)
```

再将符合条件的表达式进行函数调用解密

```
const callExpressionVisitor = {
  CallExpression(path) {
    const { node } = path;
    const calleeName = node.callee.name;
    if (
      node.arguments.length !== 1 ||
      !node.arguments[0].value ||
      typeof node.arguments[0].value !== 'string' && typeof node.arguments[0].value !== 'number'
    ) {
      return;
    }
    const argValue = node.arguments[0].value;
    if (trackedIdentifiers.has(calleeName)) {
      if (typeof a1_0x89f8 !== 'function') {
        console.warn(`Function a1_0x89f8 is not defined.`);
        return;
      }
      try {
        const result = a1_0x89f8(argValue);
        path.replaceWith(types.stringLiteral(String(result)));
      } catch (err) {
        console.error(`Error evaluating ${calleeName}(${argValue}):`, err);
      }
    }
  }
};
traverse(ast, callExpressionVisitor);
```

最后再来个常量折叠

```
const constantFold = {

"BinaryExpression|UnaryExpression"(path) {

      if(path.isBinaryExpression({operator:"/"})||

         path.isUnaryExpression({operator:"-"}) || 

         path.isUnaryExpression({operator:"void"}))

    {

      return;

    }

      const {confident, value} = path.evaluate();

      if (!confident)

          return;

      if (typeof value == 'number' && (!Number.isFinite(value))) {

          return;

      }

      path.replaceWith(types.valueToNode(value));

  },

}

traverse(ast, constantFold);
```

看看效果对比

![](https://attach.52pojie.cn/forum/202509/24/200338kgmioizd6s7dp0m0.png)
17.png (208.41 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwNTA2NXxlYzBiYTczN3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA2NXxlYzBiYTczN3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

到此解混淆差不多了，能搜索关键字就够了，当然，还有继续优化的空间，比如优化下三目运算，删除无用死代码，一些美化格式啥的，感兴趣可以继续去研究

### 定位加密位置

先将我们上一步解混淆的文件替换到浏览器中（如何替换这里不多赘述）

替换有一个关键点需要注意，解混淆文件中有一个路径代码如下

![](https://attach.52pojie.cn/forum/202509/24/200340eiglxxj2x8y7zj7y.png)
18.png (35.89 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwNTA2NnwyN2ExZDBhMXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA2NnwyN2ExZDBhMXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

需要改为和当前时间段的请求路径一致

比如，我现在当前请求路径如下

![](https://attach.52pojie.cn/forum/202509/24/200342h3foo337hxzxk7yx.png)
19.png (31.64 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjgwNTA2N3w0MTBmNmUzZnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA2N3w0MTBmNmUzZnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

那么解混淆文件中的路径就需要改为

`3nOsRQ8irc_ZtcVNrFJG8gbVR_3mK-NK2L-00vw-NqY`

不然替换后会报错，报错看控制台提示如下就说明得改路径了

![](https://attach.52pojie.cn/forum/202509/24/200347iykw6fmj6yfiofxp.png)
20.png (35.59 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwNTA2OXw0ODgxOTM5OHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA2OXw0ODgxOTM5OHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

替换完后打上关键xhr断点执行到如下位置

![](https://attach.52pojie.cn/forum/202509/24/200349hz9ghttsczjogbxo.png)
21.png (29.21 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjgwNTA3MHw0OGUwMjQ1OHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3MHw0OGUwMjQ1OHwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

在抓包分析中说过的第一次post请求传一个固定载荷

放开断点继续又跳到这个位置

![](https://attach.52pojie.cn/forum/202509/24/200351shv4rsfufsohz9sz.png)
22.png (21.2 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwNTA3MXwxMWQ5ZDJlOXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3MXwxMWQ5ZDJlOXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

第二次的post请求，有我们需要的关键载荷

所以我们补环境能到这个位置就算成功了

继续分析

js内搜索一下关键字 `aih` 定位到如下位置

![](https://attach.52pojie.cn/forum/202509/24/200354wr4ijcz848i8imip.png)
23.png (62.44 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwNTA3Mnw3ZmI3ZTIyZXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3Mnw3ZmI3ZTIyZXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

能看到一些参数，往下看 `return` 了一个 `Promise`

下个断点到这看看

清空一下cookie和storage缓存（建议每次重写访问都清一下）

这里有个关键点，动态链接断点断不住怎么办

直接在js文件第一行加一行`debuuger;`

![](https://attach.52pojie.cn/forum/202509/24/200356lqyo83ynua8auoga.png)
24.png (17 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjgwNTA3M3xmZDE2MzNhNnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3M3xmZDE2MzNhNnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

进入js文件第一行了，再定位到`Promise`位置下断点发现还是跳不过去

仔细一点能发现它进入的路径变了

![](https://attach.52pojie.cn/forum/202509/24/200358kdg6z8y366a6la9c.png)
25.png (15.32 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjgwNTA3NHw4NDMwOWY1M3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3NHw4NDMwOWY1M3wxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:03 上传

进入了一个`cachebuster`

搜索一下发现其生成位置

![](https://attach.52pojie.cn/forum/202509/24/200401rtosco4t6szzi188.png)
26.png (9.98 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjgwNTA3NXxmNTkxMzJiZnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3NXxmNTkxMzJiZnwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:04 上传

往上找这个函数的进入位置

![](https://attach.52pojie.cn/forum/202509/24/200403j2s8jr0qe788qcrq.png)
27.png (34.49 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjgwNTA3NnxjODA3Yzg1OXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTA3NnxjODA3Yzg1OXwxNzkzNTcwNTQ5fDE2ODA4NnwyMDYyMzc3&nothumb=yes)
2025-9-24 20:04 上传

发现是在一个`reloadScript`方法里进行了脚本重载

这里直接说原因

搜索关键字 `reese84interrogatorconstructor`

![](https://attach.52pojie.cn/forum/202509/24/200405iajzzgom3crp2rcy.png)
28.png (9.71 KB, 下载次数: 3)
