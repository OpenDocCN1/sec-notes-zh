
# [Android 原创] 实战教学-某APP 反Root环境检测

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1953146
- 抓取时间: 2026-11-01T16:40:47.212Z

## 前言

声明：本文仅供学习交流使用，所涉及的APP破解版不提供下载渠道。所涉及的技术请勿用于非法活动，否则所带来的一切后果自负。

## 使用工具

● `NP管理器`

## 开始实战

打开APP就有环境检测

![](https://attach.52pojie.cn/forum/202408/10/053602rluohozc1jhlnjm8.jpg)

管理器打开受害者

![](https://attach.52pojie.cn/forum/202408/10/053607u1m1kmzss7zfzcs5.jpg)

查看apk信息

![](https://attach.52pojie.cn/forum/202408/10/053610e33988wd1131mm1s.jpg)

选择`resources`文件

![](https://attach.52pojie.cn/forum/202408/10/053614tuqju55kf2z5tack.jpg)

`Arsc`编辑

![](https://attach.52pojie.cn/forum/202408/10/053618r8ggap19bgp19w79.jpg)

常量搜索

![](https://attach.52pojie.cn/forum/202408/10/053621r33r3152333oe233.jpg)

搜索“环境”

![](https://attach.52pojie.cn/forum/202408/10/053625l1g61fs6z4xowswu.jpg)

搜索结果，后面发现是`flutter`和原生Android混合开发

![](https://attach.52pojie.cn/forum/202408/10/053629zywpvom45cvo5dmq.jpg)

复制`toast_app_fail`

![](https://attach.52pojie.cn/forum/202408/10/053634fba2681j8cm8p86b.jpg)

返回`resource`搜索代码`toast_app_fail`

![](https://attach.52pojie.cn/forum/202408/10/053637auh57c2ffhczfmfr.jpg)

搜索到`0x7f0f00d7`

![](https://attach.52pojie.cn/forum/202408/10/053641pnmhe9ga1ghb9ae1.jpg)

返回`dex`

![](https://attach.52pojie.cn/forum/202408/10/053651epx9znunwa2y9aov.jpg)

`dex++`编辑

![](https://attach.52pojie.cn/forum/202408/10/053655vjduly8mxdyiqj79.jpg)

全选`dex`

![](https://attach.52pojie.cn/forum/202408/10/053658ql3xlwxlclwnqil5.jpg)

搜索十六进制`0x7f0f00d7`

![](https://attach.52pojie.cn/forum/202408/10/053701vo37vn9gzwvdd1n9.jpg)

搜到结果`0x7f0f00d7`的使用

![](https://attach.52pojie.cn/forum/202408/10/053705bpz8mepenhr6pypa.jpg)

进入`MainActivity`并定位到`g()`方法

![](https://attach.52pojie.cn/forum/202408/10/053708yl5zdptpr2zjrb62.jpg)

反编译为Java代码分析逻辑

![](https://attach.52pojie.cn/forum/202408/10/053712xdbhgntwd09htnnb.jpg)

```
public final boolean v() {
//该方法检测签名信息
PackageManager packageManager = getPackageManager();
m.d(packageManager, "this.packageManager");
Signature[] signatureArr = packageManager.getPackageInfo(getPackageName(), 64).signatures;
m.d(signatureArr, "packageInfo.signatures");
return signatureArr[zxsq-anti-bbcode-0].hashCode() == 574540939;
}
```

搜索类`DeviceUtils`

![](https://attach.52pojie.cn/forum/202408/10/053715l7oai7g8zco898av.jpg)

搜索到`DeviceUtils`

![](https://attach.52pojie.cn/forum/202408/10/053719q03dtr4b3dp4t3bz.jpg)

反编译`DeviceUtils`类代码，`b()`方法检测文件中是否存在Root，`antiToot`是个`native`方法，调用`so`库

![](https://attach.52pojie.cn/forum/202408/10/053722l2zr7ryvydeh9ra2.jpg)

依次搜索方法查看调用处是否只有一处

![](https://attach.52pojie.cn/forum/202408/10/053725bhcnf22smhnq1h1x.jpg)

确认`DeviceUtils`的两个方法只有`MainActivity`一处调用

![](https://attach.52pojie.cn/forum/202408/10/053732fsj7faac9dcvaspv.jpg)

回到`MainActivity`分析，所有`return false`都调用`v1`寄存器，所以将`v1`寄存器的值修改为`true`，即`const v1, 0x1`

![](https://attach.52pojie.cn/forum/202408/10/053729w6f0uj2h366hppb2.jpg)

保存代码

![](https://attach.52pojie.cn/forum/202408/10/053736gq5sm77o9ut0ohgn.jpg)

反编译回Java可以验证修改结果

![](https://attach.52pojie.cn/forum/202408/10/053739lzffw8rbdbbqr8bs.jpg)

退出修改并保存代码

![](https://attach.52pojie.cn/forum/202408/10/053743nf9rjfmf9f9mz5ep.jpg)

保存并编译文件

![](https://attach.52pojie.cn/forum/202408/10/053749vwfwggww3ccp222f.jpg)

打开功能

![](https://attach.52pojie.cn/forum/202408/10/053759tzlmsbtprrsrjnrt.jpg)

使用APK签名

![](https://attach.52pojie.cn/forum/202408/10/053803pfx9cxg9xx3www53.jpg)

签名并重新安装APK

![](https://attach.52pojie.cn/forum/202408/10/053756cu1zbubuh3y48850.jpg)

修改完成可以打开

![](https://attach.52pojie.cn/forum/202408/10/061449vt7hiy1ql7dhfo8t.png)
