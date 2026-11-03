
2024-10-8 06:57 上传

在程序的尾部有很多空白，选定0048B0C0处作为新的入口点

![](https://attach.52pojie.cn/forum/202410/08/065847ktmmqgzg31xm311x.png)

h1-2.png (39.64 KB, 下载次数: 2)

2024-10-8 06:58 上传

先在0047B0A4的地方输入字符串`Towngate.dll`，这是dll的文件名

![](https://attach.52pojie.cn/forum/202410/08/065927arlsbg5x8054f3as.png)

h1-3.png (73.08 KB, 下载次数: 1)

2024-10-8 06:59 上传

再在0047B0B1的地方输入`GetTown`，这是dll中的输出函数名

![](https://attach.52pojie.cn/forum/202410/08/070022k5xwj2w253tj15kg.png)

h1-4.png (85.44 KB, 下载次数: 1)

2024-10-8 07:00 上传

好，下面就可以按照上面的三步注入dll。在0048B0C0处，依次输入上面三步，结果如下图

![](https://attach.52pojie.cn/forum/202410/08/070107f66pz5k9mnt70nm6.png)

h1-5.png (85.69 KB, 下载次数: 0)

2024-10-8 07:01 上传

保存修改

![](https://attach.52pojie.cn/forum/202410/08/070145p4cpx3pvyihzyiiy.png)

h1-6.png (151.26 KB, 下载次数: 1)

2024-10-8 07:01 上传

用`CFF Explorer`打开刚修改后的程序，可以看到此时程序的入口点是在826C0处（即4826C0）

![](https://attach.52pojie.cn/forum/202410/08/070226gaecadujskckmvjd.png)

h1-7.png (65.67 KB, 下载次数: 2)

2024-10-8 07:02 上传

将其修改为8B0C0，并保存

![](https://attach.52pojie.cn/forum/202410/08/070324erbrdd5ogqur3wyq.png)

h1-8.png (72.19 KB, 下载次数: 1)

2024-10-8 07:03 上传

运行一下游戏，看看是否可以正常运行。可以正常运行，说明入口点的修改没有问题，下面就是修改程序代码的过程了。

## 四、回城术代码研读

下面是关键部分，也就是回城术是怎么选择城堡的

[Asm] 纯文本查看 复制代码
