
# [原创] 练习笔记之160Crackme-020

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1644149
- 抓取时间: 2026-11-01T23:22:56

## 160CM-020

首先，拖入`PE`看一下。妈呀

![](https://attach.52pojie.cn/forum/202206/02/122212h8yxi81mkl878b9m.png)

10.png (103.04 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUyNzIxOXxmNDU5NGExNnwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxOXxmNDU5NGExNnwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:22 上传

妈呀~~居然有壳。对于只用过`UPX Unpacker`一键脱壳的菜鸟选手来说，表示有点方……

### 1. 脱壳

幸好最近逛论坛的时候看到好像有一偏关于脱壳的文章，名字叫“关于压缩壳脱壳步骤详解！看这一篇足够。”（[//www.52pojie.cn/thread-1641368-1-1.html"](https://www.52pojie.cn/thread-1641368-1-1.html:href="https)），赶紧翻出来看看。想想宝典在手，终于可以手撕压缩壳了，内心忍不住有点小激动ing。

看看方法一：单步跟踪法，关键是我也不知道`OEP`长啥样啊，跟过去见到了也不认识啊，先pass了。

方法二：`ESP`定律法，这个看起来好像比较靠谱的样子。按文章描述的方法“`pushad`开头，`F8`一次以后，8个寄存器只有`ESP`是红色时候，就可以使用该定律。右键`ESP`选择数据窗口跟随，数据窗口下硬件访问断点，`F9`运行，此时已经接近`OEP`。”

操作方法其实已经很简洁明了了，但是对于第一次手撕压缩壳的初哥来说，其实还是会碰到不少问题。先一步一步操作一下试试：

- `OD`载入程序后，开头不是`pushad`，也没关系，按一下`F8`，注意观察`OD`右上角的寄存器中只有`ESP`的值是红色，如果还有其他是红色，继续`F8`。
- 在`ESP`地址位置，右键，选中[数据窗口中跟随]。
![](https://attach.52pojie.cn/forum/202206/02/122153tahk4hx8zalvll3h.png)

01.png (15.5 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIwOXxhNWQ2NDEzM3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIwOXxhNWQ2NDEzM3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:21 上传
- 在数据区选中刚才跳转到的地址，右键选择[断点]→[硬件访问]→[Word]。
![](https://attach.52pojie.cn/forum/202206/02/122155tb31fxf21oq30bxe.png)

02.png (21.32 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIxMHw4ZDI5NjNlZXwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxMHw4ZDI5NjNlZXwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:21 上传
- 按一下`F9`运行程序，直接来到了大跳转处。再按下`F8`，就到达程序`OEP`。这里的判断关键是大跳转，要跳的比较远的才算，至于`F8`以后见到的`OEP`长啥样，说实话好像啥样的都有。
![](https://attach.52pojie.cn/forum/202206/02/122157ba1wka7cx7fkcjcj.png)

03.png (10.24 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIxMXxhMzlhNGI1N3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxMXxhMzlhNGI1N3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:21 上传
- 在`OD`中选中[插件]→[`OllyDump`]→[脱壳在当前调试进程]，进入脱壳界面，可以看到“入口地址修正值”为进入`OEP`后的地址（注意不要在大跳转的地址位置脱壳），直接点脱壳就完成了`Dump`。
![](https://attach.52pojie.cn/forum/202206/02/122159w08080ajpespyjf0.png)

04.png (31.43 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIxMnxkNzJmMjQ2Y3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxMnxkNzJmMjQ2Y3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:21 上传
- 有些程序脱壳后可能无法正常运行，这就需要进行`IAT`修复了。打开`Import REConstructor`，在进程列表中选中需要修复程序的原始程序（运行一下脱壳前的程序，不要选`OD`打开的那个进程）,先在`OEP`位置填入地址，这个地址是相对地址，要用`OD`中的`OEP`地址减去偏移地址`400000`。然后点击“`autosearch`”按钮，地址正确的话会显示找到了，然后点击“`Get Imports`”按钮，会在左边的列表中列出已经找到的信息，正常的话应该是全部`YES`的，如果有不是`YES`的，还要手动修复（没碰到过，还不会手动修复）。最后点击“`Fix Dump`”，在弹出的选择框中选中之前脱壳出来的程序，确定就可以了。
![](https://attach.52pojie.cn/forum/202206/02/122328wta2e0cwdfexcx7w.png)

11.png (50.05 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIyMnxhMTFhNDkzNXwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIyMnxhMTFhNDkzNXwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:23 上传

PS：`OD`中设置的硬件断点在`Ctrl+F2`后不会自动清除，需要在菜单[调试]→[硬件断点]，去手动删除已设置的断点。

### 2. 爆破

运行脱壳后的程序，可以发现这个程序是要求输入序列号，然后点击按钮后会算出一个注册码，要求算出的注册码和界面上显示的一致。由于`PE`分析显示这是一个`Delphi`程序，因此`IDR`用上再说。
![](https://attach.52pojie.cn/forum/202206/02/122216qe84123xer12818e.png)

12.png (18.26 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIyMXwxNjM5YmRjYnwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIyMXwxNjM5YmRjYnwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:22 上传

`IDR`分析后，我们找到按钮响应事件的地址是`44A2E8`，在下面可以找到关键跳转地址`44A387`，将代码`nop`掉就爆破完成了。
![](https://attach.52pojie.cn/forum/202206/02/122207hg952554lwlpfg7b.png)

08.png (23.86 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIxN3xjYWRkNzE4MXwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxN3xjYWRkNzE4MXwxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:22 上传

### 3. 算法分析

因为`IDR`能够识别出`delphi`库函数，因此我们将`IDA`生成的伪代码里面的函数都改一下名字，就能得到下面的代码了。
![](https://attach.52pojie.cn/forum/202206/02/122205p3zqrrqaq39514qu.png)

07.png (19.95 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIxNnwwMjQ4NDE1M3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxNnwwMjQ4NDE1M3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:22 上传

从代码中可以看出关键比较是`StrCmp`函数，`V14`看不来源，`IntToHex`函数的第二个参数也看不懂，这个需要查一下`IntToHex`函数的参数定义，第二个参数是指定转换后16进制字符串的最小长度。`OD`中跟踪一下就可以确定第二个参数是`6`，`V14`就是`IntToHex`生成的字符串。`V7`是转64为整型后的低32位，其实和`V6`是一样，其实整个算法就是输入的数字乘`3`后转成16进制字符串，然后与界面上显示的字符串比较。因此序列好就是界面显示的16进制数字除`3`对应的十进制数字。计算器算一下后输入，验证成功。
![](https://attach.52pojie.cn/forum/202206/02/122209nwu4zyeicow8ouzs.png)

09.png (21.42 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUyNzIxOHwwY2Y2ZDBkM3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNzIxOHwwY2Y2ZDBkM3wxNzkzNTc1Mzc0fDE2ODA4NnwxNjQ0MTQ5&nothumb=yes)

2022-6-2 12:22 上传

### 4. 总结

看了脱壳文章里面介绍的方法，尝试了一下，发现只会`ESP`定律这一招，其他的照葫芦画瓢都画不出来，下一篇打算先琢磨一下手撕压缩壳手法，尽量写一篇菜鸟也能看懂，也能照着完成的经验帖，避免一看就会，一试就废的囧境。
