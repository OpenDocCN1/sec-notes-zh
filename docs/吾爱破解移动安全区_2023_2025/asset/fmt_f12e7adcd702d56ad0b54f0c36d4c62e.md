
[href="forum.php?mod=attachment&aid=MjY2MDI5NXw5ZTg4MDg3ZHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDI5NXw5ZTg4MDg3ZHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:27 上传

这样这两个方法都返回false了，所以这个方法也会返回false，也就是过了root检测了，正常root闪退手机就能进去了

![](https://attach.52pojie.cn/forum/202311/29/123101kzuxr09mtur9pxex.png)
`image-20231128162650538.png` (248 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY2MDMwM3xjYzhlOGRjNXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwM3xjYzhlOGRjNXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:31 上传

## 过虚拟机检测

（1）过完root检测我重新打包安装并打开该软件，发现依旧闪退，而且闪退位置依旧是那个位置，这不得不怀疑是第二个`finish`

![](https://attach.52pojie.cn/forum/202311/29/123148qvy0ngncmnim3nvf.png)
`image-20231128163224874.png` (190.71 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY2MDMwNHwyZmNjYjM3YXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwNHwyZmNjYjM3YXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:31 上传

（2）如之前操作并跳转第二个判断方法

![](https://attach.52pojie.cn/forum/202311/29/123219s6t7opn4uxqpm4p7.png)
`image-20231128163340904.png` (281.06 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMwNXxkMDgxNDliOHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwNXxkMDgxNDliOHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:32 上传

发现有多个判断条件

![](https://attach.52pojie.cn/forum/202311/29/123248wtcthhzw00iw4tao.png)
`image-20231128163413564.png` (281.2 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMwNnw2YzQwY2M1YnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwNnw2YzQwY2M1YnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:32 上传

让我们逐个查看

首先看第一个

![](https://attach.52pojie.cn/forum/202311/29/123307tbmwkfa4woatq5m6.png)
`image-20231128163523807.png` (266.48 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMwN3wzZWNjNGMxMXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwN3wzZWNjNGMxMXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:33 上传

该方法首先检查了一系列与模拟器相关的特征，例如设备的型号（`Build.MODEL`）、制造商（`Build.MANUFACTURER`）、硬件信息（`Build.HARDWARE`）、产品信息（`Build.PRODUCT`）、串号（`Build.SERIAL`）等。如果其中任何一个特征满足模拟器的特征，则将标记为模拟器环境，并且返回true，这个很明显是一个虚拟机判断的方法。

我们接着看下面几个方法

![](https://attach.52pojie.cn/forum/202311/29/123324sujx79gvd8lrl3r8.png)
`image-20231128164117363.png` (18.03 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY2MDMwOHxkNzAyNzljMnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwOHxkNzAyNzljMnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:33 上传

这个方法是用于检测特定条件的私有Java方法。

![](https://attach.52pojie.cn/forum/202311/29/123326a0ygy79il5ss577z.png)
`image-20231128164205042.png` (16.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMwOXw2NDYzODRlY3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMwOXw2NDYzODRlY3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:33 上传

这个方法是用来检查设备是否支持计步器传感器。

![](https://attach.52pojie.cn/forum/202311/29/123328a1n1nk19bt58e94i.png)
`image-20231128164015915.png` (21.82 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMxMHw3MDkzOWJjMXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxMHw3MDkzOWJjMXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:33 上传

这个方法是用于检查当前设备的 UI 模式是否为汽车模式。

这些方法通过大致判断都是用于检测你现在的环境是否是真机环境，不是真机环境将会闪退。

所以我们依旧进入MT管理器进行修改这些方法返回false值，实现过虚拟机检测

![](https://attach.52pojie.cn/forum/202311/29/123400jxxq5xm4tp66tmp5.png)
`image-20231128170831634.png` (177.49 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMxMXxmMDc1Mzk1M3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxMXxmMDc1Mzk1M3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

剩下一个布尔值的方法，直接跳转

![](https://attach.52pojie.cn/forum/202311/29/123403hitqyg393ezgsivz.png)
`image-20231128171122214.png` (171.84 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMxMnxkNDJkMDk4Y3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxMnxkNDJkMDk4Y3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

将这几个都改成false

![](https://attach.52pojie.cn/forum/202311/29/123405c185i4gg05lq01h4.png)
`image-20231128171241676.png` (151.43 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY2MDMxM3w4Yzk1NmZlMHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxM3w4Yzk1NmZlMHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

第一个方法有三处是返回的，将对应返回的寄存器赋值0就好

![](https://attach.52pojie.cn/forum/202311/29/123407qcyvapav2mwkaka6.png)
`image-20231128171606568.png` (156.6 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY2MDMxNHxhNTM2OGYwNHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxNHxhNTM2OGYwNHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

第二个方法照样是直接赋值0就好，将红框里的赋值0

![](https://attach.52pojie.cn/forum/202311/29/123409ydglqig3kqi5n8dl.png)
`image-20231128171731400.png` (171.23 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMxNXxlYWIyYjE3NXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxNXxlYWIyYjE3NXwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

第三个方法也是很明显有一个是返回true的地方，直接赋值0就好

![](https://attach.52pojie.cn/forum/202311/29/123412g37nhge7e3h7l7ed.png)
`image-20231128171949933.png` (168.89 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMxNnxlNjMyY2E0NHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxNnxlNjMyY2E0NHwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

第四个方法依旧是一样赋值就全部赋值完成了

![](https://attach.52pojie.cn/forum/202311/29/123414pdxs05y9zjo5d0u8.png)
`image-20231128172054936.png` (155.65 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY2MDMxN3xmM2ZlZTViN3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMxN3xmM2ZlZTViN3wxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:34 上传

（3）保存打包安装并打开，成功进入界面，不过就是广告有亿点点多

![](https://attach.52pojie.cn/forum/202311/29/123543q64jxen94k4eegj5.png)
`image-20231128172250196.png` (347.5 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY2MDMyMHxiOWJmM2E4YnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMyMHxiOWJmM2E4YnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:35 上传

## 去广告

（1）首先，点开软件会有一个开屏广告，这里会有一个活动跳转，那广告肯定是在两个活动的衔接代码之前，这样就可以定位广告代码在哪个类里面

（2）打开MT管理器的活动记录器来记录软件活动记录，并打开软件进行活动记录，发现一个很明显的广告标识

![](https://attach.52pojie.cn/forum/202311/29/123604h5m5wcmf7kqmm8q5.png)
`image-20231128223225229.png` (148.55 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY2MDMyMXw2NzkzYTg1MnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MDMyMXw2NzkzYTg1MnwxNzkzNTY4OTk5fDE2ODA4NnwxODYyOTk1&nothumb=yes)

2023-11-29 12:36 上传

（3）在jadx里面搜索这个类名并分析一下

![](https://attach.52pojie.cn/forum/202311/29/123637auutscuyt4cyrtfg.png)
`image-20231128223430158.png` (67.55 KB, 下载次数: 1)
