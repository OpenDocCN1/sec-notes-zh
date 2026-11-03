

# [CTF] 【2022】春节解题领红包之二——新手向

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1588722](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1588722)
- 抓取时间：2026-11-01T23:36:07

## 工具

在论坛的爱盘里均可下载：

- 吾爱专用虚拟机2.0
- IDA7.5

## 查壳

将程序拖入PEID查壳，发现未加壳，直接进行下一步（可以看出程序由 C/C++ 使用 VC++6.0 编写）。

![查壳结果](https://s2.loli.net/2022/02/16/ZRM1bSanjCNgFs7.png)

## 动态追溯

### 获取信息

首先打开程序，随意输入字符串（此处输入`123456`），查看有什么提示信息。程序提示`Error, please try again`，记住这句话，待会会用到。

![输入测试字符串](https://attach.52pojie.cn/forum/202203/10/222721k7vt5r0pc0dy6dam.png)

`pic (2).png`（10.14 KB，下载次数：0） 附件：[查看/下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTE3MHw5NDJiOGMwZXwxNzkzNTc2MTY0fDE2ODA4NnwxNTg4NzIy&nothumb=yes)（2022-3-10 22:27 上传）

## OD动态跟踪

### 拖入OD

![拖入OD](https://attach.52pojie.cn/forum/202203/10/222724ip3jilzjzw3mp66c.png)

`pic (3).png`（91.5 KB，下载次数：0） 附件：[查看/下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTE3MXw2NTU5YzRmZHwxNzkzNTc2MTY0fDE2ODA4NnwxNTg4NzIy&nothumb=yes)（2022-3-10 22:27 上传）

此时程序暂停，点击图中所示按钮或按`F9`使程序继续运行，出现熟悉的页面。

![程序运行页面](https://s2.loli.net/2022/02/16/UzhVMnaYuQt4Sei.png)

### 中文搜索引擎—>智能搜索

此时最小化窗口，右键依次选择中文搜索引擎 → 智能搜索。

![中文搜索引擎—>智能搜索](https://s2.loli.net/2022/02/16/9KgxuS6wJC47Wfj.png)

### 搜索字符串

找到字符串窗口，按`CTRL+F`搜索之前找到的`Error, please try again`。

![搜索字符串](https://attach.52pojie.cn/forum/202203/10/222804krsxplh8n8rxlpi4.png)

`pic (6).png`（100.29 KB，下载次数：0） 附件：[查看/下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTE3MnxmNzk4NTk5OXwxNzkzNTc2MTY0fDE2ODA4NnwxNTg4NzIy&nothumb=yes)（2022-3-10 22:28 上传）

### 分析长度判断

双击这一行内容，回到主界面。可以看到当前行上面有一个向下的`je`跳转，再往上看到`cmp eax,0x17`。这需要一点汇编知识：`cmp`比较`eax`寄存器与`0x17`的值，而`0x17`为十六进制数字，即`0x17 = 1 * 16 + 7 = 23`，由此不难知道最终`password`的长度为23。

![比较长度](https://s2.loli.net/2022/02/16/9imgrX52Idw8zfM.png)

### 在 cmp 和 je 处下断点

在`cmp`和`je`这两行下断点（双击汇编代码前区域或按`F2`），使用测试数据`11111111111111111111111`。此时看到断点处停下，与预想一致。如果仔细观察，可以发现最终答案已经在右下角出现。

![断点处的答案](https://s2.loli.net/2022/02/16/YgQ2l9mbowzUWZv.png)

### 进入 call 函数分析

到这里已经差不多结束。当然可以一路`F8`，看到熟悉的`Success`。不过不要着急，发现有一个`jnz`的跳转，`jnz`上有一个`test eax,eax`，再往上发现一个`call`函数，这里便是解题的关键。在`call`函数处打上断点，这次按`F7`进入函数内一探究竟。

![call 函数与 jnz](https://s2.loli.net/2022/02/16/kOzDSeh1jfQwulF.png)

进入函数内部，继续一路`F8`，然后又看到熟悉的`password`。输入的`11111111111111111111111`与`2022HappyNewYear52PoJie`进行`cmps`（字符串比较操作）。

![字符串比较](https://s2.loli.net/2022/02/16/HhN62xQXmUu3eaj.png)

继续一路`F8`，从`call`函数出来后，使用`test eax,eax`，此时`eax`值为`FFFFFFFF`，即`-1`，而`jnz`需要为0才能进入`Success`，因此答案与之前分析一致。

![比较结果](https://s2.loli.net/2022/02/16/LUayWihZsjkf91B.png)

### 验证答案

![验证答案](https://attach.52pojie.cn/forum/202203/10/222842f5psyosss9thmmta.png)

`pic (12).png`（10.15 KB，下载次数：0） 附件：[查看/下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTE3M3xkZjk1Mjg4ZHwxNzkzNTc2MTY0fDE2ODA4NnwxNTg4NzIy&nothumb=yes)（2022-3-10 22:28 上传）

## 静态分析

再使用IDA分析一下程序。

### 拖入IDA并初步查看

拖入IDA，点击`OK`。

![拖入IDA](https://s2.loli.net/2022/02/16/B5nOvY28yRzu4Wd.png)

在主界面可以使用右键拖动窗口，或使用滚轮缩放画面大小。可以看IDA使程序执行的过程看起来更加清晰，很容易看到需要函数的执行走向，快速定位函数地址。

![IDA 主界面](https://s2.loli.net/2022/02/16/k9vECV2s4nAFNx1.png)

### 字符串彩蛋与长度提示

有时候可能也会收获一些意外的惊喜。比如将鼠标悬停在`HappyNewYear!`上再滑动鼠标滚轮，可以得到一个tips：`Congratulations,there is a tip: flag length is 0001 0111`，这也许是官方给我们埋的彩蛋。

![字符串提示](https://s2.loli.net/2022/02/16/PdRexX43uGQZLqm.png)

点击`HappyNewYear!`字符串后，可以顺势找到源头。

![找到字符串源头](https://s2.loli.net/2022/02/16/sFPljUNog9WrAev.png)

不过这种方法似乎太欧皇了，不是谁都可以这么容易找到。可以按`shift+F12`快捷键调出字符串窗口，然后同样可以找到对应的字符串，双击后也可以到达上图的位置。

![字符串窗口](https://s2.loli.net/2022/02/16/lM9sPzwF2UhTYnm.png)

这句话的意思是，祝贺，这是一个提示，`flag`的长度为23。为什么是23呢？`0001 0111`为二进制字符串，`0001 0111 = 1 * 2^4 + 0 + 1 * 2^2 + 1 * 2^1 + 1 = 23`。还不太熟悉的同学可以通过百度二进制数转十进制来补补课（注：OD里面的字符串搜索也可以看到这个tip）。

### 定位 Success 判断模块

既然知道长度为23，便可以直奔主题。打开主界面，发现关于`Success`的函数判断模块。

![Success 函数判断模块](https://attach.52pojie.cn/forum/202301/24/174147ylon42sl7davav3l.png)

`image-20220216152113168.png`（132.5 KB，下载次数：0） 附件：[查看/下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjI1OXxjMDA3OTE1MXwxNzkzNTc2MTY0fDE2ODA4NnwxNTg4NzIy&nothumb=yes)（2023-1-24 17:41 上传）

### 使用 F5 分析 sub_402B70 函数

可以按`F5`将汇编转为伪C语言代码，出现弹窗点击`OK`即可。分析不难发现`Success`与否，与这个`sub_402B70`函数的返回值有关。可能有人觉得看不懂，没关系，且听细细分析。

![F5 伪C代码](https://s2.loli.net/2022/02/16/e3UzrfMhInFb81p.png)

点击这个`sub_402B70`函数，进入函数内部，发现最终返回一个`result`，这个值是需要重点关注的。再往上可以看到`memcmp`函数赋值给`result`，而它是用于比较字符前`v7`值是否相等。可以给它打上断点看看它做了什么：选择`Local Windows debugger`，在行号前点击出现断点，运行程序。

![sub_402B70 函数内部](https://s2.loli.net/2022/02/16/8Fuqyjrh3Tw7QEc.png)

### 动态调试 memcmp 比较逻辑

输入测试字符串`11111111111111111111111`，回车。如果与我的界面不同，按下`F5`即可。

![输入测试字符串调试](https://s2.loli.net/2022/02/16/wrOK6HDU1djaAqE.png)

鼠标悬停在`v9`、`a4`、`v3`分别看到`11111111111111111111111`、`2022HappyNewYear52PoJie`、`0x17`，这三个都是老演员了。记住这个`v4`，它是这个函数的第四个参数。继续`F8`可知`result`值为`-1`，虽然对`result`进行取反操作，但C语言中`bool`类型除0之外皆为`true`，因此返回了`-1`。回到`sub_402B70`中，发现`v4`对应了上面变量`v7`的值，打上断点，运行。

![调试比较逻辑](https://s2.loli.net/2022/02/16/cpfxzR7vgCD5E3n.png)

双击`v7`变量，按住鼠标左键选择到这个`0`的前面停止。

![选择 v7 变量](https://s2.loli.net/2022/02/16/Dk4NY21cgdUiFym.png)

右键图中操作或按`A`将字符数组转为字符串。

![转为字符串](https://s2.loli.net/2022/02/16/MLvsrno7Wpbjmal.png)

### 获得并验证 password

![获得 password](https://s2.loli.net/2022/02/16/4j6xAKhsQ2PeoSw.png)

验证答案，没得问题。

## 结语

以上内容均是通过学习论坛内外相关知识所得，所以大家没有解出来的话不用灰心，可以照我的步骤走一遍，体验一下解题的快乐。新的一年祝各位坛友身体健康，学习进步，工作顺利，万事如意，希望吾爱论坛越来越好！

我好懒，还有第三题的过程没写，抽空再写吧。

### 第三题已更新

详见：[【2022】春节解题领红包之三——新手向](https://www.52pojie.cn/thread-1588987-1-1.html)

