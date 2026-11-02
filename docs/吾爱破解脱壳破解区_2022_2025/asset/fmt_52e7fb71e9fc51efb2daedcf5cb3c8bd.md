
```js
          !isStringLiteral(tableProperty.key)) return cffTable;
        cffTable[tableProperty.key.value] = tableProperty.value;
        return cffTable;
      }, {} as ASTNodeMap);
    }
  });

  traverse(ast, {
    CallExpression (path) {
      const cNode = path.node;
      if (isMemberExpression(cNode.callee)) {
        if (!isIdentifier(cNode.callee.object)) return;
        const callParams = cNode.arguments;
        const tableName = cNode.callee.object.name;
        if (!isStringLiteral(cNode.callee.property)) return;
        const keyName = cNode.callee.property.value;
        if (!(tableName in cffTables) ||
            !(keyName in cffTables[tableName])) return;
        const shouldBeFuncValue = cffTables[tableName][keyName];
        if (!isFunctionExpression(shouldBeFuncValue) ||
            !shouldBeFuncValue.body.body.length ||
            !isReturnStatement(shouldBeFuncValue.body.body[0])) return;
        // 拿到返回值
        const callArgument = shouldBeFuncValue.body.body[0].argument;
        if (isBinaryExpression(callArgument) && callParams.length === 2) {
          if (!isExpression(callParams[0]) || !isExpression(callParams[1])) {
            throw '二元运算符中，两个参数都应为表达式';
          }
          // 处理function(x, y){return x + y}这种形式
          path.replaceWith(binaryExpression(callArgument.operator, callParams[0], callParams[1]));
        } else if (isLogicalExpression(callArgument) && callParams.length === 2) {
          if (!isExpression(callParams[0]) || !isExpression(callParams[1])) {
            throw '逻辑运算符中，两个参数都应为表达式';
          }
          // 处理function(x, y){return x > y}这种形式
          path.replaceWith(logicalExpression(callArgument.operator, callParams[0], callParams[1]));
        } else if (isCallExpression(callArgument) && isIdentifier(callArgument.callee)) {
          // 处理function(f, ...args){return f(...args)}这种形式
          if (callParams.length == 1) {
            path.replaceWith(callParams[0]);
          } else {
            if (!isExpression(callParams[0])) {
              throw '仅支持第一个参数为函数的形式，如：function(f, ...args){return f(...args)}';
            }
            path.replaceWith(callExpression(callParams[0], callParams.slice(1)));
          }
        }
      }
    },
    MemberExpression (path) {
      const mNode = path.node;
      if (!isIdentifier(mNode.object)) return;
      const tableName = mNode.object.name;
      if (!isStringLiteral(mNode.property)) return;
      const keyName = mNode.property.value;
      if (!(tableName in cffTables) ||
          !(keyName in cffTables[tableName])) return;
      const cffTableValue = cffTables[tableName][keyName];
      path.replaceWith(cffTableValue);
    }
  });
}

cff(ast);

function removeUselessCodes (ast: Node) {
  traverse(ast, {
    // 去除给string数组进行随机移位的自执行函数
    CallExpression (path) {
      if (!isFunctionExpression(path.node.callee)) return;
      if (path.node.arguments.length !== 2 ||
          !isNumericLiteral(path.node.arguments[1]) ||
          path.node.arguments[1].value !== 0x20d95) return;
      path.remove();
    },
    // 去除给string数组进行随机移位的函数
    FunctionDeclaration (path) {
      if (!isIdentifier(path.node.id)) return;
      const funcName = path.node.id.name;
      if (!['_0x546b', '_0x3ddf'].includes(funcName)) return;
      path.remove();
    },
    // 去除控制流平坦化的哈希表和用于隐藏常量串的变量
    VariableDeclarator (path) {
      if (!isIdentifier(path.node.id)) return;
      const varName = path.node.id.name;
      // 前两个变量是控制流平坦化的哈希表，后两个是用于隐藏常量串的变量
      if (!['_0xd90ee7', '_0x2d1e4f', '_0x583e52', '_0x583af1'].includes(varName)) return;
      path.remove();
    }
  });
}

removeUselessCodes(ast);

memberExpComputedToFalse(ast);

renameVars(
  ast,
  (name:string) => name.substring(0, 3) === '_0x',
  {
    check_pass: 'check_pass', test: 'test', _0x39ead2: 'i',
    _0x2ce438: 'sum', _0xaa86db: 'password'
  }
);

translateLiteral(ast);

const { code } = generator(ast);

writeOutputToFile('check_pass_demo_out.js', code);
```

还原效果（完美！）：

```js
function check_pass (password) {
  var i = 0,
    sum = 0;
  for (i = 0;; i++) {
    if (i == password.length) break;
    sum = sum + password.charCodeAt(i);
  }
  if (i == 4) {
    if (sum == 417 && password.charAt(3) > 'c' && password.charAt(3)
```
