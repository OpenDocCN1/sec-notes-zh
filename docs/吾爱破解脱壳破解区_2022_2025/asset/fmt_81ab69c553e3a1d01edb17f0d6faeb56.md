
# 逆向分析记录

image 14.png (21.4 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcwMnxiYzhkNzMyMHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwMnxiYzhkNzMyMHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

但是双击`unk_140005040`查看其初始值，并不能直接获取到什么有效信息

![](https://attach.52pojie.cn/forum/202409/28/213818bj8581l8wm5zm5j1.png)

image 15.png (37.6 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcwM3xmNmJhMTVjY3wxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwM3xmNmJhMTVjY3wxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

我们之前推测`v9`可能用来存储函数，如果真的是这样，那么`unk_140005040`应该就是那个函数。我们按下`c`键，IDA将把这些数据转化为汇编码

![](https://attach.52pojie.cn/forum/202409/28/213820rc6vqggpcg67sbv7.png)

image 16.png (194.88 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyNTcwNHxlMzEzZjExOHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwNHxlMzEzZjExOHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

`14000504A`及之后的代码挺像一回事的，但是前部分作为函数的话缺少几个push，我们就先回到`Pseudocode-A`，看看之后有没有对函数进行其他处理。

分析代码逻辑，会发现第40行将`v9`的值赋值给了`v6`，而第51-53，65-67行，有使用`v6`来对函数所在内存的开头部分以及其他一些点重新赋值（使用地址定位来进行的小范围赋值一般称为patch）

![](https://attach.52pojie.cn/forum/202409/28/213823fdm8z7jm758m77jd.png)

image 17.png (86.28 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcwNXw4MGQxODU2MnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwNXw4MGQxODU2MnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

我们想要查看重新赋值后的函数，就要进入动态分析

## 动态分析

点击菜单栏的`Debugger`，选中`Select debugger`

![](https://attach.52pojie.cn/forum/202409/28/213825lr22vc26vtvrly2f.png)

image 18.png (41.61 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyNTcwNnxkNWU4MDJjYnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwNnxkNWU4MDJjYnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

在弹出的窗口中选择`Local Windows debugger`，点击OK

![](https://attach.52pojie.cn/forum/202409/28/213827b9b02dwdt9d9z04y.png)

image 19.png (57.04 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcwN3xhMGFiY2E2YnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwN3xhMGFiY2E2YnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

这样我们就设置好了调试器，接下来就是打断点

由于我们希望看到patch后的函数，因此直接在patch后的下一条指令打下断点即可

![](https://attach.52pojie.cn/forum/202409/28/213829uzenbwecii6mypm7.png)

image 20.png (62.4 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcwOHw5NmU3ZWNjNXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwOHw5NmU3ZWNjNXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

再次点击菜单栏的`Debugger`, 会发现展开内容变化了，选择`Start process`，即开始调试。弹出的窗口全点yes或ok。

![](https://attach.52pojie.cn/forum/202409/28/213831bl9extu1wmb8ttbe.png)

image 21.png (54.24 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcwOXxlYTAxZjQzYXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcwOXxlYTAxZjQzYXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

代码会停止在下断点处

这时我们就可以查看patch后的代码，查看方式为

鼠标悬浮在`v6`上，查看`v6`的值

![](https://attach.52pojie.cn/forum/202409/28/213833gk34777kvbqk3v74.png)

image 22.png (35.74 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcxMHw5MzZhNmQ2MXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcxMHw5MzZhNmQ2MXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

我们就得到了`v6`的值为`0x17DD94E0000`（不固定，每一次调试的具体值都可能不同），也就是说`0x17DD94E0000`指向代码的开始

按下`g`键，弹出地址跳转窗口，输入`v6`的值，点击ok。

![](https://attach.52pojie.cn/forum/202409/28/213835s9zy4ccjcbclo9hj.png)

image 23.png (23.22 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcxMXxiMDVlMmEyM3wxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcxMXxiMDVlMmEyM3wxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

跳转到新界面后按下`c`键，将数据转化为汇编代码

![](https://attach.52pojie.cn/forum/202409/28/213837y4249jts9z44ie5o.png)

image 24.png (87.15 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcxMnxlMzhkM2EzMnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcxMnxlMzhkM2EzMnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

可以看到之前的nop都变成具体的代码了。想要看这些汇编代码对于的c语言代码，则右键函数的起始位置，即`17DD94E0000`，在菜单中选择`create function`

![](https://attach.52pojie.cn/forum/202409/28/213839y0kayikybmkyou7x.png)

image 25.png (84.81 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcxM3xlOTkzNTZmNnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcxM3xlOTkzNTZmNnwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

点击后可以注意到代码再次产生了变化

![](https://attach.52pojie.cn/forum/202409/28/213841pye607bvuvi9fmyk.png)

image 26.png (83.35 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTcxNHwyNjViM2VmMHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTcxNHwyNjViM2VmMHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:38 上传

这时选中函数名，按下`tab`键，就将跳转到c语言代码界面。

![](https://attach.52pojie.cn/forum/202409/28/213735p7js5cbfmfsscnsx.png)

image 27.png (38.8 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTY4M3xkODBjM2ZhNHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTY4M3xkODBjM2ZhNHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:37 上传

这段代码有一些赋值以及看不明白的函数调用，搞不清楚，于是先回到`winMain`里的那个名为`sub_140001090`的函数。按下窗口左上角的左向箭头，返回上一个界面

![](https://attach.52pojie.cn/forum/202409/28/213738uffgvbfy2ahhuth3.png)

image 28.png (125.13 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTY4NHw3NjlkMTg4OHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTY4NHw3NjlkMTg4OHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:37 上传

回到`sub_140001090`，重新梳理程序逻辑，发现第一次调用`v6`指向的函数为第64行（`__fastcall*`即意为将之后的内存作为函数调用），会发现这里并不是调用`v6`的起始位置，而是还有一个`1616`（十进制，有0x前缀才为16进制）的偏移

![](https://attach.52pojie.cn/forum/202409/28/213740wrbjl3jb2ojh666j.png)

image 29.png (59.27 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTY4NXwyOTIwNDA3MHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTY4NXwyOTIwNDA3MHwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:37 上传

这意味着`v6`指向的一大段内存中可能不止一个函数，因此我们再次按下`g`键，看一看`v6`偏移`1616`的函数。`1616`转为十六进制为`0x650`，因此函数的地址为：`v6+650`，以我这次运行的`v6=0x17DD94E0000`，加上`0x650`的偏移，就是 `0x17DD94E0650`。

按下`g`键，输入地址，按下`c`键，转化为汇编码，右键函数起始，`create function`，选中函数名，按下`tab`，以操作`0x17DD94E0000`的步骤，操作`0x17DD94E0650`，查看其c语言代码

![](https://attach.52pojie.cn/forum/202409/28/213742hqqhqlq31mvq34l4.png)

image 30.png (68.13 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyNTY4Nnw0OGRjNTM0NXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyNTY4Nnw0OGRjNTM0NXwxNzkzNTcxNzM5fDE2ODA4NnwxOTY4NDg5&nothumb=yes)

2024-9-28 21:37 上传

结果还是莫名其妙的赋值以及函数调用

但在第111行的字符串中，看到了`position`，有看到了`color`，由此推测图像绘制的核心代码大概就存在于`v6`指向的那一大块代码里。

![](https://attach.52pojie.cn/forum/202409/28/213744rrrhiwb0aarb9i2i.png)

image 31.png (46.17 KB, 下载次数: 1)
