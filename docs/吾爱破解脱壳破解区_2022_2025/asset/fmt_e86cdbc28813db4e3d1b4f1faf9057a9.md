
# [Web逆向] 某影视站加密 `js` 的还原及自动化获取真实视频地址

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1671722
- 抓取时间: 2026-11-01T23:11:48
- 主页：`aHR0cHM6Ly93d3cuNXlpbmcuZnVuLw==`
- 地址：`aHR0cHM6Ly93d3cuNWR5Ni52aXAvdm9kcGxheS85MTA5Ny02LTEuaHRtbA==`

## 一、分析执行流程

### 1. 干掉 `debugger` 反调试

首先打开 `f12` 后 进入上面的地址，会出现如下的情况

![](https://img-blog.csdnimg.cn/7851d2d268c54c628a47a66f9b17fe77.png)

这里我采用最简单的方案干掉他，点击这个忽略断点，就不会进入此 `debug`，但是会有另一个问题，就是 `cpu` 使用率是变高，但是因为我们只分析执行流程，找到关键的 `js`，所以这里就无所谓了

![](https://img-blog.csdnimg.cn/068cdab5c7154be1b3b0183cf2a00b8f.png)

### 2. 分析谁发起了 `index.m3u8` 的调用

我这边因为分析过多次，发现每次再请求 `.m3u8` 之前，都会有一次 `webscoket` 的请求，那么断定 `websocket` 一定是非常重要的

![](https://img-blog.csdnimg.cn/ca3a9a0a1ad44a6dad8186233c872003.png)

其实在这里我们已经拿到了 `m3u8` 的视频地址，但是只拿到 `m3u8` 的地址不是咱们的目的。

当我们点击上图右边蓝色链接进入这个 `main.js` 的这个 `js` 文件中发现，他是加密的如下图

![](https://img-blog.csdnimg.cn/a872e59223864f81b9e6c8b68b8e31a0.png)

其实也可以侧面的反应出，我们的推断是没错的。如果不是非常重要的东西，他基本不可能加密

### 3. 不正常的代码

当我们再网页上格式化之后，会发现很多不太正常的代码，如下图

![](https://img-blog.csdnimg.cn/86bb8aae61db463cba5c232ad9e8e534.png)

我们发现这个图的很多代码的格式他是不正确的

例如803行的 `if` 语句，如下图

![](https://img-blog.csdnimg.cn/66cd4c5ad624476bb10ea18f6c3cb7c0.png)

- 当我们把光标放到左变括号的时候，他右边的括号高亮的居然也是一个左括号，同时他的最后边没有任何括号。
- 我们知道再程序开发中，括号都是成对出现的，不可能出现这种所谓的左括号 的右边还是左括号的情况，但是他这里得代码缺没有任何得报错，说明这个代码本身是没有任何问题得。
- 那么产生这个问题的原因其实很简单，我们知道再 `ascii` 中有一个特殊的字符，这个字符的功能就是会将在他之前的各种字符串进行反转，然后加密就使用了这个字符来扰乱我们的分析。

## 二、还原代码

我们将 `main.js` 的代码复制下来，然后开始进行还原,我是复制了两份一份是没有格式化的代码，另一分是格式化过的代码。

### 1. 环境问题

- 需要 `node` 环境
- 安装 `babel`

### 2. 大致分析一下 `js` 代码

我们可以看到整个 `_0x3a74` 这个函数一共有1041个之多，说明这个函数就是解密字符串的重要函数

![](https://img-blog.csdnimg.cn/f516393239bc4d8ba1a5ca40c8e924d5.png)

### 3. 抽取解密函数

我们将这个 `_0x3a71` 的函数包括他之前的所有代码全部复制出来保存到名叫 `decrypt.js` 的文件中。

![](https://img-blog.csdnimg.cn/49ddd92f462648a085e0c34530426ee1.png)

然后再 `decrypt.js` 文件末尾增加两个导出，以供我们代码调用使用

![](https://img-blog.csdnimg.cn/7ca7a4e3fcfa4aeb9042744fb715f415.png)

### 4. 第一次还原

#### 4.1 还原代码如下

```
const parser = require("@babel/parser");
const template = require("@babel/template").default;
const traverse = require("@babel/traverse").default;
const t = require("@babel/types");
const generator = require("@babel/generator").default;

const path = require("path");
const fs = require("fs");

const {decryptStr,decryptStrFunName } = require("E:\\desktop\\2\\decrypt.js");
let filepath = "e:/desktop/main.js";

fs.readFile(filepath,{"encoding":"utf-8"},function(err,data) {
        const ast = parser.parse(data);
        step(ast);

        let {code} = generator(ast);
        fs.writeFile("e:/desktop/main1.js",code,err => {
                if(err) throw err;
                console.log("保存成功");

        });
});

function step(ast) {
        traverse(ast,{
                CallExpression: ObjToStr,
        })
}

function ObjToStr(path) {
   let node = path.node;
   try{
           if(node.callee.name == decryptStrFunName && node.arguments.length == 2) 
           {

                   let val = decryptStr(node.arguments[0].value,node.arguments[1].value);
                   path.replaceWith(t.stringLiteral(val));
           }
   }catch(e) {
           console.log(e);
   }

}
```

#### 4.2 开始运行

- 当我们运行的时候发现他报了一个错误，如下图

![](https://img-blog.csdnimg.cn/f0d965ebcce945fe99de1295c59ab91e.png)

- 那么我们开始调试，我们再 `decrypt.js` 的71行下断

![](https://img-blog.csdnimg.cn/7ba06976b79c4d9090ae29b9f9c1567b.png)

我们发现这里他会进入一个叫 `setcookie` 的函数里

- 这里我采用了一个简单的方案，就是我从网页分析这个执行流程，发现他应该走得下面得 `getcookie`，而且我发现他得这个函数是个循环，如下图。那么我们已经知道他应该进入 `getCookie` 那么我们就不浪费时间了。

![](https://img-blog.csdnimg.cn/a66ac9d00b374df6a5018580188e2a9d.png)

- 我们可以看到进入 `getCookie` 函数的条件是 `_0x163f4` 这个函数返回为 `true`，那么我们就直接让他返回 `true`，当我们再次运行的时候发现他已经可以进入 `getCookie` 了，此时这个暗桩被我门解决了

![](https://img-blog.csdnimg.cn/9ebdcf37019141838a31126df405116f.png)

- 当我们再次运行的时候，发现程序又报错了

![](https://img-blog.csdnimg.cn/3fd42a1f46c24e0386edb789c9ebe900.png)

- 我们先再我门的还原代码处调用函数处下断，如下图

![](https://img-blog.csdnimg.cn/5491fd6ab3f943e2b99bb8fe831a6bfa.png)

- 当我们单步跟踪到此处的时候，如下图，发现他会进入一个函数

![](https://img-blog.csdnimg.cn/f034e36218e0442f82b55f0ceb187fae.png)

- 当我们再进入这个函数的时候，就发现了有意思的东西出现了

![](https://img-blog.csdnimg.cn/c9df7b9968aa4008adb0b9cf6311fc31.png)

- 他会按照图上标注的顺序再以此向下执行
- 当执行到第4个函数的时候，迟迟没有执行到 `return`
- 分析代码可知，他做的事情是：
  - `push` 一个数到数组，
  - 取数组的长度赋值给 `_0x492e66`，
  - `for` 循环的退出条件是 `_0x5da125` 大于 `_0x492e66`，
  - `_0x5da125` 又永远不会 `_0x492e66`，就等于做了一个死循环，那永远也不会退出。
- 所以这里的代码也是一个暗装，没有任何用处。
- 我们直接将调用代码注释

![](https://img-blog.csdnimg.cn/a52d19e2885f4651ad5ee1f25eae7159.png)

#### 4.3 真正的开始运行

- 我们可以看到运行前后的代码比对

![](https://img-blog.csdnimg.cn/ce9d682507184e0ab696432b58eac193.png)

可以看出来左边的还原之后的效果还是不错的，已经比右边的清晰了不少了，那么我们接下来就要还原 `'Tuhel': _0x7818f1["pSzXT"], 'YUAtI': _0x7818f1["cVvhi"]` 这种代码了

### 5. 第二次还原

#### 5.1 我们对 `MemberExpression` 类型进行遍历

![](https://img-blog.csdnimg.cn/062455277c60420bada5ca79b16684d1.png)

可以看到我们需要操作的这个地方正好就是 `MemberExpression` 的 `type`

#### 5.2 代码

一些重复的代码，我就不复制上来了，可以参照前面的，比如引入包啥的

```
let filepath = "e:/desktop/2/main1.js";

fs.readFile(filepath,{"encoding":"utf-8"},function(err,data) {

        const ast = parser.parse(data);

        step(ast);

        let {code} = generator(ast);

        fs.writeFile("e:/desktop/2/main2.js",code,err => {

                if(err) throw err;

                console.log("保存成功");

        });

});

function step(ast) {

        traverse(ast,{

                 MemberExpression:HandleMemberExpression,

        })

}

        function HandleMemberExpression(path) {

                        if(path.get("object").isIdentifier() && t.isLiteral(path.node.property))

        {

                let objname = path.node.object.name;

                let propertyValue = path.node.property.value;

                let bindPath = path.scope.getBinding(objname);

                if(bindPath === undefined) return;

                try{

                        if(!t.isObjectExpression( bindPath.path.node.init))

                        return;

                }

                catch(e)

                {

                        console.log(e.message);

                        return ;

                }

                let propertys = bindPath.path.node.init.properties;

                if(propertys.length === 0)

                        return;

                try{

                        for (let i = 0;i
```

#### 5.3 第二次还原后的结果

我们可以看到右边的结果比左边又更清晰了不少，而且很多函数调用的参数上已经是明文字符串了

![](https://img-blog.csdnimg.cn/d16ad34660894e55b46e1cbd409320fe.png)

### 6. 第三次还原

此次还原的主要是是针对 `_0x7818f1["RniiM"](_0x509efb, _0x3a807c)` 这类函数进行还原

#### 6.1 对 `CallExpression` 进行遍历

![](https://img-blog.csdnimg.cn/ebb3a1df59e549ff83f7cda9df1277fc.png)

#### 6.2 代码
