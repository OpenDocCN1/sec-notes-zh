
# 解密过程分析

如果邮箱没有错，可以继续下一步的解密过程了。

刚才弹出的密码对话框，又会再次生成和实例化 `Registration.StudioKeyValidator` 对象，并再次在 `constructor` 中重复一遍前的解码、解密、XML解析过程，很是无聊，这就是面向对象编程的问题，每次 `new Object` 后，要重复 `constructor` 一次。

![](https://attach.52pojie.cn/forum/202208/15/081402pbnp37z6cje3cps2.jpg)
small_041.jpg (205.51 KB, 下载次数: 231)

把前过程重复一次后，我们来到了这里：

![](https://attach.52pojie.cn/forum/202208/15/081406omqhkk7xqg6o6mmk.jpg)
small_042.jpg (208.73 KB, 下载次数: 224)

这里将输入的`email`地址转换成全小写的字符串。

接下来就是用这个小写的 `email` 为密码，执行一个解密函数，生成另外一个对象：`StudioKeyData` 对象，保存具体的授权数据。

![](https://attach.52pojie.cn/forum/202208/15/081410b8wo3e5wdwy3vhvv.jpg)
small_043.jpg (212.76 KB, 下载次数: 255)

我们进入这个函数，跟踪处理过程（按 `F7` 进入函数）

首先处理 `StudioKeyValidator.signature` 属性：

![](https://attach.52pojie.cn/forum/202208/15/081416kqu8gwvpz45tt4vg.jpg)
small_044.jpg (232.63 KB, 下载次数: 243)

取出 `signature` 属性的字符串，这是一个 1024 个字符的字符串，包含有两个部分，每部分都是 512 个字符，后面会讲到其作用。

如下图所示，先截取其前面 512 个字符：

![](https://attach.52pojie.cn/forum/202208/15/081427i31547dv5czybi14.jpg)
small_045.jpg (230.03 KB, 下载次数: 238)

再取剩下的一半字符，如下图位置代码来完成：

![](https://attach.52pojie.cn/forum/202208/15/081431i0b66bdbzb0cn6vo.jpg)
small_046.jpg (227.79 KB, 下载次数: 237)

取出后半部分后，首先对其进行解码，变成16进制的字符串，长度也变成了 256 字节了：

![](https://attach.52pojie.cn/forum/202208/15/081436k3yy0y65dtzyxp62.jpg)
small_047.jpg (223.36 KB, 下载次数: 241)

这个 256 字节的字符串，实际上是一个加密后的 2048 bits 的 `RSA` 签名。

接下来是对这个加密的签名进行解密。

这次解密的密码是动态生成的，不是常量字符串了，并且跟 `signatrure` 的前半部分的 512字符相关。

这次密码有 64 字节，由 `memoryBlock` 保存，所以先初始化一个 `memoryBlock` 对象：

![](https://attach.52pojie.cn/forum/202208/15/081440z7ctoe1zoep1twok.jpg)
small_048.jpg (227.99 KB, 下载次数: 235)

首先计算前 16 字节的密码，这16字节是我们输入的 `email` 地址的 `MD5` 码，如下图所示，计算 `MD5` 码：

![](https://attach.52pojie.cn/forum/202208/15/081444v4zt4a0ivbqiitgv.jpg)
small_049.jpg (220.59 KB, 下载次数: 240)

将 16 进制的 `MD5` 码存入 `memoryBlock` 的缓冲区，如下图所示：

![](https://attach.52pojie.cn/forum/202208/15/081448yy6zqont6xfc0v5q.jpg)
small_050.jpg (222.86 KB, 下载次数: 246)

接下来处理序列号，生成与序列号相关的一段密码，先将序列号中的“-”去掉，如下图所示：

![](https://attach.52pojie.cn/forum/202208/15/081453pgawg53chta3t39c.jpg)
small_051.jpg (212.84 KB, 下载次数: 246)

同样进行16进制解码，变成16进制的字符串：

![](https://attach.52pojie.cn/forum/202208/15/081457w02ap2303r8crpnc.jpg)
small_052.jpg (220.56 KB, 下载次数: 234)

`serial` 变成 24 字节长的 16 进制字符后，填充到密码缓冲区：

![](https://attach.52pojie.cn/forum/202208/15/081501tuyuzxgoxpxppqp2.jpg)
small_053.jpg (221.34 KB, 下载次数: 229)

第3步就是还有24字节的密码需要生成，而这24字节的密码的生成与 `signature` 前512个字符相关了，24个字节密码的前16个是一个字节一个节生成的，最后8字节是一次生成的（是一个8字节的双精度浮点数）。

这个步骤的代码相当长且臭，就不一一説了，就是一些查字符串位置，统计某些字符的个数等，并把得到的这些数据填充到密码表中。

![](https://attach.52pojie.cn/forum/202208/15/081506c6bxyk7bkrb2dkdb.jpg)
small_054.jpg (232.58 KB, 下载次数: 241)

查找和统计的字符串有“CAF”, "BAD", "11", "8", "A", "3", "7"等等，还有取最后一个还是2个字符的操作。如下图，是统计字符“A”的个数：

![](https://attach.52pojie.cn/forum/202208/15/081509p0arwawervnvevok.jpg)
small_055.jpg (222.36 KB, 下载次数: 242)

然后将个数据写入密码表：

![](https://attach.52pojie.cn/forum/202208/15/081514gz80gg8qb9yo0yom.jpg)
small_056.jpg (220.47 KB, 下载次数: 228)

就这样生成16字节的密码，存入密码表：

![](https://attach.52pojie.cn/forum/202208/15/081519qlcjcdhvr3xcxojq.jpg)
small_057.jpg (240.17 KB, 下载次数: 244)

最后8字节的密码是一个幂计算的结果（8字节双精度浮点数），如下图所示：
