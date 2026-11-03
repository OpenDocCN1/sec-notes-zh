
# 修改收费有锁关卡URL的思路

重点来了，`url`这个地方怎么改？原帖作者已经为我们解释了，免费无锁的关卡的`url`是有内容的，收费有锁的关卡的`url`是没有内容的，而且这个`url`的形成有一个规律，每个关卡对应的`url`只有中间部分不同，中间部分恰好是各个关卡对应的`taskId`，那么`url`的格式就是`url = 统一开头 + taskId + 统一结尾`。

也就是`url = https://darwin-freetalk-v2.liulishuo.com/tasks/ + "taskId" + ?source=freetalk_list`，同时注意到每个关卡的`taskId`都是有的，那我们就可以写代码创建`url`，补齐收费有锁关卡的`url`。

## 修改思路

我们修改思路也清晰了，先看服务器返回的`url`是不是空的，如果是空的，我们就创建一条`url`再作为方法的返回结果，如果不是空的，直接将服务器返回的`url`作为方法的返回结果。

先找到`getUrl`这一方法。

![qq_pic_merged_1712767675486.jpg](https://attach.52pojie.cn/forum/202404/11/004757h2odu5ndonvh1o12.jpg)

`qq_pic_merged_1712767675486.jpg`（32.29 KB，下载次数：1；2024-4-11 00:47 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY2M3wxZDdmOTFkMnwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

## 判断URL是否为空

怎么写判断`url`是否为空的代码？这里是`smali`语法，我也不太清楚该怎么写，但是我知道`smali`语句有其对应的`java`语句，我们可以先写`java`再转`smali`，复制粘贴进来就可以。

这里用到NP管理器的`java`转`smali`功能。

![qq_pic_merged_1712767977652.jpg](https://attach.52pojie.cn/forum/202404/11/005305hwao39bzj3oe3azb.jpg)

`qq_pic_merged_1712767977652.jpg`（194.54 KB，下载次数：0；2024-4-11 00:53 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY2NHwwNWU0Yjk0YXwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

![qq_pic_merged_1712768030991.jpg](https://attach.52pojie.cn/forum/202404/11/005354gbvz7lb7vj6qqhrl.jpg)

`qq_pic_merged_1712768030991.jpg`（198.05 KB，下载次数：2；2024-4-11 00:53 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY2NXwxNjgzYmFmNXwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

先定义一个字符串类型的变量并赋值，用`isEmpty`方法去判断并获取结果，写好后点击右上角保存图标右边的转换图标，获取到`smali`语法的`isEmpty`语句，复制过去。

![qq_pic_merged_1712768731998.jpg](https://attach.52pojie.cn/forum/202404/11/010831so4mxvsxa1o11lol.jpg)

`qq_pic_merged_1712768731998.jpg`（57.09 KB，下载次数：1；2024-4-11 01:08 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY2Nnw4ZmJmYzRkN3wxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

![qq_pic_merged_1712768807318.jpg](https://attach.52pojie.cn/forum/202404/11/010851nvsb33b1vfft9rtk.jpg)

`qq_pic_merged_1712768807318.jpg`（46.48 KB，下载次数：0；2024-4-11 01:08 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY2N3wyNTdkMzY0NnwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

复制到获取`url`的语句下面，再加上判断`isEmpty`结果的语句和跳转。

每一句的含义看它上面的注释。

![qq_pic_merged_1712770302606.jpg](https://attach.52pojie.cn/forum/202404/11/013325z053j5bwoi2bq0t5.jpg)

`qq_pic_merged_1712770302606.jpg`（68.93 KB，下载次数：0；2024-4-11 01:33 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY3Nnw1Y2U3NTBjZnwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

## 创建URL的代码

再来看创建`url`的代码怎么写，先要获取`taskId`，我们可以调用`getTaskId`这个方法，把这个方法返回的结果赋给一个变量，再定义两个变量，分别赋`统一开头`和`统一结尾`，最后再按照次序把`统一开头`变量、`taskId`变量、`统一结尾`变量拼接起来，把拼接的结果赋给另一个变量，作为`getUrl`这个方法的返回结果。

还是一样，我不会写`smali`语句，先写`java`再转。

![qq_pic_merged_1712773178837.jpg](https://attach.52pojie.cn/forum/202404/11/022001ahmyd4y8ombmvitr.jpg)

`qq_pic_merged_1712773178837.jpg`（57.32 KB，下载次数：0；2024-4-11 02:20 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY4MHw4ZjhhNmYxZnwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

![qq_pic_merged_1712774314138.jpg](https://attach.52pojie.cn/forum/202404/11/023846e68w6jrw8ac2ttzn.jpg)

`qq_pic_merged_1712774314138.jpg`（80.67 KB，下载次数：1；2024-4-11 02:38 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY4MXw1NGMzNzYxNnwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

## 修改smali语句

看懂代码意思后就复制过去，再做修改。注意两个地方，一是我在`java`里写的`getTaskId`方法要由静态改为动态，因为我自己写的版本没有动态获取`taskId`，所以那时候必须用`static`修饰方法名，修改时要将`getTaskId`那句`smali`语句的`invoke-static {}`改为`invoke-virtual {p0}`。

二是全类名修改，还是定位到`getTaskId`这句`smali`语句，修改前的全类名是`LMain;`要改为`Lcom/liulishuo/lingodarwin/profile/freetalk/FreeTalkItem;`。

调用`getTaskId`的`smali`语句修改前后对比：

### 修改前

![qq_pic_merged_1712775093848.jpg](https://attach.52pojie.cn/forum/202404/11/025301haa7i7vnivui100p.jpg)

`qq_pic_merged_1712775093848.jpg`（29.63 KB，下载次数：0；2024-4-11 02:53 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY4MnwyZjFiMTNiM3wxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

### 修改后

![qq_pic_merged_1712775129493.jpg](https://attach.52pojie.cn/forum/202404/11/025317znaa4o147i3ubnol.jpg)

`qq_pic_merged_1712775129493.jpg`（23.78 KB，下载次数：0；2024-4-11 02:53 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY4M3xmNDI1ZDYyNnwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

此外还要修改`统一开头`、`统一结尾`这些字符串的值，改为实际网址，并且要调整寄存器的编号，从`getUrl`这一方法从前往后逐行逐个看，调整至符合逻辑。

修改好后完整代码如下：

![qq_pic_merged_1712776559164.jpg](https://attach.52pojie.cn/forum/202404/11/031628ovsbz2bx31ezdrvs.jpg)

`qq_pic_merged_1712776559164.jpg`（103.85 KB，下载次数：0；2024-4-11 03:16 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY4NHxhYzM1YWYzOXwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)

## 验证

修改完后保存，签名，一气呵成，没有签名校验，成功启动。再登录账号，点击情境实战，已全部解锁，有效可用。

![Screenshot_20240411-031928_-.jpg](https://attach.52pojie.cn/forum/202404/11/031948sp3j9k1mpyu3jj7u.jpg)

`Screenshot_20240411-031928_-.jpg`（318.05 KB，下载次数：1；2024-4-11 03:19 上传）[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODY4NXxhY2M0Y2JiOHwxNzkzNTgyMTMxfDE2ODA4NnwxOTEyMTc1&nothumb=yes)
