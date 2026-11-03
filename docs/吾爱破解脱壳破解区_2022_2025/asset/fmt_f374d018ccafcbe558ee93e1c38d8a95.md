
# [原创] 新手学破解记录贴【160个CrackMe】之006

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1987065
- 抓取时间: 2026-11-01T22:18:55

本人刚刚入门汇编逆向分析，本帖将记录我对160个CrackMe程序中的第006程序的分析过程。在学习的过程中，我会边学边记录。如果有不正确之处，还请各位指正。

程序打开后有一个帮助按钮、一个不可用的`ok`键和一个清除键，点击完帮助按钮会弹4个弹窗，大概意思就是说：输入正确的用户名和密码让`ok`键从禁用状态变成可用状态，然后在让清除键和`ok`键隐藏把下面的logo完整显示出来。

![](https://attach.52pojie.cn/forum/202411/30/155649aj4rjjagzj3xx4zl.png)
image.png (45.99 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc0MDQ0MXw4OTk5OWUwY3wxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ0MXw4OTk5OWUwY3wxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 15:56 上传

查壳后是无壳 Delphi编写。

![](https://attach.52pojie.cn/forum/202411/30/155939ponjkjx6wx1gjglw.png)
image.png (116.04 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc0MDQ0MnxlMjMzN2ZiMnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ0MnxlMjMzN2ZiMnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 15:59 上传

Delphi编写的程序直接查一下都有哪些事件。

![](https://attach.52pojie.cn/forum/202411/30/160254r9iil3i55ud66fdi.png)
image.png (164.25 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc0MDQ0NHxmMjVjZjM3MXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ0NHxmMjVjZjM3MXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:02 上传

这个程序就不进行爆破了，我们来直接研究它的算法，首先`OK`按钮点不了，应该是输入用户名和密码的时候会进行验证，验证正确了`OK`按钮才能恢复正常。按照这个思路先把用户名和密码输入事件下上断点。

![](https://attach.52pojie.cn/forum/202411/30/160607nm51jzrmrrbsmspw.png)
image.png (33.92 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ0NXxmOTJmMjY4NnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ0NXxmOTJmMjY4NnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:06 上传

然后输入用户名和密码。先看用户名输入事件的代码，单步跟一下。

![](https://attach.52pojie.cn/forum/202411/30/160857tfimkn9zvdmd59rz.png)
image.png (275.2 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ0Nnw1ZWE3MzcyN3wxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ0Nnw1ZWE3MzcyN3wxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:08 上传

跟到这个位置发现如果`eax+0x47`的地址值不为0就不会跳转，通过直接改标志位让这个跳转不成立进去后发现下面的`CALL`是恢复`OK`按钮使用。也就是说当`eax+0x47`不等于0就能恢复`OK`按钮的使用。

![](https://attach.52pojie.cn/forum/202411/30/161033gs4ukroauz8zm4ks.png)
image.png (83.49 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ0OHw4ZjBjMjFiZHwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ0OHw4ZjBjMjFiZHwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:10 上传

继续往下跟会发现还有一个跳转，如果跳转不成立那么`ok`按钮也会变成可用状态，这个跳转上面有一个关键`CALL`决定这个跳转是成立还是不成立，这个`CALL`在上面是一些取出用户名、密码作为参数给这个关键`CALL`进行传参。由此就可以看出这个关键`CALL`可能是算法`CALL`。

![](https://attach.52pojie.cn/forum/202411/30/161507ud2ng9fzd79dl0o7.png)
image.png (101.42 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ1NHwwMDQxYjEzNnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ1NHwwMDQxYjEzNnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:15 上传

跟进这个关键`CALL`，我们发现这有一个跳转，如果输入的用户名长度小于等于5那就直接出`CALL`了。

![](https://attach.52pojie.cn/forum/202411/30/162249elrssk1zar29l2la.png)
image.png (179.51 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ1NXxkMWY5NDc0ZXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ1NXxkMWY5NDc0ZXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:22 上传

当用户名大于5个的时候可以顺利走到这个位置，看到了一些计算操作。

![](https://attach.52pojie.cn/forum/202411/30/162647pt60iccd077xcw2v.png)
image.png (73.05 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ1N3wzNGY0YTQ5Y3wxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ1N3wzNGY0YTQ5Y3wxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:26 上传

[Asm]
```
00442A93 | 8B4D FC                        | mov ecx,dword ptr ss:[ebp-4]            | 取用户名给 ECX
00442A96 | 0FB64C01 FF                    | movzx ecx,byte ptr ds:[ecx+eax-1]       | 把用户名第一个字节给ecx 每次循环都递增一个
00442A9B | 8B75 FC                        | mov esi,dword ptr ss:[ebp-4]            | 取出用户名在给ESi
00442A9E | 0FB63406                       | movzx esi,byte ptr ds:[esi+eax]         | 把用户名第二个字节给ESI 每次循环都递增一个
00442AA2 | 0FAFCE                         | imul ecx,esi                            | ecx*esi
00442AA5 | 0FAFC8                         | imul ecx,eax                            | ecx*eax
00442AA8 | 03D9                           | add ebx,ecx                             | 最后算出来的结果 累加到ebx中
00442AAA | 40                             | inc eax                                 |
00442AAB | 4A                             | dec edx                                 |
00442AAC | 75 E5                          | jne along3x.1.442A93                    |
```

通过这算法可以看出，就是把用户名的第一个字节乘以第二个字节再乘以当前循环的次数。

正向代码：

[C++]
```
        int ecx = 0;
        int esi = 0;
        int ebx = str.length();
        for (int i = 0; i 
```

![](https://attach.52pojie.cn/forum/202411/30/164155x7how252h7qmm0kc.png)
image.png (117.02 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0MDQ1OXwwZmI4ODk3MXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ1OXwwZmI4ODk3MXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:41 上传

编写一下对应算法，在程序上面输入验证了，算法是正确的，`OK`键现在可以正常使用了。

![](https://attach.52pojie.cn/forum/202411/30/164744bat7itnazonna4qx.png)
image.png (116.98 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ2MXwxNmZlNTdiMHwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ2MXwxNmZlNTdiMHwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:47 上传

现在再来点一下`OK`按钮，发现清空了密码输入内容（由于清空密码输入框改变了内容导致密码不正确`ok`按钮又变回不可用状态）

![](https://attach.52pojie.cn/forum/202411/30/164942fqd6d6wi8qe5q598.png)
image.png (18.15 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0MDQ2Mnw5ODIwOGFlZnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ2Mnw5ODIwOGFlZnwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 16:49 上传

目前知道了刚才的算法只是能让`OK`恢复使用，并不能把它隐藏掉。那继续去进入`OK`的点击事件里面找一下有没有对应的隐藏算法。

进入到`OK`按钮的点击事件当中，发现有一个关键的比较`cmp byte ptr ds:[eax+47],1`，如果等于`0x1`那么直接就把`OK`按钮禁用了，然后一个跳转就出`CALL`了。所以这个`eax+0x47`是关键，必须要让他不等于`0x1`才可以。`OK`按钮点击事件现在就不用看了，因为`eax+0x47`等于`0x1`那它就直接出`CALL`了，所以再看最后一个清除按钮的事件。

![](https://attach.52pojie.cn/forum/202411/30/190051akmkongguopqveob.png)
image.png (108.26 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc0MDUyM3w2YTcyY2Q5NXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDUyM3w2YTcyY2Q5NXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 19:00 上传

断下来后发现有跳转，上面的`CALL`里面传入了用户名进行运算决定跳转还是不跳转，推测也是一个算法，具体干什么用的还不清楚，跟进去看一下。

![](https://attach.52pojie.cn/forum/202411/30/170036atjkazto2ptsyhyt.png)
image.png (220.12 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MDQ2N3wyOTA5ODA4ZXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MDQ2N3wyOTA5ODA4ZXwxNzkzNTcxNTMyfDE2ODA4NnwxOTg3MDY1&nothumb=yes)
2024-11-30 17:00 上传

跟进去之后确实看到有一些计算的操作，就不一行一行注释了，大概的操作流程就是先判断密码长度是否小于等于`0x5`，如果小于跳出，然后取用户名的第四个字节和`0x7`进行取模运算，得出来的结果在加`0x2`，计算出来的结果经过下面的`CALL`进行阶乘。阶乘出来的结果保存在`esi`中备用，然后走到下面的循环，循环就是用`esi`的值依次和用户名的每一个字符相乘的和值保存到`ebx`当中。然后`ebx`的值减去密码的16进制等于`0x7A69`就算成立。
