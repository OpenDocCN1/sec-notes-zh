
![](https://attach.52pojie.cn/forum/202503/22/184832ntb86mgxj1bj6zgt.png)

Pasted image 20250313211228.png (91.19 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDk4OXw4ZWY1ZGE0ZXwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk4OXw4ZWY1ZGE0ZXwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:48 上传

这里我在函数进入后直接对`bHasLicense`写入1。然后无条件跳转到下面的`return 0`语句。然后将改动直接写入到原文件。

## 修改后代码如下
![](https://attach.52pojie.cn/forum/202503/22/185001d1vzove05pod2mon.png)

Pasted image 20250313224231.png (68.19 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc2NDk5MHxjYzlkYzRlMHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk5MHxjYzlkYzRlMHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:50 上传

## 演示版程序不能保存
演示版程序不能保存，如下。

![](https://attach.52pojie.cn/forum/202503/22/185020koz1jjhga1ofgao8.png)

Pasted image 20250313213209.png (158.2 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDk5MXw4NzE3YTg5ZHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk5MXw4NzE3YTg5ZHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:50 上传

## 修改后程序可保存
修改后程序如下，可以正常保存。

![](https://attach.52pojie.cn/forum/202503/22/185022bf825z5ttm1s1i05.png)

Pasted image 20250313223956.png (142.64 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDk5MnxkMGVmYzI4OHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk5MnxkMGVmYzI4OHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:50 上传

我本以为这样就可以开开心心地爆破，但是在程序保存和读取文件时报错了，起初我还以为是SAI2的bug，还去浏览器上查了半天🥲。兜兜转转调试了很多次后发现并不是bug引起的错误，而是程序还有暗桩。

## 报错如下
![](https://attach.52pojie.cn/forum/202503/22/185112jdd5ab5jb5avvtn2.png)

Pasted image 20250322105700.png (119.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDk5M3xlNDFkYTY5MHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk5M3xlNDFkYTY5MHwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:51 上传

## CreateFileW 条件断点调试
经过我对`CreateFileW()`API下条件断点调试得出程序只读取了一次证书文件。

过程如下

给`CreateFileW()`API设置条件断点，条件为`msg("Filename: %s\n", get_strlit_contents(RCX, 180, "UTF-16LE")), 0`

这样IDA会在调试的时候自动输出`CreateFileW()`第一个参数，也就是读取的文件目录到控制台，180是字符串长度。我们知道字符串长度肯定会不一样，但是IDA这个函数对UTF-16编码的识别不是很智能只能先设置个180显示。后面的0是布尔参数，1的时候断点会中断，0则不会，两种情况都会执行`msg()`函数。

![](https://attach.52pojie.cn/forum/202503/22/185311ngozggpzzg54o845.png)

屏幕截图 2025-03-22 185303.png (240.81 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc2NDk5NXwxNzEwMDY3ZnwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk5NXwxNzEwMDY3ZnwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:53 上传

我们就可以很清楚的看到程序只读取了一次证书文件。

## 可能原因分析
既然证书只被读取了一次，那么问题出在哪里呢？目前有以下几种可能的解释：

- 程序内部有多次验证，比如证书的内容或根据证书计算出的注册码被保存到了内存中，每次保存或者读取的时候都会调用验证函数验证。
- 证书被保存在了设置文件中，每次运行的时候在这里读取。

至于修改了验证函数还是会验证失败：

- 有好几个不同实现方式的验证函数
- 验证关键函数在我之前改的函数的子调用中，其他函数仍可调用该未修改过的函数进行验证。

对于证书被保存在其他地方的情况我试过删除证书源文件，程序提示证书文件未安装。说明证书可能安装在其他文件里的，但是这不能下定论。接下来我把证书内容==设置为从a到z的小写字母==，运行程序然后去设置文件里找，发现并==没有相同内容==，所以这种可能性排除。

那么就只剩第一种情况了。

对于第二个问题，我们只能通过调试和分析寻找答案。

## 验证函数分析
这里我选择跟进验证函数，分析如下。

紫色箭头为正常流程，如果不存在证书文件则会在第一个跳转处直接走红色路径。

所以接下来的调试最好准备一个`.slc`文件放到程序根目录下，文件内容随便。

### 汇编流程分析
![](https://attach.52pojie.cn/forum/202503/22/185525reeqzxoe7ee7ijh8.png)

af5bt-vz9jf.png (2.85 MB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc2NDk5NnxjODU3NTRhOXwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDk5NnxjODU3NTRhOXwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 18:55 上传

反编译：
