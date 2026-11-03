
# [CTF] [2019红帽杯]Snake

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1609424
- 抓取时间: 2026-11-01T23:31:49

## [2019红帽杯]Snake

好久没上线了,长春疫情但是我的设备都在寝室,封寝来的太突然,好久没能做两道逆向,但是要备战今年国赛了.

看到大师傅BUU上一个月刷了7000分,顺序刷题不管分数, 我觉得我也得有这种精神, 一道题磕不动就继续磕!

题目本身很简单, 或者说是不难吧, 但是之前没怎么碰过U3D,也算是接触一下.

今年国赛队伍的web和misc都走掉了, 估计也碰不到邬江兴院士的可信计算, 估计国赛要三等扑街了...

这题看上去应该是一道.NET U3D, 先玩了几百分, 感觉没啥用, 没有遇到过这种类型的题, 先放在 `Dnspy` 里看一眼

![](https://tianyu.xin/usr/uploads/2022/03/2056209337.png)

失策了,它不是, 那翻翻DLL

![](https://tianyu.xin/usr/uploads/2022/03/3115171165.png)

在`./Manage`下

![](https://tianyu.xin/usr/uploads/2022/03/2052455246.png)

把`CSharpAssembly`扔进`Dnspy`, 尝试找找关键函数, 整个`MainUIController`实现了游戏的启动升级之类的, 但是直到无尽模式也看不到flag相关的信息

去找`GameObject`, 发现用到了`interface`这个dll, 反编译之

![](https://tianyu.xin/usr/uploads/2022/03/2932353816.png)

![](https://tianyu.xin/usr/uploads/2022/03/2029013406.png)

直接去到`Gameobject`里, 看看怎么达成这个you win

![](https://tianyu.xin/usr/uploads/2022/03/558434197.png)

感觉这个地方是类似某种高精度数据结构, 我讨厌SSE, 第一个参数是`Dst`, 第三个参数是数字长度

`A9D0`这个函数就是一个数字转字符, 但是两个涉及高精度的函数还是很麻烦的. 这时候我疑惑输入在哪里.

整个函数只有一个参数传进来, 并且也是唯一参与后面运算的. 当这个参数在0~100中才能走到win, 否则有一大堆`SKTEDG`什么的东西, 还有`7777`, 玩梗大师

那么根据猜测`v74`里面有一个1399...的大整数, `v64`里第二个大整数, `v71`里是传入的参数

![](https://tianyu.xin/usr/uploads/2022/03/4091683753.png)

它们三个被扔进了`6250`这个函数, 结果应该是扔到了`v67`里, `6250`这个函数实在不是人能分析的, 现在摆在我面前可行的路至少有两条, 一是爆破, 因为只有100个值,不会时间太长, 二是动调, 看看这个函数是干啥的, 但是后面还有一个函数`AAB0`需要动调, 动调本身也是个猜的过程, 我感觉时间成本太高, 还是爆破吧...

```
import ctypes

for i in range(0, 100) :
    dll = ctypes.cdll.LoadLibrary(".\Snake_Data\Plugins\Interface.dll")
    dll.GameObject(i)
# flag{Ch4rp_W1th_R$@}
```
