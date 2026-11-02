
```javascript
const types = require("@babel/types");

const generator = require("@babel/generator").default;

const fs = require('fs');

function base64Encode(e) {
    var r, a, c, h, o, types, base64EncodeChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';

    for (c = e.length, a = 0, r = ''; a > 2),

            r += base64EncodeChars.charAt((3 & h) > 2),

            r += base64EncodeChars.charAt((3 & h) > 4),

            r += base64EncodeChars.charAt((15 & o) > 2),

        r += base64EncodeChars.charAt((3 & h) > 4),

        r += base64EncodeChars.charAt((15 & o) > 6),

        r += base64EncodeChars.charAt(63 & types)

    }

    return r
}

function hexEnc(code) {
    for (var hexStr = [], i = 0, s; i  0){
        retval.push(decNum % 3);

        decNum = parseInt(decNum / 3);
    }

    let Identifier = retval.reverse().map(function(v){
        return flag[v]
    }).join('');

    Identifier.length 1000 key==>666 cipherNum===>370

    let value = path.node.value;

    let key = parseInt(Math.random() * (999999 - 100000) + 100000, 10);

    let cipherNum = value ^ key;

    path.replaceWith(types.binaryExpression('^', types.numericLiteral(cipherNum), types.numericLiteral(key)));

    //替换后的节点里也有numericLiteral节点，会造成死循环，因此需要加入path.skip()

    path.skip();
},

StringLiteral(path){
    let encStr = types.callExpression(
             types.identifier('atob'), [types.stringLiteral(base64Encode(path.node.value))]);

    path.replaceWith(encStr);

    path.skip();
},

StringLiteral(path){
    let cipherText = base64Encode(path.node.value);

    // 检查是否数组里面已经存在

    let bigArrIndex = bigArr.indexOf(cipherText);

    let index = bigArrIndex;

    if(bigArrIndex == -1){
        let length = bigArr.push(cipherText);

        index = length -1;
    }

    let encStr = types.callExpression(
                types.identifier('atob'),

                [types.memberExpression(types.identifier('arr'),
                                 types.numericLiteral(index), true)]);

    path.replaceWith(encStr);
},

'Program|FunctionExpression|FunctionDeclaration'(path) {
    renameOwnBinding(path);
},

});

bigArr = bigArr.map(function(v){
    return types.stringLiteral(v);
});

//声明arr 并且加入到最前面

bigArr = types.variableDeclarator(types.identifier('arr'), types.arrayExpression(bigArr));

bigArr = types.variableDeclaration('var', [bigArr]);

ast.program.body.unshift(bigArr);

let oneCode = generator(ast).code;

fs.writeFile('./AfterAstCode.js', oneCode, (err)=>{});

console.log("end...");
```
