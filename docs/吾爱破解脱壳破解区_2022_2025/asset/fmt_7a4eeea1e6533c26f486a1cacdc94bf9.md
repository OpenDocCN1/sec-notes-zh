
# [.NET逆向] Mecway 14.0 x64 Patch分析

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1654786`
- 抓取时间: `2026-11-01T23:14:46`

目标软件：`Mecway 14.0`，干啥的么。。。不关心~

程序信息：`.NET v4.0`、无壳、x64；

官网：[//mecway.com/" target="_blank"](https://mecway.com/:rel="nofollow noopener" href="https)

能力有限，只做爆破分析

![](https://static.52pojie.cn/static/image/smiley/laohu/laohu35.gif)

打开软件，收集试用信息，标题栏上有“free version”字样，“Product Activation”界面显示“Free edition: Cannot solve, save or run scripts on models with more than 1000 nodes。”：

![](https://attach.52pojie.cn/forum/202206/27/185504zwqn4jw4k42xwzsi.png)
Snipaste_2022-04-11_12-27-24.png (61.39 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg4NXxjNTM0NDdmOHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg4NXxjNTM0NDdmOHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:55 上传

`dnspy`载入静态分析一下；搜索字符串“free version”，只有一处：

![](https://attach.52pojie.cn/forum/202206/27/185535bx1mko5oqxc5qczl.png)
Snipaste_2022-04-11_12-29-55.png (91.2 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg4Nnw5YWM0ZTA2M3wxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg4Nnw5YWM0ZTA2M3wxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:55 上传

双击进入，来到“`mecway.exe`”模块中，可以看到这里有一些与授权有关的信息：

![](https://attach.52pojie.cn/forum/202206/27/185554m99xlbcxxcz8xbd8.png)
Snipaste_2022-04-11_12-31-14.png (30.46 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg4N3wxYjZlN2I4MHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg4N3wxYjZlN2I4MHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:55 上传

向上翻看代码，注意到`License.EditionValue editionValue = License.Edition(ref i);`处，单击点开`Edition`查看：

![](https://attach.52pojie.cn/forum/202206/27/185614ultg7y771hyg51ss.png)
Snipaste_2022-04-11_12-32-07.png (26.76 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg4OHw0ZGEzYzkzM3wxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg4OHw0ZGEzYzkzM3wxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:56 上传

推测该段代码在`if (alicense2.VersionOk(ref flag))`语句中根据`num`值（即`alicense.Features`）来控制版本，当`alicense.Features`值为20（即`0x14`）时，版本为Full；由此初步确认`VersionOk`需要返回true且`alicense.Features == 0x14`：

![](https://attach.52pojie.cn/forum/202206/27/185723h58c0bkzbid8w8w8.png)
Snipaste_2022-04-11_12-33-55.png (21.43 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg4OXxhZWVmM2E3ZXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg4OXxhZWVmM2E3ZXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:57 上传

点开`VersionOk`，该段代码为switch-case结构，当switch值为0或2时，返回false，故switch中的`Verify`需要返回1，执行`result = !this.Test3()`才有可能返回true：

![](https://attach.52pojie.cn/forum/202206/27/185835s6a0q7ji6di0itji.png)
Snipaste_2022-04-11_12-35-46.png (15.94 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5MXwwMTgxODk2OHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5MXwwMTgxODk2OHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:58 上传

继续点开`Verify`，该段代码在判断`ProductKey`不为空后对`ProductKey`进行验证，返回0或1：

![](https://attach.52pojie.cn/forum/202206/27/185929m2qoxomdm52m5p35.png)
Snipaste_2022-04-11_12-36-33.png (22.5 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5Mnw4OTk1ZTM5MHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5Mnw4OTk1ZTM5MHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:59 上传

![](https://attach.52pojie.cn/forum/202206/27/185944ok2j2skjl7q27k22.png)
Snipaste_2022-04-11_12-38-45.png (18.75 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5M3xlOGQ3NzI3M3wxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5M3xlOGQ3NzI3M3wxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 18:59 上传

需要注意的是软件会在其它位置取Key的长度进行运算（例如`VersionOk`下执行的`this.Test3()`）：

![](https://attach.52pojie.cn/forum/202206/27/190006gj4v57dnm757l8iu.png)
Snipaste_2022-04-11_12-41-21.png (18.35 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5NHwxYjRkMTAwZHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5NHwxYjRkMTAwZHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:00 上传

所以直接在`Verify`段首返回1会导致程序崩溃；

考虑到主程序未加壳无混淆，可以尝试在`Verify`中添加代码固定Key；代码处右键——编辑方法：

![](https://attach.52pojie.cn/forum/202206/27/190109kot7q3qv8xt4rokq.png)
Snipaste_2022-04-11_12-43-44.png (56.45 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5NnxiOTI0MjNhNXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5NnxiOTI0MjNhNXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:01 上传

添加代码：
```
this.ProductKey = "111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111";
this.Features = 0x14;
```

单击`编译`后报错，错误提示为`ALicense.Features`只读：

![](https://attach.52pojie.cn/forum/202206/27/190153oao22z00z60q6cjr.png)
Snipaste_2022-04-11_12-45-30.png (51.55 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5N3xhODhlZTExYXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5N3xhODhlZTExYXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:01 上传

这里可以曲线救国一下，利用其它局部变量间接修改。

段首下断，`F5`跑起，断下后“局部变量”中可以看到`Features`值无法编辑：

![](https://attach.52pojie.cn/forum/202206/27/190222v8h25r2555j5yr78.png)
Snipaste_2022-04-11_12-50-07.png (162.16 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5OHxmMDc2MzUwZXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5OHxmMDc2MzUwZXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:02 上传

继续向下查看局部变量，注意到`_Features`值是可以编辑的：

![](https://attach.52pojie.cn/forum/202206/27/190247azfezhmctcf4vvir.png)
Snipaste_2022-04-11_12-50-32.png (149.18 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjg5OXwyMzNjNTQ2OHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjg5OXwyMzNjNTQ2OHwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:02 上传

修改`_Features`值为`0x14`后再查看`Features`值同步更改为`0x14`：

![](https://attach.52pojie.cn/forum/202206/27/190308u9eesss57zks7qnk.png)
Snipaste_2022-04-11_12-51-02.png (157.96 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjkwMHxkMDg3ZTIxOXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjkwMHxkMDg3ZTIxOXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:03 上传

所以需要添加的代码为：
```
this.ProductKey = "111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111";
this._Features = 0x14;
```

再将返回值`result`两处0修改为1：

![](https://attach.52pojie.cn/forum/202206/27/190333inos556snzay17dl.png)
Snipaste_2022-04-11_12-59-22.png (42.74 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjUzMjkwMXwxNDJmNGE0MXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMjkwMXwxNDJmNGE0MXwxNzkzNTc0ODg0fDE2ODA4NnwxNjU0Nzg2&nothumb=yes)
2022-6-27 19:03 上传

编译正常，保存文件，最终实现效果：

![](https://attach.52pojie.cn/forum/202206/27/190351oq5xhaa7hbx0qhrt.png)
Snipaste_2022-04-11_12-59-25.png (63.45 KB, 下载次数: 0)
