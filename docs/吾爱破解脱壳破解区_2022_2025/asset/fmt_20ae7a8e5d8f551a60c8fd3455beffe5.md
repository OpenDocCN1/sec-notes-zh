
# [原创] 水印大作战

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1725831
- 抓取时间：2026-11-01T23:03:46

如图所示：已知：某款 Delphi 控件；随便编译一个 `EXE`；输出就会

![](https://attach.52pojie.cn/forum/202212/12/150108sg4gdhd4g4on13g8.png)

`image.png` (122.71 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODY4MHw5YWY3OTZhNXwxNzkzNTc0MjI0fDE2ODA4NnwxNzI1ODMx&nothumb=yes)

2022-12-12 15:01 上传

## 症状

症状1：全屏都是大水印；

症状2：偶尔弹框 Thank you for evaluating the

求证：去掉限制；找到对应的控件相关文件一次给多个版本的所有文件打补丁。

![](https://attach.52pojie.cn/forum/202212/12/150131i366lgy86bfvrcyr.png)

`image.png` (362.91 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODY4MXw5YzVkNWViZHwxNzkzNTc0MjI0fDE2ODA4NnwxNzI1ODMx&nothumb=yes)

2022-12-12 15:01 上传

## 调试

再到这个 `EXE` 文件中用 `winhex` 修改 1 个字节，看是否被调用了？是的。变了吧？

接下来就可调试了。

第 1 处的位置没难度，直接爆破 `JMP` 跳过就可以了。

注意看红线，默认本次是跳过的，但是有时则不会，所以安全起见 `jmp` 伺候

接下来水印限制就是考你基本功和经验值的问题了。

通常定位到该字符串下个硬件访问断点

![](https://attach.52pojie.cn/forum/202212/12/150226rx02aqqyzq07qad2.png)

`image.png` (359.73 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODY4MnwzNGNhMDA3MXwxNzkzNTc0MjI0fDE2ODA4NnwxNzI1ODMx&nothumb=yes)

2022-12-12 15:02 上传

注意看成功断下了，但是已经晚了八春了。

为啥会这样呢？

- 因为产生此水印的文件有好多个（`*.bpl`；`*.dcu`；`*.obj`），会产生干扰因素
- 时机晚了！为啥呢？因为是动态渲染出来的

![](https://attach.52pojie.cn/forum/202212/12/150037zznpdfnzyleh6knd.png)

`image.png` (298.11 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODY3OXw0ZDRjMWQzMXwxNzkzNTc0MjI0fDE2ODA4NnwxNzI1ODMx&nothumb=yes)

2022-12-12 15:00 上传

## 正确解法

所以正确的解法是：

看到了吧？水印就是这么来的。

`Ctrl+A`、`Ctrl-`

图中的代码很很长很长，只粘最有意义的部分：

[Asm] 纯文本查看 复制代码
