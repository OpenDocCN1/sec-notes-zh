
# [分享] 某游戏1代win版免CD

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1946484>
- 抓取时间：2026-11-01T22:27:07

## 修改关键跳转实现免CD

某游戏是95年的，但在2000年出了千禧版，加上由 Verok 的 HD 补丁，使得其在最新的 Win 系统如：Win11、Win10 上都可以完美运行，很受我的喜欢，该游戏可以选择一个地图，没事玩一下可能 1、2 个小时，很能打发时间。游戏要 CD，这个每次还得加载虚拟光驱添加 `iso` 不是很方便，故修改一下使其免 CD。

使用 OD 打开游戏

![](https://attach.52pojie.cn/forum/202407/22/144109mmp20q9d9mv09hm9.png)

[`1.png`（171.29 KB，下载次数：1）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDMzNnw3ZGVhNWFiM3wxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 14:41 上传

点击字符串搜索

![](https://attach.52pojie.cn/forum/202407/22/144220uqqo74cnoos00ycy.png)

[`2.png`（195.54 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDMzN3wyZmJjNDBmNXwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 14:42 上传

找到有关 CD 和 CD-ROM 的字符串，如上图看到 `4500A9` 和 `4500D5` 两处，回到 CPU 窗口，右键选择 `go to`，再选表达式，其中填上 `4500A9`。

![](https://attach.52pojie.cn/forum/202407/22/144809gspwl0zl4lsfxzxu.png)

[`3.png`（149.27 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDMzOXxiMDMyMjY2M3wxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 14:48 上传

很容易看出，下图中的两处条件跳转就是免 CD 的关键跳转，只要避开两处跳转下面的两个关键 `call` 就能实现免 CD 了。

![](https://attach.52pojie.cn/forum/202407/22/145138zo1i55kkc5ccmu15.png)

[`4.png`（185.31 KB，下载次数：1）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM0NHxkZTEyNzdmNXwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 14:51 上传

这样就修改这两处关键跳转。很简单地看到，只要把条件跳转改成无条件跳转即可。于是在 `4500C` 上右键选择 `Binary->Edit`。

![](https://attach.52pojie.cn/forum/202407/22/145455qk42b3l6kk9rj6gu.png)

[`5.png`（213.94 KB，下载次数：1）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM1MHwxOWZmYmI2MXwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 14:54 上传

出现下面对话框

![](https://attach.52pojie.cn/forum/202407/22/145604ozzp700pxvpp4ijv.png)

[`6.png`（139.9 KB，下载次数：2）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM1MXwwZTA0Zjg3MnwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 14:56 上传

将其改成下面这样（感谢 @爱飞的猫 朋友提醒！这里原指令长度是 6，现在改为 5 了，所以后面跳转长度要加 1，因此是：`E9 23 00 00 00 90`，错误的截图已修正）

![](https://attach.52pojie.cn/forum/202407/23/044558knnjhmn7nnhbsj65.png)

[`1.png`（213.17 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDU2OXwzNzk2ZDI5ZHwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-23 04:45 上传

点击 OK 后，变成下图样式

![](https://attach.52pojie.cn/forum/202407/23/044621eb8kyzchac78ib1i.png)

[`2.png`（160.46 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDU3MHw4Y2YyYzEyNnwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-23 04:46 上传

同样地，再把 `4500C8` 这句也改成（和上面完全相同）如下样子

![](https://attach.52pojie.cn/forum/202407/23/044641taoia8n1knaova8l.png)

[`3.png`（147.97 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDU3MXxmYzYwNzUzOXwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-23 04:46 上传

至此，两个关键跳转已经修改完成，应该能免 CD 了，保存修改

![](https://attach.52pojie.cn/forum/202407/23/051005smgnqnz8eyunuckq.png)

[`4.png`（186.79 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDU3M3wzYTFiMTE1YnwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-23 05:10 上传

## 恢复开始动画

运行新保存的游戏就不需要 CD 了，但这样修改后，运行游戏时是没有开始动画的。下面用 hexedit 类的软件修改这个文件使其具有开始动画。用 010editor 打开刚修改后的文件

![](https://attach.52pojie.cn/forum/202407/22/150657wflikcc9eow1l6ne.png)

[`13.png`（90.23 KB，下载次数：1）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM1NnwyODZmMTRkN3wxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 15:06 上传

选择 search 工具栏的 `find`，调整为选择 ASCII 码，在搜索框里输入 `\Anim`，因为该游戏的动画是放在这个目录里的

![](https://attach.52pojie.cn/forum/202407/22/150955azig8rmjjufv8hmk.png)

[`14.png`（89.79 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM1N3xjM2M0ZTdjMXwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 15:09 上传

搜到这里

![](https://attach.52pojie.cn/forum/202407/22/151032zc62a0xo2969z1ro.png)

[`15.png`（85.15 KB，下载次数：1）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM1OHw4ZTBkNzllNnwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 15:10 上传

将其改成下图样子（即：游戏本来的动画是存于 CD 上的，现在将其改到存放到游戏目录里）

![](https://attach.52pojie.cn/forum/202407/22/151213a27yz73s7f7fsfy1.png)

[`16.png`（33.59 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM1OXxhNzYxMTg5OXwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 15:12 上传

最后，将游戏 CD 里的 `anim` 目录的内容和目录一起拷贝到游戏目录，如下图所示

![](https://attach.52pojie.cn/forum/202407/22/151348wvb71v71eesrvkew.png)

[`17.png`（36.45 KB，下载次数：0）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxMDM2MHw2YTIwNTdiZnwxNzkzNTcyMDI1fDE2ODA4NnwxOTQ2NDg0&nothumb=yes)
2024-7-22 15:13 上传

至此，再游戏就不要 CD 了，还有开机动画。
