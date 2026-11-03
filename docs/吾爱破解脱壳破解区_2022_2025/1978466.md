
# [CTF] 2024 网鼎杯玄武组Reverse1 题解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1978466
- 抓取时间: 2026-11-01T22:21:26

比赛结束后一小时才做出来这道题，很遗憾。

首先将题目`main`文件拖入IDA中分析，分析`main`函数逻辑，该程序具有插入字符串（最多19个字符，最多进行15次插入）和搜索（也是最多进行15次）两个功能。

![image.png](https://attach.52pojie.cn/forum/202411/05/002128cpncnnucbnbjiu4f.png)

`main`函数中最先调用`preprocessing`函数，将本地的`flag.txt`文件读入到内存中，地址`0x0064040`处存储的就是flag，所以我们需要想办法读到这个内存地址。

![image.png](https://attach.52pojie.cn/forum/202411/05/002107irshyhy5yyyr1m5y.png)

接着分析负责插入功能的`insert`函数，发现插入用户输入的字符串的过程实质是构造一棵二叉树，且要求左子节点的值比右子节点的值大。

![image.png](https://attach.52pojie.cn/forum/202411/05/002053upc0ll81w1lp3p08.png)

![image.png](https://attach.52pojie.cn/forum/202411/05/000624j2pnb5dd6aa3rpqp.png)

分析`search`函数，发现是遍历二叉树的过程。

![image.png](https://attach.52pojie.cn/forum/202411/05/002209gvpaz4vguassj1ml.png)

我们可以发现`search`后，程序会`puts`打印找到的字符串。

![image.png](https://attach.52pojie.cn/forum/202411/05/002154sy2a9cr44fn4i94f.png)

全局变量`a`在内存中的地址为`0x4040`。当`v6 = 0x4000`时，`&a + 24 * v6`等于`0x0064040`，也就是flag处。要想`v6`为`0x4000`需要输入一组数据，要求数值逐渐减小，并且小于`0x6D7564206D612069`，只有这样`insert`函数中的变量`a1`才会以2为底的指数增长，直到达到`0x2000`。我们可以将问题转化为，需要构造一棵只有左孩子节点的二叉树。

只要输入13次数据，就可以让`0x2000`处的节点填进数据，这样就可以和`0x4000`处的flag连到一起（作为左孩子节点）。

`insert`函数中，利用进程id和系统时间来生成随机数，并在用户每次插入字符串使用随机数进行`&`操作，这会导致可能随机发生树节点发生换位。这里挺阴间的，你即使知道正确的插入数据的序列，也需要一定的运气才能获得flag（每次输入时随机数计算都不触发换位的前提下，13轮输入，说实在得是欧皇才能快速成功）。

为了方便验证序列是正确的，我将`a1>1`的判断跳转修改为`jmp`，直接跳过换位操作。`patch`这部分代码只是避开运气的部分（强行欧皇），不影响程序的整体逻辑。

![image.png](https://attach.52pojie.cn/forum/202411/05/002408fu6f6qccup9z18dk.png)

![image.png](https://attach.52pojie.cn/forum/202411/05/002336wvav1didvav87vl1.png)

在本地运行修改后的`main`文件，说明一下这里测试的如`00000008aaaaaaaaaaa`在调试时，手动修改为`\x00\x00\x00\x00\x00\x00\x00\x5Faaaaaaaaaaa`（因为无法输入不可打印的ascii字符），完成13轮输入后，查询`wd`，成功获取到flag。

![image.png](https://attach.52pojie.cn/forum/202411/05/002248ojb5mbn55ji5jnup.png)

![image.png](https://attach.52pojie.cn/forum/202411/05/002304bnmoqoydjuryqj8j.png)

贴一下解题脚本

![image.png](https://attach.52pojie.cn/forum/202411/05/001027mc93sb9s39qb12el.png)

[![rar.gif](https://static.52pojie.cn/static/image/filetype/rar.gif)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMzQ4NXwwMGRhYTE5NnwxNzkzNTcxNjg0fDE2ODA4NnwxOTc4NDY2)

(4.41 KB, 下载次数: 10)

2024-11-6 11:42 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB
