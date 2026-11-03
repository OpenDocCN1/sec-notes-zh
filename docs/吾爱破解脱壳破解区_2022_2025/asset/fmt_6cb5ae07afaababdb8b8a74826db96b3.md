
[forum.php?mod=attachment&aid=MjcxMTU0MXwyOGJiZjNjZHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU0MXwyOGJiZjNjZHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:35 上传

再回到 ce 看看什么访问了，什么写了。对于每一个都可以看看`more infomation`，其中就能看到下图的（`dec`，`83`）这样关键信息（`dec` 代表数字减少 1，`83` 正好是我们要关注的魔法施放次数的位置，把这个画面截图，等一会要找这个地址，它就是魔法使用的子程序）：

![](https://attach.52pojie.cn/forum/202407/25/073604cukijiwivvkbbimz.png)

`12.png` (92.16 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjcxMTU0MnxmMWY2MzdkZXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU0MnxmMWY2MzdkZXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:36 上传

再看看什么写了这个地址的内容，这里不像什么访问了这个地址那样有很多内容，而只有一条，点击更多信息，显示有`mov`、`66`、`83`这样的关键信息，这正是我们要找的向魔法书中写魔法的子程序，也就是英雄学会魔法的子程序，也留下截图以记住这个地址，等会要用到（如下图）。

![](https://attach.52pojie.cn/forum/202407/25/073642a4ncb4tzbadcatlu.png)

`13.png` (170.77 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjcxMTU0NHwzZDU3ZTdhZHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU0NHwzZDU3ZTdhZHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:36 上传

## 三、用OllDBG把关键子程序复制出来

关了游戏，用OllyDBG打开《英雄无敌》1的主程序（一般就是`heroes.exe`），在反汇编区用鼠标右键选择`goto`，在对话框中填上刚刚截图的使用魔法的子程序地址，如下图：

![](https://attach.52pojie.cn/forum/202407/25/073742v66nilkuffhssnps.png)

`14.png` (323.21 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjcxMTU0NXxjZDU5MzcxOXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU0NXxjZDU5MzcxOXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:37 上传

点击确定后来到这个地址，将鼠标向上移动找到子程序的开始位置，如图：

![](https://attach.52pojie.cn/forum/202407/25/073836iibap8dbc8w8nng8.png)

`15.png` (95.09 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjcxMTU0OHxmOWY3YTY3MHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU0OHxmOWY3YTY3MHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:38 上传

选中第一行，再把鼠标向下移动到这个子程序的结尾，按住`shift`键，鼠标点击结尾，让这个子程序全部选中，再在鼠标右键选择复制到剪贴板。如图：

![](https://attach.52pojie.cn/forum/202407/25/073917u06u4ou4o2nto22z.png)

`16.png` (271.66 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjcxMTU1MHwwN2UxYWRmMXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU1MHwwN2UxYWRmMXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:39 上传

再用`notepad`建立一个新文件，把刚刚复制的内容贴到上面，如图：

![](https://attach.52pojie.cn/forum/202407/25/073949okkqxhilsgwwksvi.png)

`17.png` (150.96 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcxMTU1MXw4MTQ3YmFlZHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU1MXw4MTQ3YmFlZHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:39 上传

同样地再对什么写了这个地址的子程序相同操作：

![](https://attach.52pojie.cn/forum/202407/25/074024px8isor4hf453wb4.png)

`18.png` (456.53 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcxMTU1Mnw4ZjlmMmJiNHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU1Mnw4ZjlmMmJiNHwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:40 上传

把这个子程序也复制到`notepad`中：

![](https://attach.52pojie.cn/forum/202407/25/074059mepb2ftvjf8btl8s.png)

`19.png` (131.1 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjcxMTU1M3wxZGYwNWE3OXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMTU1M3wxZGYwNWE3OXwxNzkzNTcyMDE5fDE2ODA4NnwxOTQ3NjQz&nothumb=yes)

2024-7-25 07:40 上传

太好了，原来这两个子程序是连在一起的。下面我们要读懂这个子程序，只有读懂了它才好修改它，不是吗？

## 四、分析代码，搞懂程序的原理

下面是刚刚复制下来的使用魔法子程序的代码：

[Asm] 纯文本查看 复制代码
