
```
    };

}

bool HelloWorld::runOnFunction(Function &F){

    //todo 对函数的分析或修改代码

    outs()   X("hlw","对Pass的描述");  //注册该Pass
```

`test.sh`：编译 LLVM Pass 并对 `Test` 文件夹中的代码进行测试(复制时把我的注释去掉)。

在vscode控制台输入：

```
chmod 777 test.sh
```

然后运行：

```
./test.sh
```

笔者在此处遇到一个问题，第一次编译并不会在`Build`文件夹下产生`LLVMObfuscator.so`文件，需要进入`Transforms`文件夹手动进行复制到`Build`文件夹进行第二次编译。

```
cd ./Build
cmake ../Transforms               //对transforms的项目进行编译，得到编译后的`.so`文件
make
cd ../Test
clang -S -emit-llvm TestProgram.cpp -o TestProgram.ll       //clang将源代码转换为中间代码
opt -load ../Build/LLVMObfuscator.so -hlw -S TestProgram.ll -o TestProgram_hlw.ll  ////opt加载so文件，用hlw pass进行优化
clang TestProgram_hlw.ll -o TestProgram_hlw     //将优化后的中间代码编译为可执行文件
./TestProgram_hlw                         //运行可执行文件
```

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306142141733.png)

实现了对样本程序中的每个函数进行扫描，并输出了`"Hello,"+函数名`。

## CMake学习

[https://github.com/ttroy50/cmake-examples](https://github.com/ttroy50/cmake-examples)

## LLVM IR概述

LLVM编译过程回顾：

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306142206295.png)

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306142307005.png)

### 什么是LLVM IR?

- LLVM IR 是一门低级编程语言，语法类似于汇编

- 任何高级编程语言（如C++）都可以用 LLVM IR 表示

- 基于 LLVM IR 可以很方便地进行代码优化(任何编程语言都能统一转换为LLVM IR)

### LLVM IR的两种表示方法

LLVM IR 共有两种表示方法：

- 人类可以阅读的文本形式，文件后缀为`.ll`

- 易于机器处理的二进制格式，文件后缀为`.bc`

通过LLVM自带的工具，该两种表示形式也可以互相转换。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151126144.png)

### LLVM IR结构

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151126720.png)

源代码被LLVM IR编译后，有如下结构：

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151128610.png)

#### 模块 `Module`

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151130875.png)

- 一个源代码文件对应 LLVM IR 中的一个模块。

- 头部信息包含程序的目标平台，如X86、ARM等等，和一些其他信息。

- 全局符号包含全局变量、函数的定义与声明。

#### 函数 `Function`

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151130767.png)

- LLVM IR 中的函数表示源代码中的某个函数。

- 参数，顾名思义为函数的参数。

- 一个函数由若干基本块组成，其中函数最先执行的基本块为入口块。

#### 基本块 `BasicBlock`

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151131326.png)

- 一个基本块由若干个指令和标签组成。

- 正常情况下，基本块的最后一条指令为跳转指令（`br` 或 `switch`），或返回指令（`retn`），也叫作终结指令(Terminator Instruction)。

- 其他指令：`PHI` 指令是一种特殊的指令。

LLVM IR和IDA反汇编的结构比较相似。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151139963.png)

基于 LLVM 的混淆，通常是以函数或者比函数更小的单位为基本单位进行混淆的，我们通常更关心函数和基本块这两个结构。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151140432.png)

#### 了解 LLVM IR 的结构是我们学习代码混淆的基础：

- 以函数为基本单位的混淆：控制流平坦化。

- 以基本块基本单位的混淆：虚假控制流。

- 以指令为基本单位的混淆：指令替代。

## LLVM IR指令

总共有六大类指令：

- 终结指令 Terminator Instructions

- 二元运算 Binary Operations

- 按位二元运算 Bitwise Binary Operations

- 内存访问和寻址操作 Memory Access and Addressing Operations

- 类型转换操作 Conversion Operations

- 其他操作 Other Operations

### 终结指令

基本块的最后一个指令，通常为跳转指令或返回指令。跳转到下一个基本快进行执行或从当前执行的函数中返回。

#### `ret`指令

`return`

函数的返回指令，对应 C/C++ 中的`return`。

分为有返回值的`ret`指令和无返回值的`ret`指令。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151406265.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151408324.png)

#### `br`指令

`branch`

跳转指令，分为条件分支和非条件分支。

- `br` 是”分支”的英文`branch`的缩写，分为非条件分支和条件分支，对应 C/C++ 的`if`语句。

