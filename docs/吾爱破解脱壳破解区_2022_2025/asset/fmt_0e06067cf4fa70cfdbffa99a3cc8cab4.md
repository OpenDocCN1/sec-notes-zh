
# [Web逆向] 某qMusicSign参数vmp逆向分析
- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2079071
- 抓取时间: 2026-11-01T21:58:31

## 前言
文章中所有内容仅供学习交流使用，不用于其他任何目的，抓包内容、敏感网址、数据接口均已做脱敏处理。严正声明禁止用于商业和非法用途，否则由此产生的一切后果与作者本人无关。
- 站点：某Qmusic的vmp
- 加密点：载荷数据中的`sign`值
- 网址：aHR0cHM6Ly95LnFxLmNvbS8=
- 接口：aHR0cHM6Ly91Ni55LnFxLmNvbS9jZ2gtYmluL211c2ljcy5mY2c/
废话：上一篇的文章写的太糙了，这一篇尽量写的详细。
- 先抓包，找到需要逆向的参数，此次只是以学习目的分析`sign`的生成过程，其他参数不做分析，如图
![](https://attach.52pojie.cn/forum/202512/11/100107zi1c2h7839k8b4q4.png)
175701-1.png (108.26 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkxOHwyNDVjYTZlOHwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkxOHwyNDVjYTZlOHwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
1
2025-12-11 10:01 上传
- 我们直接全局搜索sing值
![](https://attach.52pojie.cn/forum/202512/11/100112vtr69u303jsuawsq.png)
175724-1.png (164.86 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkxOXw2MDlkZDFhY3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkxOXw2MDlkZDFhY3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
2
2025-12-11 10:01 上传
- 看到 `sign`: `c` 发现这个参数在 `swich-case` 语句中，我们直接在他的上一步 `return` 处打上断点看下。
![](https://attach.52pojie.cn/forum/202512/11/100116g0ltdu10ewzc0tnw.png)
175754-1.png (106.23 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyMHxiYjk3ODM4NXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyMHxiYjk3ODM4NXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
3
2025-12-11 10:01 上传
- 可以看到 `r.data` 的明文值经过了 `P` 方法后生成了我们想要的加密参数，那么猫腻就是这个 `P` 方法了！我们先狠狠的进入这个 `p` 中！
![](https://attach.52pojie.cn/forum/202512/11/100121j55hh847n7s7qg8f.png)
175822-1.png (80.49 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyMXxmNjQyOGFiYXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyMXxmNjQyOGFiYXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
4
2025-12-11 10:01 上传
![](https://attach.52pojie.cn/forum/202512/11/100125ufk8bi9z1bezf00k.png)
175839-1.png (32.15 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyMnw2ZmE2YWFjZHwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyMnw2ZmE2YWFjZHwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
5
2025-12-11 10:01 上传
- 稍微往上拉一下代码，可以看到它是在一个自执行函数中，把这个自执行函数折叠起来，可以看到它传入了一长传的字符串巴拉巴拉一堆东西。它比较鬼的是，他把 `G._getSecuritySign` 赋值给 `P`后它就给删除掉了，但其实对我们影响不大，我们把它整个复制出来。
![](https://attach.52pojie.cn/forum/202512/11/100129b4hrsugodohf4f0u.png)
175857-1.png (80.35 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyM3xhNWFhMzM0NnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyM3xhNWFhMzM0NnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
6
2025-12-11 10:01 上传
- 新建一个代码片段，把前面拿到的明文值传进 `P` 方法，点击执行后，发现它报错了，`G` is not defined ，说明 `G` 未找到，我们回到原文去往上翻代码找 `G` 。
![](https://attach.52pojie.cn/forum/202512/11/100135mxiq8xz9m0sigcfq.png)
175928-1.png (25.92 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyNHxmNjZmZTVkN3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyNHxmNjZmZTVkN3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
7
2025-12-11 10:01 上传
- 发现 `G` 就在自执行函数上面定义的，我们直接复制下来放到代码片段中再执行。
![](https://attach.52pojie.cn/forum/202512/11/100139eaff0dnvaqa4nzal.png)
175942-1.png (88.01 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyNXxkYmU2NjBmNXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyNXxkYmU2NjBmNXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
8
2025-12-11 10:01 上传
- 没问题，现在已经可以随便执行，这样方便我们调试代码。
- 我们先简单看下这个代码的执行流程。
![](https://attach.52pojie.cn/forum/202512/11/100144psuhmyrtbf3xzhr3.png)
175956-1.png (50.7 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyNnxjYTI1N2Y5N3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyNnxjYTI1N2Y5N3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
9
2025-12-11 10:01 上传
- 这就很明显了，我们把重点放在自执行函数中，进入函数看它先执行的代码位置。
![](https://attach.52pojie.cn/forum/202512/11/100150j4zll7gcwy8k0l6w.png)
180019-1.png (26.1 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyN3w3ZmM2ZjAzY3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyN3w3ZmM2ZjAzY3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
10
2025-12-11 10:01 上传
- 把 `e` (长字符串)参数传入 `o` 函数中，`o` 函数中又经过了 `t` 函数、`n` 函数，叮叮咣咣的一顿，出来就变成了指令集，这个过程感兴趣的大佬可以自己单步执行去看一下过程，我看不明白，我就不乱说误导了~
![](https://attach.52pojie.cn/forum/202512/11/100156v2cm93j2jfner9j8.png)
180102-1.png (30.36 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkyOHwzZjc3OGQ1MnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkyOHwzZjc3OGQ1MnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
11
2025-12-11 10:01 上传
- 出来看 `n` 的返回值就已经变成了指令集。
![](https://attach.52pojie.cn/forum/202512/11/100232udv2os939bc2ofao.png)
180125-1.png (25.87 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkzMHw1NmNhNzBkYnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkzMHw1NmNhNzBkYnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
12
2025-12-11 10:02 上传
- 接着进行单步它没有直接跳入`i`或者是 `a` 函数，它是跳到了一个三元表达式，`t` 我们传入的是一个 false 所以它会跳进 `a` 函数中，但是我看了一下，`i` 函数和 `a` 函数俩个是一模一样的，我后边在进行逆推算法时也是在俩个函数中都进行了插桩，因为我记得在 `i` 函数中打印日志也会输出，这个各位大佬可以自行测试下，不知道有没有发生变化。
![](https://attach.52pojie.cn/forum/202512/11/100239kbpoyfd6qtyarqco.png)
180138-1.png (25.26 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkzMXw1NzBjYmZlN3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkzMXw1NzBjYmZlN3wxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
13
2025-12-11 10:02 上传
- 进入到 `a` 函数中就是一个 `for` 的死循环，然后就是 `swich-case` 还好这个不是很多，也不是很繁琐，不需要费太多脑子，我们直接找 `call` 或者是 `apply`，都还是比较明显的，现在这俩个位置打上日志断点看下日志。
![](https://attach.52pojie.cn/forum/202512/11/100244ezgwpfydg8dd1dbz.png)
180148-1.png (31.81 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkzMnwyYWI2NGRjYXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkzMnwyYWI2NGRjYXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
14
2025-12-11 10:02 上传
![](https://attach.52pojie.cn/forum/202512/11/100252mpp6uht4zpvadnvr.png)
180202-1.png (31.21 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkzM3wyYmJkYjQ2YnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkzM3wyYmJkYjQ2YnwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
15
2025-12-11 10:02 上传
- 可以看到，`apply` 一个都没有，`call` 有12个，建议是在12个 `call` 位置全部都打上日志断点，6个在 `i` 方法，6个在 `a` 方法，也不是很多。唯一需要注意的点就是这个是 `++h` ，要注意打印日志的时候去减掉 ++ 的值，因为我们要看的是 `h`当时的执行结果。
![](https://attach.52pojie.cn/forum/202512/11/100256a6jkv2ja991726nj.png)
180217-1.png (51.05 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkzNHw0ZTdiYjRmMHwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkzNHw0ZTdiYjRmMHwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
16
2025-12-11 10:02 上传
- 打好日志断点后运行，看控制台输出的日志，现在是有1027条日志，接下来才是重头戏，我们先把日志全部都复制出来，简单看一下。
![](https://attach.52pojie.cn/forum/202512/11/100302kn7xwx2xpzn8xayp.png)
180228-1.png (102.77 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjgxOTkzNXxkN2E4ODNkMXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxOTkzNXxkN2E4ODNkMXwxNzkzNTcwMzA4fDE2ODA4NnwyMDc5MDcx&nothumb=yes)
17
2025-12-11 10:03 上传
