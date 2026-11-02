
# ID 值生成分析

发现这个字串确实是根据 `1234` 来定义的 ID 值，同时也可以确认一下这个 ID 值到底是不是固定的

![](https://img.vinua.cn/images/OhzoX.png)

可以看到输入不同的 `username`，ID 值应该也是不同的

因此具体的密码会根据 `username` 计算的，需要再次进入核心算法查看一下

不过经过刚才的一通分析，了解到：核心算法应该是根据

`Yo15z9atB8gZJRhpi0dLfCSA3wVFEyuHKOrTWG4mlXvPeU2j67NbMxsknQqcI`

这个字串，来确定符号下标，一共要找 16 次，内部有 4 个为一个小循环，然后还有 4 次大循环。

![](https://img.vinua.cn/images/OhPaY.png)

## 寄存器分析

在重新分析核心算法时，要紧盯这几个寄存器，明白这几个寄存器的用处

`rsi` 专门用来存放序号位置，`r8` 用来存放取出来的一个 `username` 字符值，`rbp` 是存放键值表

![](https://img.vinua.cn/images/OhQJj.png)

继续往下分析，在这里发现有一个熟悉的字符出现了，就是 L ,上次密码的第一个字母 (`LRZFZCEJELIGIZVQ`) 而这个 L 就是通过上边一条指令获取到的，所以应该目前该关注的就是上一条指令

![](https://img.vinua.cn/images/OheCp.png)

跟进上一条指令的内存

![](https://img.vinua.cn/images/OhiNV.png)

发现 L 就在这个地方，下边还有那一开始的字串

往上滑一下，还发现了其他字符

![](https://img.vinua.cn/images/OhA00.png)

`EYIPVMTKFXJSGWQOADRNCBLHZU`

将上边这一串字符也摘下来，发现这是 26 个长度的字符串

然后再结合 IDA 里那个取余的数字 “26”，可以猜测到就是从这26个字符中选择一个字符，然后填入到 ID 中

## 运算推导

上下查看，发现 `v9` 第一次是 0 ，`v6` 是从 61个字符串中与 `username` 字符对比来的序号值，也就是1的位置 3

所以就是 3 而 L 在这 26 个字串 `EYIPVMTKFXJSGWQOADRNCBLHZU` 的位置正好是第 23 位，也就是 [22] 的位置，所以运算应该就是这样：

![](https://img.vinua.cn/images/Oh6yR.png)

先从 61 字符串取序号位置 `v6` 然后 `v6 + v9`，然后左移 16 位，然后对 26 取余

，计算获得字符下标，获取目标字符。

![](https://img.vinua.cn/images/OhF2w.png)

![](https://img.vinua.cn/images/OhU32.png)

![](https://img.vinua.cn/images/Oh0iA.png)

而其他三个也是有其他额外数值增加的，`0x539`，`0xA72`，`0xFAB`

从 `x64Debug` 中也能看到，所以运算方法就知道了，但是如果是 `1234 1234 1234 1234` 排列，那么下一组值在运算中

也应该是一样的，但是进行下一个大循环，`1234` 取值是不同的，经过再次查看发现到

![](https://img.vinua.cn/images/OhEoW.png)

![](https://img.vinua.cn/images/OhjvQ.png)

`v9` 每次大循环都是要 + `0x14E4` 的，这个 `x64Debug` 中也能看到，

所以现在的情况就很明朗了

具体的 ID 运算数值：

就是先从 61 字串中找到序号位置，然后 +`v9`，(`v9` 每个大循环会自增)，取得的值左移 16 位之后对 26 取余，然后

从 26 字串中取一个字符，放入到结果中。

知道这些之后，我在下方提供我注册机代码的编写。其实在复测过程中发现，`123abfD` 一开始是计算错误的，因为在

那 61 个字符之中是没有 D 这个字符的，所以如果有查询不到的字符，就返回 0 ，这个在 `x64Debug` 中能自己追踪到。

(第一次写这些记录，有的地方可能会晦涩一点，大佬们都可以提出来，我也会努力更正)

```
#include
#include
#include
#define LEN 16
#define NUM1 0x539
#define NUM2 0xA72
#define NUM3 0xFAB
#define TOTAL 0x14E4

char gKey[] = "Yo15z9atB8gZJRhpi0dLfCSA3wVFEyuHKOrTWG4mlXvPeU2j67NbMxsknQqcI";
char gKey2[] = "EYIPVMTKFXJSGWQOADRNCBLHZU";
char gResult[16] = "";

// 用于下标查找
int GetIndex(char ch)
{
char cTemp = 0;
int nLen = sizeof(gKey);
for (sizet i = 0; i  LEN || UserLen 
```
