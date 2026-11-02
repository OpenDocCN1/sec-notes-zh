
# [原创] 弃用Ollydbg，转战x64dbg的10个理由：

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1573909
- 抓取时间: 2026-11-01T23:42:40

@anyint[//www.52pojie.cn/thread-1573471-1-1.html" target="_blank"](https://www.52pojie.cn/thread-1573471-1-1.html:href="https)

[//www.52pojie.cn/thread-350397-1-1.html" target="_blank" class="relatedlink"](https://www.52pojie.cn/thread-350397-1-1.html:href="https)诞生于2000年，曾经是很不错的调试器，和`W32dasm`被誉为屠龙刀和倚天剑，它继承了`softice`、`TRW2000`的很多特性，这个咱就不细说了。
而`x64dbg`诞生的年代就比较靠现在了，具体多少没查出来，这些咱也不细究了。还是少说费话说重点吧：

## Ctrl+N窗口对比

![](https://attach.52pojie.cn/forum/202201/10/094446z6uuvh2idlflzulf.png)
image.png (130.94 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjM2NDM5M3xjY2I1NjY1NnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDM5M3xjY2I1NjY1NnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 09:44 上传
`Ollydbg`的该窗口，你只能一个字母一个字母的键入； 这对于眼神不好的同学十分不便
哪怕你输入错了一位也不会找到
而且不支持过滤掉不想要的,键入的内容会在标题窗口上显示键入的字符
![](https://attach.52pojie.cn/forum/202201/10/094634vcc7z7jc7cims30g.png)
image.png (333.27 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjM2NDM5NHxjMTkzMWIyNnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDM5NHxjMTkzMWIyNnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 09:46 上传
再来看看`x32dbg`/`x64dbg`的该窗口吧。
左右两栏非常的清晰，想要系统`dll` 就键入过滤下（同理右边也是一样道理）
而且懂正则表达式的还可以来个模糊匹配
按下`Ctrl+` 上下滚轮放大 缩小 窗口功能
这个在`Ollydbg`中是完全不被支持的
`x32dbg`/`x64dbg`可就不一样了
虽然有不少同学看不习惯它的字体，但是通过热键，你可以解决
![](https://attach.52pojie.cn/forum/202201/10/095427hjjegqj34esz3ezq.png)
image.png (134.53 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjM2NDQwMXxhMmFlZjg0NnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQwMXxhMmFlZjg0NnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 09:54 上传

## 3A地址转换

即`VA`、`RVA`、`FOA`（虚拟地址/相对虚拟地址/文件偏移地址）
这个对于汇编工具来说极其重要的东西，比如你用`HIEW`/`W32dasm`/`IDA`/`2进制忍者`/`Ollydbg`/`MDebug`
16进制编辑器。你第一步得整明白，谁对应谁吧？这就好比问你家住哪里？
虚拟地址？ 好比说，某某小区xx地址
相对虚拟地址？好比说吾爱[//www.52pojie.cn" target="_blank" class="relatedlink"](https://www.52pojie.cn:href="https)大楼对过的
文件偏移地址？中国天津塘沽区xx。。。。
这样能更好的理解了吧？
再来看看它们都是各自如何解决的？
`Ollydbg`默认只能`Ctrl+X` 得到 `VA`地址
其他两个地址可以通过`viXX`插件来得到
原生是不支持的
![](https://attach.52pojie.cn/forum/202201/10/100603mhh6ylr4xh04hhlg.png)
image.png (50.13 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjM2NDQzMnwxODYxYjg2ZnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQzMnwxODYxYjg2ZnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:06 上传
再来看看`x32dbg`/`x64dbg`中的吧，这是法王设计的最合理的热键了
`Ctrl+X`是`VA`， 加上`SHIFT`就是相对 `RVA`， 另一个单独记一下就全部搞定了

## 字符串搜索

![](https://attach.52pojie.cn/forum/202201/10/100905e68sla9axl66nxkx.png)
image.png (84.95 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjM2NDQzOXxiY2M0YjgxNHwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQzOXxiY2M0YjgxNHwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:09 上传
`Ollydbg`中是这样的，通过`Ctrl+L`搜索下一条，搜索结果不能过滤掉不要的
![](https://attach.52pojie.cn/forum/202201/10/101023ugythhv1m9ovpvrw.png)
image.png (279.75 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjM2NDQ0MXwxNzE0YjA0OXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ0MXwxNzE0YjA0OXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:10 上传
`x32dbg`/`x64dbg`中是这样的
不但支持针对当前模块 或 所有模块 来搜索
还能对搜索结果过滤，并支持正则

## HIT跟踪和Run跟踪

![](https://attach.52pojie.cn/forum/202201/10/101224h05ujbjj8szaq5ic.png)
image.png (35.76 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjM2NDQ0Mnw5NGExMWIzOXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ0Mnw5NGExMWIzOXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:12 上传
`hit`跟踪就是哪里被运行过了，就是命中了
这个`x32dbg`/`x64dbg`是没有的，但是我们可以编脚本来模拟实现
Run跟踪呢？
![](https://attach.52pojie.cn/forum/202201/10/101519dbxxhbxkhohlbkbe.png)
image.png (56.46 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjM2NDQ0NHxkZmZkNzhhNnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ0NHxkZmZkNzhhNnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:15 上传
这种模式下，`Ollydbg`极其容易让调试器崩溃，具体就不表演功能了
来看看`x32dbg`/`x64dg`中是啥样的吧？
![](https://attach.52pojie.cn/forum/202201/10/101647mx8d2h8qjjjhj3wp.png)
image.png (30.74 KB, 下载次数: 6)
[href="forum.php?mod=attachment&aid=MjM2NDQ0NnwzOGJkNmMzZXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ0NnwzOGJkNmMzZXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:16 上传
来到`x32dbg`/`x64dbg`中，启动运行跟踪
并设好断点
![](https://attach.52pojie.cn/forum/202201/10/101739ozqhtketqqwn2w8a.png)
image.png (31.21 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjM2NDQ0N3w3NTAzYzNhMnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ0N3w3NTAzYzNhMnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:17 上传
它将存储到`.trace32` / `.trace64` 中
比如我们要把`Fscapture`点关于时断下
你不是得这么想？
![](https://attach.52pojie.cn/forum/202201/10/102229v5ntfiqett5ntchn.png)
image.png (132.51 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjM2NDQ1NHxlMTcyNzI2OXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ1NHxlMTcyNzI2OXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:22 上传
先找到注册成功 ，失败的字符串？
![](https://attach.52pojie.cn/forum/202201/10/102330lffs4icraspl9rj3.png)
image.png (158.25 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjM2NDQ1NXwxMjgwNTc5Y3wxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ1NXwxMjgwNTc5Y3wxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:23 上传
这里会出现一托托 ，或 很长的字符串 ；有经验的同学不用想，肯定就知道与注册算法有关
顶上埋个陷井吧，同理下面也埋上一个
![](https://attach.52pojie.cn/forum/202201/10/102416u4ltd4n0hqq47q70.png)
image.png (29.49 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjM2NDQ1Nnw0ZmM5MmMxNXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ1Nnw0ZmM5MmMxNXwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:24 上传
然后顺势运行就会断下
`Ctrl+F8`经过了比较长的运行
就会来到下一个字符串的调用处
```
006D3DED | B8 0C436D00 | mov eax,fscapture.6D430C | 6D430C:"ACjizQuiyoyJfUh4/b6b"
```
![](https://attach.52pojie.cn/forum/202201/10/102646y4gpgerpiaf1m34a.png)
image.png (140.28 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjM2NDQ1OXw2ODAzNWQzNnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ1OXw2ODAzNWQzNnwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:26 上传
然后呢，就会读取到你的注册时用到的注册名，注册码
公司网址。。。
最后是授权给哪一种类型的人士了。
![](https://attach.52pojie.cn/forum/202201/10/102821zbg2m2q2gubmq6s0.png)
image.png (141.64 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjM2NDQ2MHw5NWQxOTBlZHwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ2MHw5NWQxOTBlZHwxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:28 上传
然后呢，这个跟踪窗口就是一览无余的展示调用的地址
堆栈前后寄存器，相关信息
![](https://attach.52pojie.cn/forum/202201/10/102927skfh58wcrq4ggzcl.png)
image.png (208.33 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjM2NDQ2MXw5YzcxNWNiY3wxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NDQ2MXw5YzcxNWNiY3wxNzkzNTc2NTU3fDE2ODA4NnwxNTczOTA5&nothumb=yes)
2022-1-10 10:29 上传
![](https://attach.52pojie.cn/forum/202201/10/103047r936z9ip74zmo7fj.png)
image.png (401.36 KB, 下载次数: 4)
