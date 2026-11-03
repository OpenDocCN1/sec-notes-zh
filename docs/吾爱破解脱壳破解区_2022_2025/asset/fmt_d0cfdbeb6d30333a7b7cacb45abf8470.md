
## 经验与教训

出来的，血的教训就是在那狂点步进几个钟头，别问为啥，为啥就是二百五呗，原因就是游戏不像其他的软件，其他的软件是注册一次，之后就可以一直用，游戏是我输入了账号，密码点击登录，就加载你的个人数据进入了，到登录界面时，他基本上把能加载的资源都加载了，你要手动到登录界面，只能说手疼。（有个二货条件步进20000行，点了两个小时，我也不认识他，你们笑话他吧）

## 定位游戏加载界面

这里我们先用线程入口来找到游戏加载的界面，然后再这个附近位置分析进程，查看字符，搜索能确定的信息，附近打断点。也很麻烦，因为不同的事情可能再不同的线程处理，有的已加载，分到到具体还得去打断点，打完断点，你重新加载的时候又可能要不停的跳。上来就又给自己加难度啊，多线程。

![](https://attach.52pojie.cn/forum/202503/01/212510d1dydtzv1ptadd99.png)

`设置.png` (187.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1ODE2NHxlMzQ2ODNiNnwxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODE2NHxlMzQ2ODNiNnwxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes)

2025-3-1 21:25 上传

## 程序规范与明文

如果游戏没壳，并且程序规范（这个时候越规范对你越友好）很多东西都是明文的

![](https://attach.52pojie.cn/forum/202503/01/212727fdadzrj0a8kszj0m.png)

`春天.png` (186.14 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1ODE2NXxmMGYxZGRmN3wxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODE2NXxmMGYxZGRmN3wxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes)

2025-3-1 21:27 上传

## OD 与 Xdbg 使用经验

最后一个，使用经验，可能是 OD 我不会用，说错话，大佬捶我就好，我认错。OD 作者好久没更新了，听说有 2.0 我也不知道，Xdbg 是在维护的，有中文最最最让我喜欢的是有流程图，快捷键别的没记住这个记住了`G`，外挂？框图的作用是能让我这种小菜鸡简单的理清楚跳转的脉络。对哪个地方分析的越细，流程图就越复杂。这个觉得是对新手非常友好的功能。

![](https://attach.52pojie.cn/forum/202503/01/214322se0005zq0yr1krki.png)

`微信截图_20250301175905.png` (23.99 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1ODE2OXxiMmMxZjI5ZXwxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODE2OXxiMmMxZjI5ZXwxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes)

2025-3-1 21:43 上传

## 断点与数据流分析

我们要重点分析哪个位置就只断那些位置，不调试的位置不会出现线，下面是，`get` 服务器数据，跟不 `get` 的区别，我们可以看出来交换数据在哪些断里面。在加解密的位置肯定会有个断开的，数据一定是加密了发送出去，解密了传递到程序。

![](https://attach.52pojie.cn/forum/202503/01/214335esuuvtps0zzurgsr.png)

`框线图.png` (38.9 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc1ODE3MHxiMmQ0MWNkNHwxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODE3MHxiMmQ0MWNkNHwxNzkzNTcxMjU1fDE2ODA4NnwyMDEwNjQ0&nothumb=yes)

2025-3-1 21:43 上传

## 总结

好吧，总结一下，破解需要一颗强大的内心，对大佬的敬意，然后老老实实的学`debug`吧，还需要一个 DeepSeek 帮我们写代码。我接下来就是好好学学习一下 Xdbg，有看到这里的大佬，给你准话，这次这条路能通不了。
