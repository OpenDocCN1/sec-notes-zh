
```js
        "type": "SwitchCase",

        "test": {

            "type": "Literal",

            "value": index,

            "raw": index + "",

        },

        "consequent": res

    });

}
```

最后得到的语句就是转换之后的 `switch-case` 了，简单看下效果：

![](https://sergiojune.oss-cn-guangzhou.aliyuncs.com//img/image-20230807213755727.png)

## 6. 将多层 `switch` 转换成一层 `switch`

上面虽然看到舒服很多，调试也很舒服，但是还可以继续处理，就是直接将多层 `switch` 转成一层。这里的思路其实和上面的 `if` 转 `switch` 一致。

- 找到控制流

```js
 case Syntax.ForStatement:
    // 找到控制流 status 是上面 if 转 switch 遗留下来的 这里就不需要再判断别的了
    if (node.body.body.length === 2 && node.body.body[1].status){
        let status_name = node.init.declarations[0].id.name;
        let status_list = [ node.init.declarations[0].init.value];

    }
    return node;
```

- 收集整个控制流的所有状态变量

```js
estraverse.traverse(ast, {

   enter: function (node, parentNode){

       switch (node.type){

           case Syntax.AssignmentExpression:

               if (node.left.name === status_name && (node.right.type !== Syntax.UnaryExpression && node.right.operator !== "void")){

                   if (node.right.value === undefined)

                       throw "收集状态变量有新情况";

                   status_list.push(node.right.value);

               }

               break;

       }

   }

})
```

- 然后收集每个 `case` 对应的语句块，保存对应关系，然后 `case` 里面的语句块改掉，直接改成和上面的 `if` 转 `switch` 的形式

```js
estraverse.replace(node, {
   leave: function (node, parentNode){
       switch (node.type){
           case Syntax.SwitchCase:
               if (node.consequent[0].type === Syntax.SwitchStatement && status.indexOf(node.consequent[0].discriminant.name) > -1){
                   break;
               }
               if((parentNode.discriminant.type === Syntax.BinaryExpression && status.indexOf(parentNode.discriminant.left.name) 
```

这个的话需要注意一个控制流嵌套另一个控制流，他们的状态变量不一致的，不能收集错了。

- 模拟执行所有状态变量的值，整合成 `ast`

```js
while (true){

    index = status_list[num++]

    let eval_code = "var " + test.name + "=" + index + ";\n" + curr_code + "\n " + test.name + ";";

    let res_num = eval(eval_code);

    if(status_list && num > status_list.length)){

        break;  // 结束了

    }

    init_state.push(res_num);  // 记录语句 遇到重复的说明已经可以跳出了

    let res = state_table[res_num];

    new_node.cases.push({

        "type": "SwitchCase",

        "test": {

            "type": "Literal",

            "value": index,

            "raw": index + "",

        },

        "consequent": res

    });

}
```

简单整合在一起，最后就可以得到结果了

![](https://sergiojune.oss-cn-guangzhou.aliyuncs.com//img/image-20230807215435064.png)

## 6. 真正还原控制流

前面做的所有东西，其实都是还原控制流的预处理，这样做好处是，在还原控制流的时候写的规则会少很多，同时如果其他混淆有变化，但是控制流的形式不变，那后面更新了也可以只修改变的那部分，很方便，不同形式的控制流也可以这样做，比如将某数的控制流修改成某里的控制流形式，这样就可以用我接下来写的代码来还原，也很方便。

![](思路：

- 识别出控制流（同上）
- 收集所有 `case num` 以及他对应的代码块，同时把这个 `num` 作为属性添加给该块（同上）
- 找到初始化的状态变量，就是 `for` 循环里面的初始化变量
- 找到对应的代码块，然后解析出下一个块的 `num`，也就是下一个状态变量，并且将该 `num` 保存到数组中，代表已执行
- 遇到判断，就是该代码块的下一个状态变量有两个，
  - 先保存 `if` 的语句块，不执行
  - 将 `else` 里的当主分支继续执行

```js
if(last_node.type === Syntax.ExpressionStatement){
   cur_node = check_is_status(cur_node, len, status_name, case_table);
   next_node = cur_node[cur_node.length - 1];
} else if(last_node.type === Syntax.IfStatement){
   let last_node_seq_len = last_node.consequent.body.length;
   if (last_node.consequent.body[last_node_seq_len-1].type === Syntax.ExpressionStatement){
       last_node.consequent.body = check_is_status(last_node.consequent.body, last_node_seq_len, status_name, case_table);
       if_node = cur_node;  // 保留 if 语句 放到后面再整合
   }
   let last_node_alt_len = last_node.alternate.body.length;

   if (last_node.alternate.body[last_node_alt_len-1].type === Syntax.ExpressionStatement){
       last_node.alternate.body = check_is_status(last_node.alternate.body, last_node_alt_len, status_name, case_table);
       next_node = last_node.alternate.body[last_node.alternate.body.length-1];
   }
   if (if_node){
       // 头部插入，这样子后面处理的话是先处理内部在处理外面的 方便抽取相同块
       if_list.unshift(if_node);
       // console.log("保存 if 块：", if_node.num);
       if_node = undefined;
   }

}
pre_node = cur_node;
cur_node = next_node;
```

- 一直执行到代码结束，
  - 执行到 `return` 语句代表函数结束
  - 执行到状态变量为 `void 0` 代表结束（因为控制流的判断条件就是 `状态变量 !== void 0` 会一直执行）

```js
if (last_node.type === Syntax.ReturnStatement || escodegen.generate(last_node, {parse:n=>{return n;}}) === (status_name + " = void 0;")){

   break;

}
```

- 这里还没有真正执行完毕，因为还有 `if` 的语句块没执行，继续遍历 递归执行收集到的 `if` 语句块，回到 上面第四步
- 执行 `if` 语句块的话 跳出条件很关键，要不然很容易死循环，或者会执行到很多重复代码浪费时间，我这里跳出条件如下：
  - 先用第三步的数组判断目前的 语句块对应的 `num` 在不在数组
  - 不在的话证明百分百和主分支没有重复，继续往下执行
  - 如果存在，那就代表当前的语句需要合并到主分支了，同时需要结束当前 `if` 了，但是往下的情况处理也会有好多种
    - 先判断是不是单纯的 `while` 循环，若是循环语句 当前的 `num` 肯定是和 当前 `if` 语句的 `num` 值相同，代表的是统一代码块，如这种：

```js
while (U 
```

    - 判断是不是 `while true` 循环，这种有点复杂，需要用到当前的 `num` 和 其外部的 `if` 相继判断 `num` 是否一致，具体形式如下：

```js
while (true) {

...;

  if (ae = $ 
```

### 怎么判断是不是外部的 `if`

- 比如我在第一次运行的时候收集到一堆 `if` 列表，如：`a = [1,2,3];`
- 然后开始对 `if` 列表的第一个语句块 `1`，也就是索引为0，开始还原
- 那在这个 `if` 里的语句的外部 `if` 就是上面的 `a` 列表的从 索引为 `1` 往后的元素
- 遇到上面的 `while true` 循环的情况就是 在第一个 `if` 语句块 `1` 里面遇到了 `if` 语句块 `2`，他在外部 `if` 列表里面，所以这个是 `while true` 循环，同时他的 `else` 部分 就是主分支了，同时 在 `else` 部分改为 `break` 语句，因为这个是跳出循环的条件，如上
- 判断是不是在这个 `if` 语句一进来就存在重复语句，若是，那肯定这个 `if` 语句块是为空的，直接跳出（前提是先判断是不是 `while` 和 `while true` 循环）
- 直接在已运行的状态列表获取这个 `num` 对应的语句块即可结束，不过需要深度拷贝，不可以直接引用
- 为什么可以这样？在之前每运行一个语句块，都会去保存这个语句块的对应关系，比如：
  - 首先第一个语句块为 `5`，保存到已运行列表以及对应语句块
  - 然后下一个语句块为 `10`，继续保存
  - 当开始运行 `if` 语句时候，若下一个语句块是 `5`，那我可以直接去已运行列表获取 `5` 对应的语句块，因为他记录了他的之后的语句块的情况，而我们不需要继续去模拟运行，浪费时间。
- 为什么需要深度拷贝呢？因为在合并分支的时候会破坏语句块，比如下面这种情况

```js
if (Z) {
  I = y(Z, A[1]);
  if (I) {
      S.push(I[Ao]);
  } else {
      S.push(0); // 5
  }
} else {
  S.push(0);  // 5
}
...;
```

比如在运行完 `z` 判断的 `else` 部分时候，记录了所有的语句块，若 `S.push(0);` 是语句块 `5`；然后再去运行 `z` 判断的 `if` 部分，首先会遇到 `I` 判断，但是我们直接忽略 `if` 部分（会先记录），直接运行 `else` 部分，然后又运行到了 语句块 `5`，对应的就是 `S.push(0);`  我们在判断他不是 `while`、不是 `while true`，同时也不是空语句的时候，就可以直接在 已运行列表获取 这个 `5` 对应的语句块了 ，因为他们后面的语句肯定是一样的。

接下来是重点，我们在结束 `I` 判断 `if` 语句的时候，是需要合并分支的，怎么合并？那肯定是需要裁剪的，这个时候语句块 `5` 就会被裁剪，如果上面取语句块 `5` 的时候没有采用深拷贝，就会影响到后面的句子。

- 当处理完当前 `if` 语句的时候，需要处理语句，比如判断到是 `while` 循环、`while true` 循环的处理，同时若不是以上这两种就需要合并分支

```js
function handle_branch(last_body, cut_node, is_while, is_do_while, prev_body){

    let if_node = cut_node[cut_node.while_true?cut_node.length-2: cut_node.length-1];

    // 判断是不是遇到循环了

    let last_node = last_body[last_body.length-1];

    let seq_body = if_node.consequent.body;

    let alt_body = if_node.alternate.body;

    if (is_while && cut_node.length === 1){

        // 这个是直接用if条件做循环

        cut_node[cut_node.length-1] = if_to_while(if_node.test, seq_body, alt_body);

        cut_node[cut_node.length-1].num = cut_node.num;

        return "while";

    }else if(is_while || is_do_while){

        let temp_seq_body;

        // 这个直接是while true 循环

        if(last_body.num === cut_node.num){  // 这个是跳出条件和当前需要还原的if条件一致

            temp_seq_body = cut_node;

            cut_node = [];

        }else{

            remove_while_node(cut_node, prev_body);  // 这个删除节点直接把 cut_node 在 ast 上删除了 所以后面要补上 要不然怎么修改都不会对原节点有影响

            // 这个跳出条件是前面 不一致 cut_node顺便清空 因为把内容都给了 prev_body

            prev_body.splice(prev_body.length, 0, cut_node.splice(0, cut_node.length));

            prev_body[prev_body.length-1].num = cut_node.num;

            temp_seq_body = prev_body;

            prev_body.while_true = true;

        }

        let alt_body = if_node.alternate.body;  // 这部分是while true 下面的语句，并且else需要填上break

        if_node.alternate.body = [

            {

                "type": "BreakStatement",

                "label": null,

            }

        ];

        let true_node = {

            "type": "Literal",

            "value": true,

            "raw": "true"

        };

        cut_node = if_to_while(true_node, temp_seq_body, cut_node);

        cut_node.splice(cut_node.length, 0, ...alt_body);

        return is_while?"while":"while_true"

    }else{

        // 这个再遍历if else分支 看看有没有重复的

        while (true){

            if (!(seq_body[seq_body.length-1] instanceof Array))

                break;
```
