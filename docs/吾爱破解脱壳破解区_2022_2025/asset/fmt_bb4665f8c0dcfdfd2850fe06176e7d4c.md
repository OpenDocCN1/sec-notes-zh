
```
MOV EDX, 0BBAh
MOV RCX, RSI
CALL sub_xxxxxxxx
JMP createNewFolder
```

现在可以关掉 `Patching` 窗口了。这里调用了 [`InsertMenuW`](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-insertmenuw) 函数，因为比较简单一些。`0x0C00` 代表是分隔符，`EDX:=1` 代表第2个项目的位置插入（从0开始数）。然后 `NewItem` 的写法是参考了原位置下方的分隔符的写法。

在插入的代码第一行点一下，然后菜单里 `Edit -> Code` 点一下就转换为代码了。

![](https://attach.52pojie.cn/forum/202412/17/214048iwdiaxsvzasvqvdw.png)

`2024-12-17-[20-29-22].png`（324.77 KB，下载次数：0）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0OHwyMTNlZjVkZHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)

2024-12-17 21:40 上传

这个时候在代码第一行按 `N` 或者右键 `Rename`，输入一个 `Label` 方便上面跳转下来。我在这里输入 `addSeparator`。

![](https://attach.52pojie.cn/forum/202412/17/214106yh23ha98i8b3a0e0.png)

`2024-12-17-[20-36-15].png`（314.71 KB，下载次数：0）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM0OXxlMTZmNjk5NHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)

2024-12-17 21:41 上传

然后点两下 `JMP createNewFolder` 可以跳回刚才那里，或者用 `View -> Open subviews -> Bookmarks` 跳回刚才的书签也行。随后在刚才 `NOP` 掉的第一行右键 `Assemble...`，然后写上跳转到下方自己加的代码并回车：

```
JMP addSeparator
```

![](https://attach.52pojie.cn/forum/202412/17/214131qhrz6r6rr6g2bggc.png)

`2024-12-17-[20-49-57].png`（334.8 KB，下载次数：1）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM1MHxiMTg1MjU0NnwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)

2024-12-17 21:41 上传

然后就可以按 `F5` 查看修改后的反编译结果。

![](https://attach.52pojie.cn/forum/202412/17/214202zsfdb3koamptkttt.png)

`2024-12-17-[20-51-58].png`（391.75 KB，下载次数：1）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM1MXxlNDRkOWYwZnwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)

2024-12-17 21:42 上传

接下来是惯例的对比环节：

![](https://attach.52pojie.cn/forum/202412/17/214226ni2l9234ct72isf4.png)

`无标题2.png`（247.42 KB，下载次数：1）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM1M3wyYzUwNWI1OXwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy&nothumb=yes)

2024-12-17 21:42 上传

## 4. 总结

本次逆向中，第一次尝试进行了复杂代码的添加，而不是仅仅在原位改指令。之前尝试了很多方法，包括用 `Call` 和 `Return` 来处理控制流的重定向，但是会出很多问题，程序很容易跑飞。后来结合反编译才明白 `Call` 相当于函数的调用，而不仅仅是简单的指令指针跳转。会自动进行一些栈和寄存器方面的操作。（微机原理汇编学过的全忘了(‾◡◝)）

过程虽然比较艰辛，结果在很多朋友看来可能也是可有可无的，但我自认为在过程中的收获还是非常丰富的。而且也达到了自己当初设定的目标，算是比较圆满地完成了自己设定的任务了。

附这次的原文件（其实就是上一篇的成品）和本次的成品（密码 `52pojie`）：

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDM1NnxiMjM2ODY4OHwxNzkzNTcxNDY5fDE2ODA4NnwxOTkyMTMy)

（522.98 KB，下载次数：5）2024-12-17 21:47 上传

点击文件名下载附件

下载积分：吾爱币 -1 CB

[https://wwyp.lanzoul.com/iV8Am2idi24d](https://wwyp.lanzoul.com/iV8Am2idi24d)

再次感谢大家的鼎力支持！！感谢每一个看了帖子、评论了帖子的朋友(´▽`ʃ♡ƪ)