- 无条件分支类似于x86汇编中的`jmp`指令，条件分支类似于x86汇编中的`jnz`, `je`等条件跳转指令

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151410426.png)

```
br il, label , label    //i1为条件分支指定的热条件， i1是1位整数，可以理解成一个bool类型，若i1为true，则跳转到第一个基本块；否则跳转到第二个基本块

br label    //直接跳转到目的基本块
```

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151410021.png)

```
Test：
    %cond = icmp eq i32 %a, %b                     //比较ab变量是否相等，返回true or flase
    br i1 %cond, label %IfEqual, label %IfUnequal  //根据%cond判断跳转到哪个基本快
IfEqual:
    ret i32 1
IfUnequal:
    ret i32 0
```

#### 比较指令

比较指令，通常与`br`指令一起出现。

- 在x86汇编中，条件跳转指令(`jnz`, `je`等)通常与比较指令`cmp`, `test`等一起出现。

- 在 LLVM IR 中也有这样一类指令，他们通常与条件分支指令`br`一起出现。

##### `icmp`指令

整数或指针的比较指令。

- 条件`cond`可以是`eq`（相等）, `ne`（不相等）, `ugt`（无符号大于unsigned greater than）等等。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151426284.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151426459.png)

##### `fcmp`指令

浮点数的比较指令。

- 条件`cond`可以是`oeq`（ordered and equal）, `ueq`（unordered or equal）, `false`（必定不成立）等等。

- ordered的意思是，两个操作数都不能为`NAN`

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151429215.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151429048.png)

```
fcmp oeq   //比较两个浮点数是否相等

fcmp one   //比较两个浮点数是否不相等     ordered and not equal
```

#### `switch`指令

分支指令，可看做是`br`指令的升级版。

- 支持的分支更多，但使用也更复杂。对应 C/C++ 中的`switch`。

- 根据`switch`变量值`value`进行跳转，若`value`没有对应值，则执行默认的基本块

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151436584.png)

```
switch  ， label  [ , label ...]
```

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151444743.png)

### 二元运算指令

#### `add`指令

整数加法指令

- 整数加法指令，对应 C/C++ 中的“`+`”操作符，类似x86汇编中的`add`指令。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151446829.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151446722.png)

#### `sub`指令

整数减法指令

- 整数减法指令，对应 C/C++ 中的“`-`”操作符，类似x86汇编中的`sub`指令。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151447691.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151447138.png)

#### `mul`指令

整数乘法指令

- 对应 C/C++ 中的“`*`”操作符，类似x86汇编中的`mul`指令。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151448557.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151507274.png)

#### `udiv`指令

无符号整数除法指令

- 对应 C/C++ 中的“`/`”操作符。如果存在`exact`关键字，且`op1`不是`op2`的倍数，就会出现错误。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151528052.png)

加入`exact`时，需要精确保证`op1`是`op2`的倍数，指令才会正常执行。

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151533692.png)

#### `sdiv`指令

有符号整数除法指令

- 对应 C/C++ 中的“`/`”操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151534076.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151535087.png)

#### `urem`指令

无符号整数取余指令

- 对应 C/C++ 中的“`%`”操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151536549.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151537131.png)

#### `srem`指令

有符号整数取余指令

- 对应 C/C++ 中的“`%`”操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151539086.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151540994.png)

### 按位二元运算指令

逻辑移位统一当成无符号数

算数右移按照原来符号数来决定正负

#### `shl`指令

整数左移指令

- 对应 C/C++ 中的“`<<`”操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151547641.png)

```
 = shl  ,     //op1左移op2位
```

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151548130.png)

#### `lshr`指令

整数逻辑右移指令

- 对应 C/C++ 中的“`>>`”操作符，右移指定位数后在左侧补0。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151551736.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151551882.png)

`-2`当做无符号数来处理，右移得到16进制数   无符号的右移

#### `ashr`指令

整数算术右移指令

- 右移指定位数后在左侧补符号位（负数的符号位为1，正数的符号位为0）。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151623894.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151623986.png)

`-2`右移1：带符号的右移

逻辑移位：无符号右移

算数移位：带符号右移

#### `and`指令

整数按位与运算指令

- 对应 C/C++ 中的“`&`”操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151627531.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151627343.png)

#### `or`指令

整数按位或运算指令

- 对应 C/C++ 中的“`|`”操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151628387.png)

实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151628837.png)

#### `xor`指令
