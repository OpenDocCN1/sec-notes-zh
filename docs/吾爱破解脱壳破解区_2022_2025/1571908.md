
# [分享] 学破解第154天，《适合新手破解程序的中文字符串搜索法》总结

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1571908
- 抓取时间: 2026-11-01T23:42:52

## 前言：

坛友们，年轻就是资本，和我一起逆天改命吧，我的学习过程全部记录及学习资源：[//www.52pojie.cn/thread-1503734-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1503734-1-1.html:href="https)
立帖为证！--------记录学习的点点滴滴

### 0x1发现目标
1. 新手CM，给新手找回自信专用，大佬当没看见：[//www.52pojie.cn/thread-1571404-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1571404-1-1.html:href="https)
2. 看到这标题小菜鸟很开心，这应该搜字符串就能出来了吧？

### 0x2搜索目标
1. 看看入口信息
```
00403861 >    55            push ebp
00403862      8BEC          mov ebp,esp
00403864      6A FF         push -0x1
00403866  |.  68 F0624000   push 嘿嘿IE嘿.004062F0
0040386B  |.  68 D44C4000   push 嘿嘿IE嘿.00404CD4                                    ;  SE 处理程序安装
00403870  |.  64:A1 0000000>mov eax,dword ptr fs:[0]
00403876  |.  50            push eax                                               ;  kernel32.BaseThreadInitThunk
00403877  |.  64:8925 00000>mov dword ptr fs:[0],esp
```
2. 这不是易语言就是C++之类的，再去`OD`搜索字符串，看到
[//imgtu.com/i/Tjqsht" target="_blank" rel="noopener noreferrer nofollow"](https://imgtu.com/i/Tjqsht:rel="nofollow noopener" href="https)
3. `krnln.fnr`和`shell.fne`百度一下，这是易语言的库文件，判断程序是易语言写得了。

### 0x3易语言通杀法
1. 按钮事件`FF 55 FC`
2. 比较通杀特征码`test edx,0x3`
3. 纳尼，都不行，楼主，你给我出来，你这明明就是打击新手的。
4. 坛友们，可怜的小菜鸟感情就这样被欺骗了。

### 0x4字符串大法
1. 话说咋们这样的小菜鸟通用的是啥方法呀？字符串搜索嘛。
2. 看看原帖楼主给出破解成功的正确截图：
[//imgtu.com/i/TjOlJ1" target="_blank" rel="noopener noreferrer nofollow"](https://imgtu.com/i/TjOlJ1:rel="nofollow noopener" href="https)
3. 既然正确会显示“密码正确”，那么来到数据窗口搜索一下，搜不到？？？
4. 这不科学，新手CM难度都这么大吗？点`OD`上面的小写字母`m`，或者`Alt+M`打开内存窗口。
[//imgtu.com/i/TjXt10" target="_blank" rel="noopener noreferrer nofollow"](https://imgtu.com/i/TjXt10:rel="nofollow noopener" href="https)
5. 两个`data`段，`407000`和`409000`，鼠标双击`407000`进去，啥都没发现。
6. 鼠标双击`409000`进去，哇，小菜鸟发现了关键字符串：
[//imgtu.com/i/TjjS4s" target="_blank" rel="noopener noreferrer nofollow"](https://imgtu.com/i/TjjS4s:rel="nofollow noopener" href="https)
7. 看到了密码错误和密码正确的字符串，密码正确前面又看到了`flag`，[//www.52pojie.cn/forum.php" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/forum.php:href="https)

### 0x5验证Flag
1. 打开程序输入：[//www.52pojie.cn/forum.php，成功" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/forum.php，成功:href="https)！
[//imgtu.com/i/Tjj8bD" target="_blank" rel="noopener noreferrer nofollow"](https://imgtu.com/i/Tjj8bD:rel="nofollow noopener" href="https)

### 0x6总结
1. 我们新手碰到程序，看提示信息，然后搜索字符串。
2. 字符串搜索三法：右键搜索中文字符串，`Ctrl+B`搜索，`Alt+M`内存搜索。
