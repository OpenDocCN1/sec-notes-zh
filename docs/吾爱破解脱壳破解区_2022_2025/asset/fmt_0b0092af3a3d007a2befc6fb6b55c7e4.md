
## 函数实现

函数名为`autoRestoreStringLiteralViaIIFE`，顾名思义，这里我选择的切入点就是自执行函数。[`github.com/Hans774882968/control-flow-flattening-remove-public/blob/main/src/restoreStringLiteral.ts`](https://github.com/Hans774882968/control-flow-flattening-remove-public/blob/main/src/restoreStringLiteral.ts)：

```ts
import {
  isArrayExpression,
  isBlockStatement,
  isCallExpression,
  isExpressionStatement,
  isFunctionExpression,
  isIdentifier,
  isNumericLiteral,
  isReturnStatement,
  isStringLiteral,
  isVariableDeclaration,
  Node,
  stringLiteral,
  File
} from '@babel/types';
import traverse from '@babel/traverse';
import { strict as assert } from 'assert';
import generator from '@babel/generator';

// 如果常量表不止1处，则此代码不正确
export function restoreStringLiteral (ast: Node, stringLiteralFuncs: string[], getStringArr: (idx: number) => string) {
  // 收集与常量串隐藏有关的变量
  traverse(ast, {
    VariableDeclarator (path) {
      const vaNode = path.node;
      if (!isIdentifier(vaNode.init) || !isIdentifier(vaNode.id)) return;
      if (stringLiteralFuncs.includes(vaNode.init.name)) {
        stringLiteralFuncs.push(vaNode.id.name);
      }
    }
  });
  traverse(ast, {
    CallExpression (path) {
      const cNode = path.node;
      if (!isIdentifier(cNode.callee)) return;
      const varName = cNode.callee.name;
      if (!stringLiteralFuncs.includes(varName)) return;
      const literalNode = cNode.arguments[0];
      if (cNode.arguments.length !== 1 || (!isNumericLiteral(literalNode) && !isStringLiteral(literalNode))) return;
      const idx = Number(literalNode.value);
      path.replaceWith(stringLiteral(getStringArr(idx)));
    }
  });
}

export function rotateArray (a: T[], count: number) {
  count %= a.length;
  return [...a.slice(count), ...a.slice(0, count)];
}

export function autoRestoreStringLiteralViaIIFE (ast: File) {
  let constArrName = '';
  const INITIAL_SHIFT_NUM = -1234567;
  let shiftNum = INITIAL_SHIFT_NUM;
  ast.program.body.findIndex((bodyItem) => {
    if (!isExpressionStatement(bodyItem) ||
        !isCallExpression(bodyItem.expression) ||
        !isFunctionExpression(bodyItem.expression.callee) ||
        bodyItem.expression.arguments.length !== 2) return false;
    const [arg0, arg1] = bodyItem.expression.arguments;
    if (!isIdentifier(arg0) || !isNumericLiteral(arg1)) return false;
    constArrName = arg0.name;
    shiftNum = arg1.value;
    return true;
  });
  assert.ok(constArrName);
  assert.notEqual(shiftNum, INITIAL_SHIFT_NUM);

  let constArrContent: string[] = [];
  let stringHideVarName = '';
  let globalOffset = 0;
  traverse(ast, {
    VariableDeclaration (path) {
      const decl = path.node.declarations[0];
      if (!isIdentifier(decl.id)) return;
      if (decl.id.name === constArrName && isArrayExpression(decl.init)) {
        constArrContent = decl.init.elements.map((item) => {
          assert.ok(isStringLiteral(item));
          return item.value;
        });
      }
      if (isFunctionExpression(decl.init)) {
        if (decl.init.params.length !== 2 ||
            !isBlockStatement(decl.init.body) ||
            decl.init.body.body.length !== 3) return;
        const [s1, s2, s3] = decl.init.body.body;
        if (!isExpressionStatement(s1) ||
            !isVariableDeclaration(s2) ||
            !isReturnStatement(s3)) return;

        path.traverse({
          BinaryExpression (path) {
            assert.ok(isNumericLiteral(path.node.right));
            globalOffset = path.node.right.value;
          }
        });

        const { code } = generator(s2);
        if (!code.includes(constArrName)) return;
        stringHideVarName = decl.id.name;
      }
    }
  });
  constArrContent = rotateArray(constArrContent, shiftNum);

  restoreStringLiteral(ast, [stringHideVarName], (idx: number) => {
    return constArrContent[idx - globalOffset];
  });
}
```

## 完整代码

```ts
import {
  isIdentifier,
  isStringLiteral
} from '@babel/types';
import traverse from '@babel/traverse';
import CryptoJS from 'crypto-js';
import superagent from 'superagent';
import * as cheerio from 'cheerio';
import * as parser from '@babel/parser';
import fs from 'fs';
import { strict as assert } from 'assert';
import path from 'path';
import { autoRestoreStringLiteralViaIIFE } from '../restoreStringLiteral';
import os from 'os';
import { getLegalFileName } from '../file_utils';

export function parseHTML (inputHTML: string, data: string, keywordsEnc: string, ivEnc: string) {
  const keywords = CryptoJS.enc.Latin1.parse(keywordsEnc);
  const iv = CryptoJS.enc.Latin1.parse(ivEnc);
  const decrypted = CryptoJS.AES.decrypt(data, keywords, {
    iv,
    padding: CryptoJS.pad.ZeroPadding
  });
  const secWords = decrypted.toString(CryptoJS.enc.Utf8).split(',');
  // console.log('secWords', secWords); // dbg
  const words: string[] = [];
  for (let i = 0; i /g, (span) => {
    const idx = Number((span.match(/\d+/) || [])[0]);
    return words[idx];
  });
  const $ = cheerio.load(htmlHasOnlyP);
  return $('p').text().split('\n').map((txt) => txt.trim()).join(os.EOL);
}

export function getDataFromJSCode (jsCode: string) {
  const ast = parser.parse(jsCode);
  autoRestoreStringLiteralViaIIFE(ast);
  let data = '';
  let keywordsEnc = '';
  let ivEnc = '';
  traverse(ast, {
    VariableDeclarator (path) {
      const idNode = path.node.id;
      if (!isIdentifier(idNode)) return;
      if (idNode.name === 'keywords') {
        path.traverse({
          CallExpression (path) {
            const args = path.node.arguments;
            assert.equal(args.length, 1);
            assert.ok(isStringLiteral(args[0]));
            keywordsEnc = args[0].value;
          }
        });
      }
      if (idNode.name === 'data') {
        assert.ok(isStringLiteral(path.node.init));
        data = path.node.init.value;
      }
    },
    AssignmentExpression (path) {
      if (!isIdentifier(path.node.left) || path.node.left.name !== 'iv' || ivEnc) return;
      path.traverse({
        CallExpression (path) {
          const args = path.node.arguments;
          assert.equal(args.length, 1);
          assert.ok(isStringLiteral(args[0]));
          ivEnc = args[0].value;
        }
      });
    }
  });
  return {
    data, keywordsEnc, ivEnc
  };
}

export function getDataFromHTML ($: cheerio.CheerioAPI) {
  const scriptTags = $('script');
  let data = '', keywordsEnc = '', ivEnc = '';
  scriptTags.each((i, el) => {
    const jsCode = $(el).text();
    if (!jsCode.includes('CryptoJS')) return true;
    const res = getDataFromJSCode(jsCode);
    data = res.data;
    keywordsEnc = res.keywordsEnc;
    ivEnc = res.ivEnc;
    return false;
  });
  return {
    data, keywordsEnc, ivEnc
  };
}

export function getInputHTML (text: string) {
  const $ = cheerio.load(text);
  return { $, inputHTML: $('.rdtext').html() || '' };
}

function main () {
  const novelURLs = [
    // html列表，简单的逐个遍历
  ];
  const headers = {
    'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
    'accept-language': 'zh-CN,zh;q=0.9',
    'cache-control': 'no-cache',
    'pragma': 'no-cache',
    'sec-ch-ua': '"Not_A Brand";v="99", "Google Chrome";v="109", "Chromium";v="109"',
    'sec-ch-ua-mobile': '?0',
    'sec-ch-ua-platform': '"Windows"',
    'sec-fetch-dest': 'document',
    'sec-fetch-mode': 'navigate',
    'sec-fetch-site': 'same-origin',
    'sec-fetch-user': '?1',
    'upgrade-insecure-requests': '1',
    'Referer': '',
    'Referrer-Policy': 'strict-origin-when-cross-origin'
  };
  novelURLs.forEach(async (novelURL) => {
    const resp = await superagent.get(novelURL).set(headers);
    console.log('resp.text', resp.text.substring(0, 100)); // dbg
    const { $, inputHTML } = getInputHTML(resp.text);
```
