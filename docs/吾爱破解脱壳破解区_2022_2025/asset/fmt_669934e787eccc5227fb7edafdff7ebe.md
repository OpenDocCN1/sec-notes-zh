
# [CTF] 【2022】春节解题领红包之三——新手向

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1588987
- 抓取时间: 2026-11-01T23:36:00

## 那年我找到了一个`String`，却以为抓住了整个`Key`。

### 如果说上一篇是动静分离，那么这一篇就是动静结合了，对于大佬而言可能只需要`OD`工具就调出最终的结果了，但对我来这种菜鸡来讲需要多个工具一起分析，再加上一个不太灵光的脑袋，也是可以勉强一战的，推荐先阅读上一篇文章【2022】春节解题领红包之二——新手向 ( [//www.52pojie.cn/thread-1588722-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1588722-1-1.html:href="https) ) 其中的一些思路和操作会对这道题的分析有帮助，如果你能动手跟着做一下那最好不过了，Enjoy the Time!

### 工欲善其事，必先利其器

吾爱专用虚拟机2.0 ( [//www.52pojie.cn/thread-661779-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-661779-1-1.html:href="https) )
`IDA`7.5 ( [//www.aliyundrive.com/s/4yE1LbvctT5" target="_blank" rel="noopener noreferrer nofollow"](https://www.aliyundrive.com/s/4yE1LbvctT5:rel="nofollow noopener" href="https) )

### 查壳

![](https://attach.52pojie.cn/forum/202202/17/210019aj7yjjtqdbj5ho6s.png)
pic-55.png (32.17 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcwM3w5MTY2ZDU3YnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwM3w5MTY2ZDU3YnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### `UPX`壳,我们的老朋友，使用`ESP`定律即可，关于`ESP`定律脱壳详细操作可参考我之前文章 ( 2020春节红包之二———小白成长的血泪史[//www.52pojie.cn/thread-1101472-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1101472-1-1.html:href="https) ) 观看脱壳部分学习即可，此处不多赘述。

### 收集信息，需要我们的`UID`和最终的`Key`，随便输入，提示`Error`

![](https://attach.52pojie.cn/forum/202202/17/210022z52ejjmz636j2hss.png)
pic-56.png (9.23 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcwNHwzZTM5MmI0OHwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwNHwzZTM5MmI0OHwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

## 动态追溯

### 将脱壳后的文件拖入`OD`，这里选择否

![](https://attach.52pojie.cn/forum/202202/17/210026o6x5l4uzkh1zlhu4.png)
pic-57.png (76.32 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcwNXw0MWU0YTI2NnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwNXw0MWU0YTI2NnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 右键依次选择中文搜索引擎—>智能搜索,`Ctrl+F` 找我们的提示信息`Error`(按`N`寻找下一个匹配项),这里除了`Error`,`Success`,还有一串神秘疑似`key`的字符串

![](https://attach.52pojie.cn/forum/202202/17/210029w7dp7dfzq4aozfw7.png)
pic-58.png (78.52 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcwNnw4Y2ZiOTQ5MXwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwNnw4Y2ZiOTQ5MXwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 结合前几年的题目，开始我的偷懒猜想，可就是这个字符串，困了我整整一天

![](https://attach.52pojie.cn/forum/202202/17/210033jh665lhoq1uocqlq.png)
pic-59.png (46.12 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjQ5NTcwN3wzMWRjMDM5MnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwN3wzMWRjMDM5MnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 既然看到我们的`Success`，那就它对应的汇编代码，双击后回到我们的汇编页面，熟悉的`jnz`，熟悉的`cmp`，还是原来的配方，还是原来的味道

![](https://attach.52pojie.cn/forum/202202/17/210036nfhrurfifzyyhzjr.png)
pic-60.png (91.18 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcwOHw0MGMwZDRkNHwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwOHw0MGMwZDRkNHwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 往上我们发现了两个`call`函数，在离的最近的下个断点(按`F2`或者双击汇编代码前区域)，待会看看有没有什么发现，按`F9`运行程序后，输入测试的1和123456（假装自己的`UID`为1哈哈哈哈），`F7`进入断点，一通`F8`过后，手指都快按断了，却什么也没有发现

![](https://attach.52pojie.cn/forum/202202/17/210039jazyxx4ysxc3eecs.png)
pic-61.png (88.53 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjQ5NTcwOXw1ZDNmZjc1OXwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcwOXw1ZDNmZjc1OXwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

## 静态分析

### 于是决定使用`IDA`分析一波，将文件再拖入`IDA`中看看有什么发现，阿，很快啊，我们就找到`Success`了，然后发现它上方面的判断模块，`F5`将汇编代码转为伪代码，(使用`Shift+F12`调处字符串窗口，再搜索`Success`也是可以的)

![](https://attach.52pojie.cn/forum/202202/17/210042vrkz84bmfrbf8o2d.png)
pic-62.png (134.66 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcxMHxmOGI2YjZkNnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxMHxmOGI2YjZkNnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 这里的`sub_401520`函数就是决定我们成功或是失败的条件，这里要求它的返回值为1才能进入`Success`，点击`sub_401520`函数，进入它的内部看看

![](https://attach.52pojie.cn/forum/202202/17/210045e93dkddaap2zdkcc.png)
pic-63.png (160.86 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjQ5NTcxMXxhZDJiMWRhZXwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxMXxhZDJiMWRhZXwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 直接滑到底部发现`result`，而`result`的结果又与`v8`的值有关，而`v8`的结果又与另外一个函数`sub_403ED0`有关，这搁这套娃呢这是。我们必须保证`v8`返回值为`false`(为0)，才可以使`result`的值为1，我们在这里打上断点(双击)，调试看看（这里不清楚的同学可以看我开头提到的前一篇文章）这里的`if`语句以及这个函数中很长的代码部分需要注意一下，我们后边会用到

![](https://attach.52pojie.cn/forum/202202/17/210049wgg57ncbv0zlgnv7.png)
pic-64.png (161.24 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcxMnw1NmYxYzUzY3wxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxMnw1NmYxYzUzY3wxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 运行后输入我们的测试数据1，123456后回车，在断点的地方停下来，显示这个页面不用担心，按`F5`将代码转为伪`C`代码

![](https://attach.52pojie.cn/forum/202202/17/210051vzexss6s760g7s05.png)
pic-65.png (152.49 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjQ5NTcxM3w1NThlNTlkN3wxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxM3w1NThlNTlkN3wxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 按`F7`进入函数内部，然后`F8`执行，如果没有思路的时候，可以把鼠标悬停变量上，或许会有意想不到的收获，这里的`v3`为`0x21`，16进制转十进制即 33，这会预示什么呢？

![](https://attach.52pojie.cn/forum/202202/17/210054vni71a1ugzom7wnz.png)
pic-66.png (171.64 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcxNHwwNmY0ODY0ZnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxNHwwNmY0ODY0ZnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 继续`F8`，这个`V4`变量显示一个`String`(字符串)

![](https://attach.52pojie.cn/forum/202202/17/210057ucl6422k948vz6ad.png)
pic-67.png (171.73 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcxNXw0Y2U1NmFjY3wxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxNXw0Y2U1NmFjY3wxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:00 上传

### 双击`V4`，找到它的源地址，并把这个`String`拿到，得到"`flag{Happy_New_Year_52Pojie_2022}`"，字符串长度恰好为33 = (`0x21`)，这就是答案吗？不，才刚刚开始，，，

![](https://attach.52pojie.cn/forum/202202/17/210100reaxe96ldpz1j366.png)
pic-68.png (199.37 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcxN3w2ZTdkYzUzZnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxN3w2ZTdkYzUzZnwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:01 上传

### 继续向下分析，我们发现这个`v6`中存放的值是3，也就是我们输入的字符串"`123`"的长度，而接下来又看见我们熟悉的`result` 和`memcmp`函数了，前面提到需要保证返回值为0，才可以进入`Success`，据此我们可知`Key`的长度为33

![](https://attach.52pojie.cn/forum/202202/17/210103yuulek8kiq7l5v1z.png)
pic-69.png (166.95 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjQ5NTcxOHw1NTMzOTU0MHwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NTcxOHw1NTMzOTU0MHwxNzkzNTc2MTU4fDE2ODA4NnwxNTg4OTg3&nothumb=yes)
2022-2-17 21:01 上传
