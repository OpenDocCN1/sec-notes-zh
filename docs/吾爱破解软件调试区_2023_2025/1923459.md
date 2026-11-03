
# [调试逆向] ROP之ret2libc

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1923459
- 抓取时间: 2026-11-01T21:53:56

## Ret2Libc概述

程序动态加载`libc`库中功能函数，利用溢出漏洞，使用`libc`中比如`system()`的地址覆盖返回地址，执行`libc`功能，控制执行流程，或者拿到`shell`等。

## 实例分析

### 静态分析，同样`gets()`函数的溢出漏洞

![](https://attach.52pojie.cn/forum/202405/13/092127mafvjd3jujd1oojv.png)
image.png (180.4 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTQ0MXw2MzgwZjE5OHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0MXw2MzgwZjE5OHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:21 上传

### 确认文件属性

![](https://attach.52pojie.cn/forum/202405/13/092141dbzgzr8t1onq90m9.png)
image.png (216.81 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY5NTQ0MnxkMmIzZWUzMHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0MnxkMmIzZWUzMHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:21 上传

![](https://attach.52pojie.cn/forum/202405/13/092149kgmmd4hmg9agqk24.png)
image.png (230.83 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTQ0M3wxYjdjNmYzNnwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0M3wxYjdjNmYzNnwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:21 上传

### 调试，确定溢出偏移

![](https://attach.52pojie.cn/forum/202405/13/092203qmy4ybv2cwrr2yf1.png)
image.png (912.3 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTQ0NHw5YjM5NDk3M3wxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0NHw5YjM5NDk3M3wxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:22 上传

![](https://attach.52pojie.cn/forum/202405/13/092229njn2b02ok8qcielk.png)
image.png (427.91 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5NTQ0NnwxMmY5OWIzMXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0NnwxMmY5OWIzMXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:22 上传

![](https://attach.52pojie.cn/forum/202405/13/092237ll3ls9lw96q11sc5.png)
image.png (94.44 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTQ0N3xmNDY3ZDFkNnwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0N3xmNDY3ZDFkNnwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:22 上传

所以，溢出长度位`108 + 4 = 112`

![](https://attach.52pojie.cn/forum/202405/13/092252b1ppp0jwiknnp7mz.png)
image.png (86.01 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTQ0OHxiNDJhYzNhZXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0OHxiNDJhYzNhZXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:22 上传

### 确定`/bin/sh`偏移

![](https://attach.52pojie.cn/forum/202405/13/092313jvcgmlhjnm8ic8m7.png)
image.png (190.47 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY5NTQ0OXw1MmUyYWRiOXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ0OXw1MmUyYWRiOXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:23 上传

### 有关`libc`中`system()`函数的偏移位置，需要参考`PLT`和`GOT`的关系

`GOT`表是存储外部函数库的表`PLT`则是由代码片段组成的，每个代码片段都跳转到`GOT`表中的一个具体的函数调用二者的关系参考 https://saku376.github.io/2021/04/30/GOT%E8%A1%A8%E5%92%8CPLT%E8%A1%A8/

### 函数第一次调用过程

![](https://attach.52pojie.cn/forum/202405/13/092336e2ba6pfy6yizakbw.png)
image.png (182.82 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5NTQ1MHxlN2UxNTk4YXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ1MHxlN2UxNTk4YXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:23 上传

第一步由函数调用跳入到`PLT`表中，然后第二步`PLT`表跳到`GOT`表中，第三步由`GOT`表回跳到`PLT`表中，这时候进行压栈，把代表函数的`ID`压栈，第四步跳转到公共的`PLT`表项中，第五步进入到`GOT`表中，第六步 `_dl_runtime_resolve()`对动态函数进行地址解析和重定位，第七部 把动态函数的真实地址写入到`GOT`表项中，第八步 执行函数并返回。 `dynamic`段：提供动态链接的信息，例如动态链接中各个表的位置`link_map`: 已加载库的链表,由动态库函数的地址构成的链表`_dl_runtime_resolve()`: 在第一次运行时进行地址解析和重定位工作

### 之后的调用过程

![](https://attach.52pojie.cn/forum/202405/13/092354a8v48kefx8v4v6zc.png)
image.png (157.35 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5NTQ1MXxiY2U2Zjg2MHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ1MXxiY2U2Zjg2MHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:23 上传

所以，为了构造`exp`，找`libc`中的`system()`时，找对应的符号在`PLT`表中的地址即可

![](https://attach.52pojie.cn/forum/202405/13/092428rfrgqtqppz0f82og.png)
image.png (246 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY5NTQ1M3w3ZWIxNzUyOHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ1M3w3ZWIxNzUyOHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:24 上传

### 至此，构造`exp`

![](https://attach.52pojie.cn/forum/202405/13/092506iv1i0f9wt0lv0ltl.png)
image.png (412.19 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5NTQ1NXw1OTEwZWEzMHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ1NXw1OTEwZWEzMHwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:25 上传

### 验证，获取`shell`

![](https://attach.52pojie.cn/forum/202405/13/092522f2n5j595ou5299k2.png)
image.png (309.33 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY5NTQ1NnxiMGJmMzllOXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NTQ1NnxiMGJmMzllOXwxNzkzNTcwMDMzfDE2ODA4NnwxOTIzNDU5&nothumb=yes)
2024-5-13 09:25 上传
