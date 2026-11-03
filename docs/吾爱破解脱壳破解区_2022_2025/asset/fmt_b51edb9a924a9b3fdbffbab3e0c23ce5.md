
```
0050C21C  |.  B8 03000000   mov eax,3
0050C23F  |.  B8 04000000   mov eax,4
0050C302  |.  33C0          xor eax,eax
0050C4D1  |.  B8 02000000   mov eax,2
0050C592  |.  33C0          xor eax,eax
```

至此，知道了`eax`的情况，大致有：0、2、3、4、5、6。

下面，先试试不调用这个子程序情况能否运行？即将`call`改成直接跳过。结果是程序中断。

![h3-15.png](https://attach.52pojie.cn/forum/202409/02/191239ajgrn23dgb1m6rgz.png)

`h3-15.png` (114.01 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI2N3wxNGRhYThmNXwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:12 上传

既然直接跳过不行，那就直接给`eax`赋值。先试试1。

![h3-16.png](https://attach.52pojie.cn/forum/202409/02/191321idyem5yatdq7yada.png)

`h3-16.png` (105.42 KB, 下载次数: 0) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI2OHw1OTVmNDllYnwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:13 上传

结果还是出现没有CD的提示，说明1不正确。再试试7，结果成功了。

![h3-17.png](https://attach.52pojie.cn/forum/202409/02/191404pqnc3k64bzq83inz.png)

`h3-17.png` (118.21 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI2OXwzOTNlOWE3ZHwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:14 上传

再试试8，也是可以的。

![h3-18.png](https://attach.52pojie.cn/forum/202409/02/191439xbm7z64g3951553b.png)

`h3-18.png` (104.63 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI3MHw3OTAzZTU3NnwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:14 上传

说明大于6的返回值都是可行的。至此，免CD基本成功了。

## 三、不足之处

这样免CD后，由于游戏中有的资源（如：战役模式中的动画，应该是存在光盘上的，所以只有文字而没有动画了，还有就是失败后，被斩首的动画也没有）是存在光盘上，涉及这些资源的游戏调用，自然就调用不到了。

## 四、hex编辑软件直接修改

如果不想使用OD的，也可以直接使用hex类的编辑软件如：010Editor，进行直接修改。从前面的图片可知，要修改的命令是在`004ED9A6`地址的`call 0050C1C0`，将其改为：`mov eax,7`即可。

下面用010Editor打开`heroes3.exe`，选search工具栏里的`go to`。

![h3-19.png](https://attach.52pojie.cn/forum/202409/02/191707zzb29ddg9yzbygyy.png)

`h3-19.png` (160.91 KB, 下载次数: 0) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI3MXw1ZWMwYjE2ZnwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:17 上传

在地址栏里输入`000ED9A6`，这个就是`call 0050C1C0`命令行的偏移地址，再回车。

![h3-20.png](https://attach.52pojie.cn/forum/202409/02/191804y8lad0678uxmxuuu.png)

`h3-20.png` (137.42 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI3Mnw4ZWZhM2Q1ZXwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:18 上传

来到这里，其中：`E8 15 E8 01 00`正是`call 0050C1C0`。

![h3-21.png](https://attach.52pojie.cn/forum/202409/02/191841f0iao9vlr3rr9ivv.png)

`h3-21.png` (94.37 KB, 下载次数: 0) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI3M3w0ODY2ZTQ1ZnwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:18 上传

将其改成：`B8 07 00 00 00`，即`mov eax,7`即可，保存的程序就是免CD的了。

![h3-22.png](https://attach.52pojie.cn/forum/202409/02/191914rbyzjy1ctkhg11xz.png)

`h3-22.png` (92.06 KB, 下载次数: 1) · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMTI3NHxiNTg1YzI3MnwxNzkzNTcxODY1fDE2ODA4NnwxOTYwNjg3&nothumb=yes) · 2024-9-2 19:19 上传

## 五、资源下载

《英雄无敌》3complete版（英文）下载：<https://www.123pan.com/s/7jpKVv-7bjCh>

《英雄无敌》3complete版免CD修改后的程序和win11可以直接运行的HD补丁下载：<https://www.123pan.com/s/7jpKVv-YZjCh>
