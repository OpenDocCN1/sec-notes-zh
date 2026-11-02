
# [原创] CrackMe160学习记录1 Acid burn

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1609332>
- 抓取时间：2026-11-01T23:31:51

## 1 Acid burn

### 1. 前期信息探查

#### EXEinfo

![]()  
``95$`DSN6LN(WRPO2F[RHRKR.png`` (101.17 KB, 下载次数: 3)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc3NHw1ZWQzMzJjNnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:41 上传

#### 运行一下

首先会弹出一个弹框：

![]()  
``K95(~6$K0M@$A6B{QG}V4U.png`` (12.86 KB, 下载次数: 3)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc3NXwwMGM4NjE0M3wxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:47 上传

确定后就进入了主界面：

![]()  
``H8D7H9L[QZ2$}M}A{}H9RJO.png`` (184.02 KB, 下载次数: 1)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc3Nnw2MWI5ZDU4YnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:48 上传

先看左边的用户名和密码模块，进去看一下：

![]()  
``1OG6)9((O0M8A$X$K@$K2VO.png`` (101.31 KB, 下载次数: 2)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc3N3w4NDVmMjk2ZnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:49 上传

尝试随便输一些东西：

![]()  
``M{~V3AN%HVDG_N`LKQ7P@`D.png`` (10.33 KB, 下载次数: 1)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc3OXw1MGFiYWExNnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:50 上传

然后是右边的序列号验证：

![]()  
``}_RN8EQ4(H9GDR1`VWYP[A8.png`` (91.17 KB, 下载次数: 1)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4MHxiNTI0OGY1MnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:50 上传

![]()  
``9DVY6)TQ5Z4J(36(7WAO3FI.png`` (8.76 KB, 下载次数: 2)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4MXxkNzRlNGE0MnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:51 上传

### 2. 动态调试

先分析右边的序列号(因为这个简单一点)

#### 序列号验证功能

当序列号输入错误时，会弹出一个对话框：“Try Again!!”

所以第一个想到的就是搜索参考字符串：

![]()  
\`\`ZZ5KD1JZCIKU7C]V0BRFX.png (12.13 KB, 下载次数: 2)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4MnxkZjFlY2Q4YnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:52 上传

这里发现有两处“Try Again!!”，这让我有点异或，这里选择先跟过去看看有哪些地址引用了它：

![]()  
``J~3YL8CKO3(O)R1D~U[T{HM.png`` (15.02 KB, 下载次数: 0)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4M3wxMzJhZTNkY3wxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:53 上传

![]()  
``J~3YL8CKO3(O)R1D~U[T{HM.png`` (15.02 KB, 下载次数: 3)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4NHw5MTkxZDI2OXwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:54 上传

我在两处都下了断点然后运行程序想看一下程序会停在那个地方，但可惜的是它哪都没停，之后我在下面找到了

![]()  
``771QUVTSLEETLPQ$)PX{#.png`` (25.81 KB, 下载次数: 3)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4NXxiMWZlYmQ1YnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:55 上传

我个人猜测这应该是识别错了，我转用`x32dbg`，就可以正常的的操作了。

![]()  
``U`I~F5@FO6DFJI42$QLOR27.png`` (17.77 KB, 下载次数: 2)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4N3wwNGIyNWU5ZnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:56 上传

可以看到验证序列号的字符串是第一个有两个感叹号的字符串，双击跟过去：

这部分的代码大概意思如下：`42F4D0`处的代码利用`call`跳转到序列号的判断函数，参数有两个分别在`42F4CA`、`42F4CD`处压栈保存，判断结果在标志寄存器里面，然后在`42F4D5`处`jne`（当`ZF`标志为0时跳转）对标志寄存器进行判断，也就是说这里判断返回的结果是否为0，为0，`ZF`置一，跳转至`42F4F1`处输出“Try Again!!”，结果为1，则输出“God Job dude !!”。

![]()  
``8EU@CF[XYU598JLVCX4TE~B.png`` (84.59 KB, 下载次数: 1)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4OHw1YTQ3MGRjZXwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:57 上传

![]()  
``~0%`8PI3F)5R~S@LC0UV@ZF.png`` (39.07 KB, 下载次数: 2)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc4OXxjNDY4M2VkOHwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-25 23:58 上传

在`42F4D0`处下断点然后运行，选择`Serial`窗口，然后输入就以默认输入，之后直接`F9`，可以发现：

程序停在`42F4D0`处时，就可以看到调用函数验证时的两个实参的值了，第一个为我们自己输入的内容，另外一个就是真正的序列号字符串“Hello Dude!”

#### 用户名密码功能

根据之前收集到的信息得知用户名密码输错会报一个字符串“Sorry，The serial is incorect！”

还是一样直接搜索字符串，通过错误跳转处看引用其的条件跳转语句：

![]()  
``_]J[~OL7XS_[4U03MU6N8.png`` (62.42 KB, 下载次数: 1)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc5MHw4MGM5MTljNnwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-26 00:03 上传

在`42FAFE`处下断点，然后进入程序的`Serial/Name`功能，随便输入一些东西

![]()  
``A]3A2)YHALEW29F)MJM6(}O.png`` (100.65 KB, 下载次数: 1)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc5MXxkYjQ0MmQ0YXwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-26 00:04 上传

`Check it`后再看调试界面就能看到`Serial`了。

![]()  
``{6FO_H_1OA}A]$]N1KR@YWR.png`` (33 KB, 下载次数: 2)  
[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNzc5Mnw0NGM4MTM4MXwxNzkzNTc1OTA4fDE2ODA4NnwxNjA5MzMy&nothumb=yes)  
2022-3-26 00:04 上传

但这里不像之前的操作那么容易，一般既要输入用户名又要输入密码的话，密码可能是通过用户名计算的，这里尝试一下别的用户名

![]()  
``0KB5O4M~(_7B]R32KWOEY}W.png`` (61.17 KB, 下载次数: 2)
