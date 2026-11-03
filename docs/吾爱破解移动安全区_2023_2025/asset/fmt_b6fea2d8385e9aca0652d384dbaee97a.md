
# [iOS 原创] 《挑战不用macOS逆向iOS APP》之两百元成本的iOS逆向环境搭建

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1769605
- 抓取时间: 2026-11-01T21:45:25

## Intro

很多人在学习逆向iOS app的时候，都有两个痛点：

- iOS设备太贵
- 需要macOS环境

前者需要小几千的iPhone，后者需要至少大几千的Macbook。本系列文章就是为了解决这两个痛点，全部操作在一百多块钱的iPhone6上完成，且电脑端在r0env/Win上进行演示，让我们康康，这几乎无成本的一套环境，究竟可以让我们走多远，走多深！

本篇文章是《挑战不用macOS逆向iOS APP》系列的第一课环境搭建主要为了实现在iOS APP逆向过程中一些环境方面的常规需求，具体实现目标如下：

- windows中iPhone基本信息获取/软件安装/投屏
- 越狱设备分析推荐/工具推荐/unC0ver流程详解
- 越狱后手机常用工具:日志/网络/包管理/免签名
- Frida/Objection/任意版本安装切换/动态分析
- ios砸壳/IPA静态分析基本流程/IDAF5函数定位
- linux中iphone手机信息获取:ID/名称/详情/签名/截屏/定位
- 安装APP/IPA:安装/卸载/升级/备份/回复APP
- 收发传输文件：`usbmuxd` `OpenSSH` = `adb pull/push`
- 远程文件管理:磁盘映射/电脑上浏览手机文件夹
- 升降级和激活:恢复模式/固件升降级/手机激活

## 1.Windows中iPhone基本信息获取/软件安装/投屏

### (1)基本信息获取

Windows中操作iPhone官方推荐方式是使用iTunes，但是后续我们会安装未经签名的应用，因此这里我们推荐使用爱思助手进行操作。直接去官网下载安装爱思助手，打开并在手机上信任此电脑。

