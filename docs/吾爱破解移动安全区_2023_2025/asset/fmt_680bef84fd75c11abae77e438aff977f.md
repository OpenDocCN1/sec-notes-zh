
# [Android 原创] 某打车平台Autojs脚本的逆向分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1883058
- 抓取时间: 2026-11-01T21:31:46

之前有位兄弟有偿求破一脚本卡密，说是某打车抢单的，因为没有接触过这种脚本的逆向，所以給看了看

## 第一步，先上`GDA`，看看有没有壳

![](https://attach.52pojie.cn/forum/202401/21/151615zd95cjmzrs9p5k5p.png)
image.png (294.83 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY3MTU1OXwzZTZmNmZlNnwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTU1OXwzZTZmNmZlNnwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes)
2024-1-21 15:16 上传

看到是没有壳，而且又得到一个信息，这是`Autojs`脚本

## 第二步，安装，看看界面（打码是为了不砸人饭碗，以后可能也在这个圈子混呢，哈哈哈）

![](https://attach.52pojie.cn/forum/202401/21/151853eyi7ppb67npqn7ny.png)
image.png (38.68 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=MjY3MTU2MXwwMTliN2MyOHwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTU2MXwwMTliN2MyOHwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes)
2024-1-21 15:18 上传

可以看到有卡密，这里有个点要注意下，像一般脚本去卡密到这步，可能要上`MT`管理器去搜关键字了，当然我也这样做了，但是`dex`里面根本搜不到什么有用的信息，而且看这个布局，如果写过`Autojs`脚本的应该可以猜到这个卡密模块是写在`js`里面的，这里用`MT`看看`js`脚本是不是有加密

![](https://attach.52pojie.cn/forum/202401/21/160347xjzz6i45sf8uq050.jpg)
8dda11ab94b5d931fddbaa168ce688d.jpg (186.15 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY3MTU2N3xmN2IwMWEyMHwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTU2N3xmN2IwMWEyMHwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes)
2024-1-21 16:03 上传

![](https://attach.52pojie.cn/forum/202401/21/160424hg0unq3033zamnb0.jpg)
a3409cf784eba18872a0273491b212a.jpg (366.21 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjY3MTU2OHxlOWYxNzQxZnwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTU2OHxlOWYxNzQxZnwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes)
2024-1-21 16:04 上传

可以看到是加密了，所以下一步，我们要`dump`里面的`js`脚本下来研究了

## 第三步，上`Frida`脱脚本

分析脱脚本的文章网上有很多，我也是直接拿的其它人的成果，这里不深入分析

先上代码，把加载的`js`代码先按照`byte`数组打印出来

```javascript
Java.perform(function(){
    function bytesToString(bytes){
        var str="{";
        for(var i = 0; i
```

再用`python`处理转为`js`

```python
arr=[39, 117, 105, 39, 59, 13, 10, 13, 10, 105, 109, 112, 114, 116,] # byte数组放这里

with open("jiaoben.js","w",encoding="utf-8") as f:
        for s in arr:
                f.write(chr(s&0xff))
```

这里有个坑需要提下，因为这个脱脚本的点是在打开`app`加载`js`脚本的时候，网上一些文章说是要先用`spawn`模式启动`frida`去`hook`，再启动`app`，但是我试了是不行的，这里还有个操作顺序：
1. 先启动`app`，等加载到主界面，不要操作其它
2. 电脑上运行命令：`frida -U -l hook.js app`，在手机上按返回键返回到桌面，这里很重要
3. 点击`app`启动，这里才算是`hook`到加载脚本的点，加载了多少脚本这里都会打印出来

![](https://attach.52pojie.cn/forum/202401/21/155103r1nlnltn76u766kt.png)
image.png (153.36 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=MjY3MTU2Nnw2NWExOGU4NXwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTU2Nnw2NWExOGU4NXwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes)
2024-1-21 15:51 上传

但是........，这里脱下的脚本，`main.js`里面的字符串没有解开，只能脱下`common.js`

![](https://attach.52pojie.cn/forum/202401/21/161130zmeak6ikekaipxm5.png)
image.png (47.1 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY3MTU2OXwyNjY1MWEwNHwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3MTU2OXwyNjY1MWEwNHwxNzkzNTY4NzAzfDE2ODA4NnwxODgzMDU4&nothumb=yes)
2024-1-21 16:11 上传

所以这里要上第二个脚本，操作步骤和上面一样，这个不需要转换`byte`，直接就是`js`代码

```javascript
```
