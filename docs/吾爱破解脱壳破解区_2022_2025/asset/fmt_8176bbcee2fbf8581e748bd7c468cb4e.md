
# [CTF] 吾爱 2024 春节解题领红包 Web 题 WriteUp

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1893063
- 抓取时间: 2026-11-01T22:37:46

Web(×) Misc(√)

辛苦出题大大，整体体验感不错。小问题就是难度设置不是很舒服，下面也会提到`x`。

## 正文

### `flag1{52pj2024}`

视频第 2 秒，白色亮圈从中心扩大的时候，很容易发现后面有字（始终保持黑色的区域）。我直接看出来了，因为是有意义的内容。也可进行叠加如下：

- 逐帧导出视频

```bash
ffmpeg -i input_video.mp4 output/frame%d.jpg
```

- Photoshop 中进行线性叠加

![flag1.png](https://attach.52pojie.cn/forum/202402/24/211234ht7grgse9rsrzffj.png)

`flag1.png` (1.13 MB, 下载次数: 1) 2024-2-24 21:12 上传

### `flag2{xHOpRP}`

视频第 5 秒开始的二维码，用 Photoshop 拼接后反相，识别后得到网址 [https://2024challenge.52pojie.cn/index.html](https://2024challenge.52pojie.cn/index.html)（下称“**主网址**”）。登录后在响应标头中即可看到 `flag2{xHOpRP}`。

![flag2.png](https://attach.52pojie.cn/forum/202402/24/211236zuhc6hrbnnrcduc9.png)

`flag2.png` (133.2 KB, 下载次数: 1) 2024-2-24 21:12 上传

这么前面的 flag，然后我就一直在视频中找ww

同时还看到了 `flag4_flag10.png` 这张图片。

### `flag3{GRsgk2}`

视频第 0 秒，和 `flag1` 一样直接看了。或许有一些我不知道的处理方法，希望各位大神指点。

### `flag4{YvJZNS}`

`flag2` 中说了，图片中明文。

### `flag5{P3prqF}`

主网址源码中注释。发现颜色被改成白色，并设置不可选中，导致一开始并没有看到。修改后即得到 `flag5`。

![flag5.png](https://attach.52pojie.cn/forum/202402/24/211239aa5e0a5xexzjsaza.png)

`flag5.png` (219.81 KB, 下载次数: 1) 2024-2-24 21:12 上传

同时看到 `flag9` 也在这里。

现在是在 ……….. 中找到字母一个个拼出来，在 `flag9` 中会看到更直观的显示。

### `flag6{20240217}`

主网页中进入 `flag6`，查看源码发现它自己会去算的，`1e8` 次循环，循环体中也没看到卡时间的部分，就让它自己算即可。甚至贴心的在 `console` 中有输出，65s 就出了结果。

![flag6.png](https://attach.52pojie.cn/forum/202402/24/211241kqqg9i79zzgw9h59.png)

`flag6.png` (82.04 KB, 下载次数: 0) 2024-2-24 21:12 上传

### `flag7{Djl9NQ}`

视频第 21 秒，前往对应 github 仓库，查看 commit 记录，得到 `flag7`。

![flag7.png](https://attach.52pojie.cn/forum/202402/24/211243qshs3wwkxpkhswk2.png)

`flag7.png` (120.91 KB, 下载次数: 1) 2024-2-24 21:12 上传

甚至 commit message 都这么直接，是故意的还是不小心？

### `flag8{OaOjIK}`

主网页中进入 `flag8` & `flagB`，玩到 10000 分领了 `flag8`。正确姿势见 `flagB`。

### `flag9{KHTALK}`

`flag5` 中说了，看这 `/ \ _` 总感觉很眼熟，但一时想不出来。Brainfuck、二进制对应等等都想过，怎么映射都对不上。后来终于想起来了，请看：

- 知乎

  ![flag9-1.png](https://attach.52pojie.cn/forum/202402/24/211245qv4ewr3pvbrg4dg3.png)

  `flag9-1.png` (252.89 KB, 下载次数: 1) 2024-2-24 21:12 上传

- Bilibili

  ![flag9-2.png](https://attach.52pojie.cn/forum/202402/24/211248ih4tr345yqqezo4l.png)

  `flag9-2.png` (137.54 KB, 下载次数: 0) 2024-2-24 21:12 上传

调整网页宽度，`flag9` 就出来了。同时 `flag5` 更清楚了。

![flag9-3.png](https://attach.52pojie.cn/forum/202402/24/211250rlshr2qptukqvpxj.png)

`flag9-3.png` (61.09 KB, 下载次数: 1) 2024-2-24 21:12 上传

### `flag10{6BxMkW}`

`flag4_flag10.png` 那张图片，考虑隐写，先把老套路的方法都上一遍再说，结果 Stegsolve 一下就出来了。

![flag10.png](https://attach.52pojie.cn/forum/202402/24/211255euhq27ihur2hirxe.png)

`flag10.png` (88.75 KB, 下载次数: 1) 2024-2-24 21:12 上传

### `flag11{HPQfVF}`

主网页中进入 `flag11`，发现是一张图片被分成很多块后显示在特定位置，源码中提示在 0 ~ 100 范围内调整 `var1` 和 `var2` 两个值。

![flag11-1.png](https://attach.52pojie.cn/forum/202402/24/211259r3vyv21v2c238cd8.png)

`flag11-1.png` (559.9 KB, 下载次数: 0) 2024-2-24 21:12 上传

下载 `index.html` 和 `flag11.png` 到本地，编写脚本：

```python
from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.get('file:///xxx/index.html')

for var1 in range(0, 101):
    for var2 in range(0, 101):
        script = f"document.documentElement.style.setProperty('--var1', '{var1}');"
        script += f"document.documentElement.style.setProperty('--var2', '{var2}');"
        driver.execute_script(script)
        print(var1, var2)
        time.sleep(0.01)

driver.quit()
```

快速遍历 0~100 的范围，每 0.01 秒改变一次图像。不断缩小遍历范围，最后得到 `flag11`。

![flag11-2.png](https://attach.52pojie.cn/forum/202402/24/211304y66xji665xz66f6w.png)

`flag11-2.png` (921 KB, 下载次数: 0) 2024-2-24 21:13 上传

一开始还以为是小方块组成对应文字，后来才意识到是重排顺序组成一张图片，被自己蠢的笑了。。。

### `flag12{HOXI}`

主网页中进入 `flag12`，发现是将你的输入跑一段汇编，结果非零则输出正确 `flag`。无奈汇编看不懂，只能跑暴力。

启用本地替代，修改程序，开始暴力：

![flag12-1.png](https://attach.52pojie.cn/forum/202402/24/211314vsdldddgg7f6dzz6.png)

`flag12-1.png` (129.17 KB, 下载次数: 0) 2024-2-24 21:13 上传

非常漫长，如果不自信到底有没有在跑可以时不时设个断点看一下。记得等了个 5~10 分钟才出了结果。

![flag12-2.png](https://attach.52pojie.cn/forum/202402/24/211321y9gjdj6eg6smzcdd.png)

`flag12-2.png` (38.33 KB, 下载次数: 0) 2024-2-24 21:13 上传

此时 `secret` 几乎是上界……之后重新倒着暴力了一遍，不用一分钟就出来了。

### `flagA`

没出来，主网页登录后 `login` 响应标头中有 `flagA`，看上去像 base64 加密的，但解密出来是乱码。在网页的各个地方找解密接口找不着。

### `flagB`

看到 `flagB` 价格的数字这么“精准”，能想到可能和溢出相关，`v50` 的提示的确就是溢出。

对于不大了解的，暂时可以简单理解为，当数字达到正的最大值时，再加 1，则会变成负的最小值，反之同理。相当于一个循环的结构。

购买个数随便输了个很大的数字，提示“购买后金币大于原金币”，被拦截了。因此需要精心构造一个数字。

首先要确定存储金币的数据类型的范围。用便宜的 100 一个的东西开始试（精确），逐步加大购买个数（后面加 0），找到提示“钱不够”和“购买后金币大于原金币”的分界线，就能确定最高位，然后继续精确下一位，等等。前几位试了知道是 `9.22e16` 左右，乘上单价也就是说（负数的）数据范围在 `-9.22e18` 左右，熟悉的就知道应该是 64 位有符号整数了（ $-2^{63} \sim 2^{63}$ ）。

然后就可以构造购买个数了。记 $N=2^{63}$，则购买价格为 $Q$ 的东西时，数量为 $\lceil\frac{2N}{Q}\rceil$ 。

![flagB.png](https://attach.52pojie.cn/forum/202402/24/211324gy5btkzr7tjikfk9.png)

`flagB.png` (209.99 KB, 下载次数: 0)
