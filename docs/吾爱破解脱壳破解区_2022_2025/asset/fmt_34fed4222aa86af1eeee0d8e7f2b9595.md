
每次循环会解密 8 个字节的数据，如下图所示，已完成前面 8 字节的解密：

![small_024.jpg](https://attach.52pojie.cn/forum/202208/15/081233f9wrjqjujqnw90sc.jpg)

[`small_024.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkyNnxjNGM0YjYzY3wxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（223.7 KB，下载次数：232） · 2022-8-15 08:12 上传

全部解密完成后，就是一个 XML 文件，如下图所示，已经完成整个授权文件地解密了。

![small_025.jpg](https://attach.52pojie.cn/forum/202208/15/081239cvog4newhp44449g.jpg)

[`small_025.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkyN3wyOGJiMjY2MnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（231.01 KB，下载次数：232） · 2022-8-15 08:12 上传

可以看到，这是一个 XML 文件，第 1 步工作已经完成了，下一步是对授权内容进行处理了。

![small_026.jpg](https://attach.52pojie.cn/forum/202208/15/081245dt5hh5v9rvharg8a.jpg)

[`small_026.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkyOHxhODQ5ODc3MnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（231.83 KB，下载次数：235） · 2022-8-15 08:12 上传

先把解密后的 XML 文件内容从 `memoryBlock` 中取出来，生成一个字符串，如上图所示。

首先，验证解密后的内容是不是一个 XML 格式的内容，如下图所示，检查前 6 个字符是否

![small_027.jpg](https://attach.52pojie.cn/forum/202208/15/081250fe5z4866z3pix50f.jpg)

[`small_027.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkyOXxlN2FmYmEwM3wxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（227.49 KB，下载次数：247） · 2022-8-15 08:12 上传

就是简单的取出前 6 字节进行较，如下图所示：

![small_028.jpg](https://attach.52pojie.cn/forum/202208/15/081255l44o5405044h54zo.jpg)

[`small_028.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzMHw3NTU2YzQ4OHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（217.12 KB，下载次数：244） · 2022-8-15 08:12 上传

如果检查是 XML 格式的字符串，则初始化一个 `XMLDocument` 对象类，并通过其构造函数来解析 XML，生成 `XMLDocument` 对象：

![small_029.jpg](https://attach.52pojie.cn/forum/202208/15/081259i6ldzdq7vb67a6qq.jpg)

[`small_029.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzMXxmMjYzOGEwOHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（233.35 KB，下载次数：239） · 2022-8-15 08:12 上传

`XMLDocument` 对象生成成功后，就是循环取出其节点值，进行一下步的解析处理：

![small_030.jpg](https://attach.52pojie.cn/forum/202208/15/081306k02elpcem10b3ec0.jpg)

[`small_030.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzMnxmZDJjOTQ0MHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（216.22 KB，下载次数：239） · 2022-8-15 08:12 上传

主要是解析 3 个标签的值：

1. `address`
2. `signature`
3. `serial`

其它的标签是不解析的，如上面的授权文件中我加了一个 `email` 标签，就不会取出来。如下图所示，是检查 3 个标签的位置：

![small_031.jpg](https://attach.52pojie.cn/forum/202208/15/081310esjig2owx0olq2xa.jpg)

[`small_031.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzM3w0ZjkzNWRiYXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（210.87 KB，下载次数：231） · 2022-8-15 08:13 上传

取出来的标签的值，会保存到 `KeyValidator` 对象的属性中，如下图所示，保存的是 `serial` 属性：

![small_032.jpg](https://attach.52pojie.cn/forum/202208/15/081315y985btm8t9zvx888.jpg)

[`small_032.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzNHw1ZmIwOWNlOHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（283.27 KB，下载次数：235） · 2022-8-15 08:13 上传

下面保存是 `address`：

![small_033.jpg](https://attach.52pojie.cn/forum/202208/15/081319g5djjoirl39d0i2h.jpg)

[`small_033.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzNXxhZmY2NDU5M3wxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（279.56 KB，下载次数：239） · 2022-8-15 08:13 上传

以及 `signature`：

![small_034.jpg](https://attach.52pojie.cn/forum/202208/15/081325m649q31hgkf4fexu.jpg)

[`small_034.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzNnwwZTE0ZjdmNnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（277.18 KB，下载次数：234） · 2022-8-15 08:13 上传

接下就是执行 `KeyValidator` 的构造函数，对刚才取出来的三个 XML 标签值进行下一步的验证，检查授权是否合法：

![small_035.jpg](https://attach.52pojie.cn/forum/202208/15/081329a3m8xivg389313xy.jpg)

[`small_035.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzN3xiNzAxYTQxMnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（211.79 KB，下载次数：233） · 2022-8-15 08:13 上传

因为后面还有解密过程，并且是用的 `email` 作为密码解密，程序会从授权信息数据库查询以前的 `email`，这样就可以不用用户来输入解密密码了，如下所示，查询授权文件数据库（是一个全文加密的 SQLite 数据）：

![small_036.jpg](https://attach.52pojie.cn/forum/202208/15/081334c5atmub5qtm8m8bx.jpg)

[`small_036.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzOHxkNGQwNzRiZXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（231.28 KB，下载次数：237） · 2022-8-15 08:13 上传

该文件保存在目录 `c:\Users\用户名\AppData\Roaming\Xojo\Xojo` 中，文件名是 `License Keys`。如下图所示，是查询密码的 SQL：

![small_037.jpg](https://attach.52pojie.cn/forum/202208/15/081338gftwk8qsdltz4c3w.jpg)

[`small_037.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTkzOXwxNDYwMzQ3NXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（194.05 KB，下载次数：238） · 2022-8-15 08:13 上传

如果没有查到密码，也就是没有以前的授权文件，就会显示一个对话框，由用户输入邮箱名作为后的解密密码：

![small_038.jpg](https://attach.52pojie.cn/forum/202208/15/081343l9g94kxnbjkark3x.jpg)

[`small_038.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTk0MHwxNzQ3YzhjZnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（209.7 KB，下载次数：230） · 2022-8-15 08:13 上传

显示密码对话框：

![small_039.jpg](https://attach.52pojie.cn/forum/202208/15/081347d57nw8ey1amjk3y3.jpg)

[`small_039.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTk0MXxiZjhjZDNmZHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（210.52 KB，下载次数：240） · 2022-8-15 08:13 上传

该对话框如下图所示（2 个图，1 个没有输入，一个已输入 `email`）：

![small_040.jpg](https://attach.52pojie.cn/forum/202208/15/081351ilrlqr9t97mmm8cl.jpg)

[`small_040.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTk0MnxmZDk4ZDQyMHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（186.5 KB，下载次数：235） · 2022-8-15 08:13 上传

如果输入的邮箱地不正解，后面的解密数据会不对，在解密后的校验中，会弹出以下错误：

![small_040_1.jpg](https://attach.52pojie.cn/forum/202208/15/081355b7o2m7boz72iv1l1.jpg)

[`small_040_1.jpg`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NTk0M3w5ZGFjZDIwM3wxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（100.19 KB，下载次数：246） · 2022-8-15 08:13 上传