![](https://attach.52pojie.cn/forum/202304/04/183959ngtr4wlrngr48xnx.png)

`202304041.png` (402.62 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwMHxjMDYwYzZlMXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

![](https://attach.52pojie.cn/forum/202304/04/183957gi1vaffqrqp999ip.png)

`202304042.png` (251.59 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5OXwyMWUzY2NmY3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

### (2)软件安装

软件安装：点开爱思助手的应用游戏可以直接安装，爱思上安装的软件有APP Store上架的也有未上架但有企业账号签名的，具体内容涉及IPA签名，会在后续文章解释。

![](https://attach.52pojie.cn/forum/202304/04/183954ujeykeeeeieejizi.png)

`202304043.png` (795.96 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5OHw4MjcxM2U0OHwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

### (3)手机投屏

直接使用爱思助手中的投屏，可以使用有线投屏，同一局域网下也可以使用无线投屏。

![](https://attach.52pojie.cn/forum/202304/04/183950oazdzm1qnufxqt2l.png)

`202304044.png` (537.86 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5N3xkNTdlNDIxMnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

手机端上滑点击屏幕镜像点击爱思投屏即可。

## 2.越狱设备分析推荐/unC0ver流程详解

### (1)越狱设备的分析与推荐

查看爱思助手的越狱方式可以看到

可以支持ios版本最高的是unc0ver方式可以支持到iOS14.8，而又由于iPhone越狱失败后重启设备失败的话需要恢复出厂设置，此时会自动更新到当前设备支持的最新iOS版本，而iPhone6最高版本是12.5.4，iPhone6s和iPhone7最新版本均是iOS 15，因此我们选择使用iPhone6作为越狱设备，这样即使越狱失败最新版本也依然在越工具支持版本之下。

越狱方式对比，这里主要分析两种方式unc0ver和Checkra1n

- checkra1n：比较复杂还要制作u盘但是胜在稳定
- unc0ver：过程简单但是成功比较看运气，需尝试多次才能成功

### (2)checkra1n越狱详细流程

#### （1）制作一个越狱u盘

![](https://attach.52pojie.cn/forum/202304/04/183947wwexgip476z66xej.png)

`202304045.png` (287.8 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5NnxiYWIwYjhiNXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

![](https://attach.52pojie.cn/forum/202304/04/183945yst6d56b5m3rt5b8.jpg)

`202304046.jpg` (61.99 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5NXw2ZjkxOTZlYXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

![](https://attach.52pojie.cn/forum/202304/04/183942oj0k93vakkj0k5cc.jpg)

`202304047.jpg` (48.29 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5NHwzMzIxMTZmNHwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

#### （2）刷入手机实现越狱

进入电脑BIOS选择`VendorCo ProductCode`从u盘启动

![](https://attach.52pojie.cn/forum/202304/04/183940hjlufxvbzd8fvbff.jpg)

`202304048.jpg` (329.05 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5M3xlZjBjODQzMXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

选择`ALT+F2`进入Checkra1n刷机系统

![](https://attach.52pojie.cn/forum/202304/04/183938vhkwuq0k1z1bh4fb.jpg)

`202304049.jpg` (160.19 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5MnxmMDEwODEzNnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

上下左右空格键控制`start`开始越狱

![](https://attach.52pojie.cn/forum/202304/04/183934h359evwbojo1s2o1.jpg)

`2023040410.jpg` (174.6 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDc5MXw5NDYxMzQ4N3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:39 上传

`next`

![](https://attach.52pojie.cn/forum/202304/04/184137avhvuim9mozcmuum.jpg)

`2023040411.jpg` (128.71 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgxM3w1ZjI0YjE4MnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

![](https://attach.52pojie.cn/forum/202304/04/184134zrh2iz1ww1h7p99r.jpg)

`2023040412.jpg` (130.1 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgxMnw4MmIzZDY3N3wxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

进入刷机界面后根据提示操作

![](https://attach.52pojie.cn/forum/202304/04/184131xfzhevm8kwk8frkz.jpg)

`2023040413.jpg` (157.16 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgxMXwxNTVmZjM0ZnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

1. 点击`start`
2. 同时摁住侧边键和home键
3. 摁住home键

这个界面就是在刷入，ALL Done就是刷入成功了。

![](https://attach.52pojie.cn/forum/202304/04/184127x9hmpqqmhtptwc9q.jpg)

`2023040414.jpg` (126.37 KB, 下载次数: 2)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgxMHxmYjQ4NTI0MXwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

补充：中间可能会失败，没关系多来几次总能成功的，但是一定注意，手机不要买到带锁机，二手iphone卖家有时会隐藏ID锁，这种机子可以登录自己ID但是一旦越狱失败就寄了……

## 3.越狱后手机常用工具:日志/网络/包管理/免签名

手机越狱成功后，会在桌面显示checkra1n图标，点击安装cydia，cydia是一个需要越狱后使用的三方软件仓库。这里我们主要安装以下几个工具：

### (1)cydia自带源工具安装

cydia自带源的工具可以直接搜索安装。

![](https://attach.52pojie.cn/forum/202304/04/184124g347o8h789490xo4.png)

`2023040415.png` (434.91 KB, 下载次数: 0)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwNDgwOXxjMTkzMDk1YnwxNzkzNTY5NTIzfDE2ODA4NnwxNzY5NjA1&nothumb=yes)

2023-4-4 18:41 上传

- `oslog`日志查看工具，默认安装在`/usr/bin`目录下
- `OPENSSH`远程连接软件，作用是远程连接手机，默认账号密码为`root/alpine`
- Filza File Manager是手机端的文件管理软件，作用是让我们更方便地操作手机上的文件。
- Apple File Conduit "2" 是IOS上的一个插件工具，作用是帮助我们在电脑端操作手机上的文件。

### (2)三方源工具安装

- AppSync Unifield

AppSync Unifield是IOS上的插件工具，作用是帮助我们安装未经苹果签名的IPA，安装后可以安装未经苹果签名的软件。安装具体流程如下：

首先添加源：编辑 -> 添加 -> 输入`cydia.angelxwind.net`

![](https://attach.52pojie.cn/forum/202304/04/184108m7y4l1q3qu77cq1t.png)

`20230404162.png` (716.4 KB, 下载次数: 0)
