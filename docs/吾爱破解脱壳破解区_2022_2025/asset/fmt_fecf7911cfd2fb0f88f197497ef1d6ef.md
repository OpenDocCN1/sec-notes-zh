
# [.NET逆向] VisualNDepend逆向工程

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1959406
- 抓取时间: 2026-11-01T22:24:35

## 背景介绍

搜.Net Assembly Diff工具时候，在StackOverflow一个回答中了解到NDepend工具，可以试用14天，试了下diff效果还不错，别的功能也很丰富，下载到的版本是`2024.1.1.9735`。

```
.NET Assembly Diff / Compare Tool - What's available? [closed]
https://stackoverflow.com/questions/1280252/net-assembly-diff-compare-tool-whats-available
```

### 激活流程

将程序断网后点击`Start Evaluation`会进入NDepend manual server access流程，自动生成了`Date Request text`，到网站可以获取到`Data Response text`，输入后即可激活。

![](https://attach.52pojie.cn/forum/202408/29/164927nds87ojsd8so8sjr.png)

1.png (26.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDUzOXw1ODU1MzI3NXwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDUzOXw1ODU1MzI3NXwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

1

2024-8-29 16:49 上传

程序联网的话，应该是后台直接发起请求验证结果完成激活，对程序断网可以方便分析，并且程序会随时联网发送消息，比如patch造成或修改授权造成的运行异常报告，还是给断了好。

### 展开分析

14天后，打开`dnspy`开始看下，大部分函数名、变量名、字符串都被混淆了，程序流程还是好的。注意有弹窗，试着断在`MessageBox.Show`，运行断下来了，看下调用堆栈回溯，可以以此展开调试分析。

![](https://attach.52pojie.cn/forum/202408/29/165250lx578q5zz4cz3xgx.png)

2.png (172.8 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU0MXw4OWRkZjg1N3wxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0MXw4OWRkZjg1N3wxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

2

2024-8-29 16:52 上传

### 授权文件

经过一番杂乱但并不复杂的调试分析，知道程序解密验证服务器返回的数据后，将进行加密保存为`C:\ProgramData\NDepend\ActivationEval`文件，程序每次启动都会解析此文件进行验证。

断在`NDpend.Core.dll`的`oJX.xJg`函数中，调试可知前半部分是读`ActivationEval`文件内容，在`kSx.cSZ`函数中解密文件。

![](https://attach.52pojie.cn/forum/202408/29/165326eeszf7oalis5ysd7.png)

3.png (130.55 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyMDU0Mnw3ZjVmNDYzNHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0Mnw3ZjVmNDYzNHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

3

2024-8-29 16:53 上传

![](https://attach.52pojie.cn/forum/202408/29/165403mfzwfpi9bwhltohl.png)

4.png (141.56 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU0M3wzNzJmY2Q4ZHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0M3wzNzJmY2Q4ZHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

4

2024-8-29 16:54 上传

### 内容解密

跟进`kSx.cSZ`函数，看到函数调用链中出现`RijndaelManaged`、`Rfc2898DeriveBytes`、`CpherMode.CBC`、`PaddingMode.PKCS7`等字样，可知主体上是PKCS7填充AES-CBC解密的C#调用。

先进入`CSq`函数，`m4f`是key和iv的长度，程序会用256和128都试一下，前者会解密失败，再用128进行解密。

![](https://attach.52pojie.cn/forum/202408/29/165428e0f63oh23uc3t23q.png)

5.png (37.5 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU0NHwzODg0NTg1OHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0NHwzODg0NTg1OHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

5

2024-8-29 16:54 上传

进入`jSp`，先对文件内容进行base64解密，随后分为`[:0x10]`、`[0x10:0x20]`和`[0x20:]`三部分。第一部分用来作为`password`，和`salt`字符串传入`Rfc2898DeriveBytes`初始化，随后生成AES解密的key，第二部分是AES解密的iv，第三部分是密文。

![](https://attach.52pojie.cn/forum/202408/29/165443elziiotg0zszgqgf.png)

6.png (170.16 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcyMDU0NXxjMTFjOTQ1YnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0NXxjMTFjOTQ1YnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

6

2024-8-29 16:54 上传

AES解密完后会进入`YSA`函数，可以看到又进行了一遍base解密，随后解压操作，即可得到授权数据的明文。

![](https://attach.52pojie.cn/forum/202408/29/165518df5ezswhse1sit66.png)

7.png (61.59 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyMDU0NnxmZDNlYWIyZnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0NnxmZDNlYWIyZnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

7

2024-8-29 16:55 上传

### 签名验证

可以看到末尾段是有一段签名信息的，根据判断会使用`rsa-sha1`验签。

![](https://attach.52pojie.cn/forum/202408/29/165543x5jsh0mvcfvmvfgm.png)

8.png (94.5 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcyMDU0N3wwZWEwNWZhMnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0N3wwZWEwNWZhMnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

8

2024-8-29 16:55 上传

根据调用堆栈，回溯到`HIc`函数，再进入`X2t.k2x`函数，根据局部变量内容，我们可以看到从`HadrwareID`提取出了要验签的内容，加载了`RSAKeyValue`并进行了哈希验证，最后进入`l4h.M4D`函数开始验签。

![](https://attach.52pojie.cn/forum/202408/29/165612grcgzb6m32d8b4km.png)

9.png (167.6 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU0OHxiYmU3NzM4NHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0OHxiYmU3NzM4NHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

9

2024-8-29 16:56 上传

调试进入`c4S`函数中，根据局部变量的信息，可以判断`Y4y`是`rsaCryptoPublic.VerifyData(hashToSignBytes,signature)`函数。

![](https://attach.52pojie.cn/forum/202408/29/165651bm80u1dq11z1whe1.png)

10.png (88.98 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyMDU0OXxkZjhjYTQ2M3wxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU0OXxkZjhjYTQ2M3wxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

10

2024-8-29 16:56 上传

`RsaCryptoPublic`实例的初始化是`c4X`函数。

![](https://attach.52pojie.cn/forum/202408/29/165718b8a8wla58vajj3j7.png)

11.png (109.48 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyMDU1MHxlMjk2YzI1MHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU1MHxlMjk2YzI1MHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

11

2024-8-29 16:57 上传

调试看到从`RSAKeyValue`中初始化了`Modulus`和`Exponent`，是模数和公钥指数。

![](https://attach.52pojie.cn/forum/202408/29/165726zre36956z9kai566.png)

12.png (23.41 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcyMDU1MXxmMzE1NzE4ZHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU1MXxmMzE1NzE4ZHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

12

2024-8-29 16:57 上传

模数是两个大素数的乘积，到`factordb.com`上分解下，未果，那只能patch过掉了。

![](https://attach.52pojie.cn/forum/202408/29/165742tzjq868dq6z8dbv6.png)

13.png (38.24 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU1Mnw4NDJkZjk3YXwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU1Mnw4NDJkZjk3YXwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

13

2024-8-29 16:57 上传

### 信息校验

查看调用堆栈回溯到`G1Z4`函数，进入`EIx`函数，便是信息校验部分。分为两部分，一部分是本机`HardwareID`信息是否和解密的授权数据匹配，另一部分是检查授权相关日期相关信息。

![](https://attach.52pojie.cn/forum/202408/29/165748mg8u4kf7uhuee77s.png)

14.png (117.47 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU1M3xlYmM1MjMzZnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU1M3xlYmM1MjMzZnwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

14

2024-8-29 16:57 上传

`HardwareID`部分信息我们可以解密软件生成的`data request text`数据获取，授权信息日期中日期相关部分内容如下，可以猜测信息有注册时间、到期时间、多少天后弹出激活、多少天后弹出请求更多试用、过期后多少内还能再请求更多试用。

```
23 Aug 2024

07 Sep 2024

False

6

4

240

2024.1.1.9735
```

可以看到程序对几个日期有一些比较，我们在构造授权文件时候，几个日期的大小可以按照服务器真正返回的内容来。

![](https://attach.52pojie.cn/forum/202408/29/165759kz3wwqgvaynnppgw.png)

15.png (95.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyMDU1NHxhNmQwOGMzOHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyMDU1NHxhNmQwOGMzOHwxNzkzNTcxODczfDE2ODA4NnwxOTU5NDA2&nothumb=yes)

15

2024-8-29 16:57 上传

### 生成授权

思路是先解密软件手动激活时候弹窗里面的`data request text`，得到`HardwareID`部分信息，构造出授权信息明文，修改过期时间，再加密生成`ActivationEval`文件放在`C:\ProgramData\NDepend\`目录下即可。

整个过程并不复杂，简单提一下使用`python`实现的点。

1. `C#`实现AES加解密，是传入`password`和`salt`生成`Rfc2898DeriveBytes`实例再生成的key，加解密时的`salt`是不同的，可以调试获取。
