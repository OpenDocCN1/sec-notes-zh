
### 整数按位异或运算指令

- 对应 C/C++ 中的 `^` 操作符。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151629469.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151629724.png)

变量和 `-1` 进行异或，相当于把变量按位取反

## 内存访问和寻址操作指令(重点)

### 静态单赋值

- 在编译器设计中，静态单赋值（Static Single Assignment, SSA），是 IR 的一种属性。
- 简单来说，SSA 的特点是：在程序中一个变量仅能有一条赋值语句。
- LLVM IR 正是基于静态单赋值原则设计的。

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151921331.png)

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151921072.png)

引例：

- 假设 C++ 也是基于静态单赋值原则的（即一个变量只能被赋值一次），要怎样修改这个 for 循环，使其符合 SSA 原则？

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151930642.png)

- 以下是一种实现方式，也是 LLVM IR 采取的实现方式。在 LLVM IR 中也有类似 `malloc` 和指针操作的指令：

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306151930137.png)

只有第一次给指针 `i` 进行了赋值，后面都是对指针指向的区域进行赋值

### `alloca` 指令

内存分配指令

- 在栈中分配一块空间并获得指向该空间的指针，类似于 C/C++ 中的 `malloc` 函数。(区别于 C 语言 `malloc` 函数在堆中分配空间)

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152013876.png)

`type` 指定了分配什么类型的空间，`NumElements` 指定了分配空间的个数

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152013703.png)

### `store` 指令

内存存储指令

- 向指针指向的内存中存储数据，类似于 C/C++ 中的指针解引用后的赋值操作。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152026576.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152026784.png)

### `load` 指令

内存读取指令

- 从指针指向的内存中读取数据，类似于 C/C++ 中的指针解引用操作。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152029949.png)

#### 实例！

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152030503.png)

分配 4 字节内存

存储 3

读取 `ptr` 指向的内存，即 3

## 类型转换操作指令

### `trunc ... to` 指令

截断指令

- 将一种类型的变量截断为另一种类型的变量。对应 C/C++ 中大类型向小类型的强制转换（比如 `long` 强转 `int`）

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152033068.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152033054.png)

```
%X = trunc i32 257 to i8      //截断低8位，十进制为1
%Y = trunc i32 123 to i1      //转换为i1类型，即bool类型 保留最低一位，即1或true
```

### `zext ... to` 指令

零拓展（Zero Extend）指令

将一种类型的变量拓展为另一种类型的变量，高位补 0。对应 C/C++ 中小类型向大类型的强制转换（比如 `int` 强转 `long`）

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152043925.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152043403.png)

### `sext .. to` 指令

符号位拓展（Sign Extend）指令

通过复制符号位（最高位）将一种类型的变量拓展为另一种类型的变量。

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152044650.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152044218.png)

## 其他操作指令

不太好分类，又比较常见的指令

### `phi` 指令

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152047639.png)

通过引入 Φ 函数来解决这个问题，Φ 函数的值由前驱块决定，这里的 Φ 函数对应 LLVM IR 中的 `phi` 指令：

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152048108.png)

- `phi` 指令可以看做是为了解决 SSA 一个变量只能被赋值一次而引起的问题衍生出的指令。
- `phi` 指令的计算结果由 `phi` 指令所在的基本块的前驱块确定

以下是一个用 `phi` 指令实现 for 循环的实例：

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152051954.png)

### `select` 指令

- `select` 指令类似于 C/C++ 中的三元运算符 `... ? ... : ...`

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152052639.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152052518.png)

### `call` 指令

- `call` 指令用来调用某个函数，对应 C/C++ 中的函数调用，与 x86 汇编中的 `call` 指令类似

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152052791.png)

#### 实例

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202306152053186.png)

# 一个简单的 LLVM IR 阅读与分析(了解结构)

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202307021135314.png)

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202307021137269.png)

![](https://image-1311319331.cos.ap-beijing.myqcloud.com/image/202307021138579.png)

## LLVM IR for 循环分析

for 循环的实现方法

- 内存存取指令
- `phi`

# Reference

看雪 《LLVM与代码混淆技术》
