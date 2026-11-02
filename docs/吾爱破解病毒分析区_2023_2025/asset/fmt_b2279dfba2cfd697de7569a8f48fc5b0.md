
![](https://attach.52pojie.cn/forum/202505/21/180227i8gz4qzq5rd25di8.png)

Image-15.png (173.53 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM4N3w5NzVkOGRlMXwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

Image-16.png (625.17 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM4OHxlNTI4ZTkzNnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

# 洗号

## 二、“赛博花柳”的辨别与自救

### 带毒程序的特征

#### 特征一：文件结构异常

带毒的安装包通常结构单一，仅为一个`.exe`文件（无其他CG、音频、剧本等文件）。其压缩包内常出现可疑命名的程序文件，如：`Galgame_loader.exe`、`STEAM-LOGIN-UNLOCK.exe`、`解锁补丁点击我.bat`、`CG自动解锁.vbs`。

#### 特征二：特殊条件要求

- 下载指定`7z`版本的解压程序（实际上是需要利用旧版本软件中的漏洞执行木马）。
- 关闭指定杀软（不要相信“误报”，此类提示的软件有90%含有病毒）。
- 重新登录Steam账户（一般是为了借机窃取Steam登录凭证）。

#### 特征三：运行时异常

执行文件后没有界面或弹出伪装的“登录窗口”。

- 弹窗提示输入Steam账号、密码、手机令牌（此为高危操作）。
- 要求关闭杀毒软件或退出防火墙运行。
- 无响应或提示“正在验证资源权限”，实际上在后台联网上传数据。

### 快速自救

切记，时间是保障安全的关键！若怀疑或发现Steam账号被盗，请立刻按照以下步骤操作，以最大限度降低损失(Steam令牌已被绕过)、防止攻击者的进一步入侵。

1. 立即切断网络并关闭当前系统活动

   若怀疑是后门、远控、键盘记录器，应立刻：

   - 关闭Wi-Fi或拔掉网线
   - 不再输入任何账号和密码
   - 退出所有网页、退出Steam等程序

2. 使用另一台安全的设备登录Steam官网修改密码

   尽量使用未中毒的新手机或干净电脑执行以下操作：

   - 立即修改密码，切断当前攻击者会话
   - 修改邮箱密码，防止被同步控制
   - 清空授权设备

3. 检查库存、交易记录、邮箱改绑等情况

   若库存物品（饰品、游戏）已被转移，请：

   - 打开Steam交易历史
   - 尽快记录交易对象和时间，并联系Steam客服进行申诉

## 三、样本分析

### 样本一

初始样本伪装成正常的游戏启动程序，通过《壁纸引擎》的创意工坊进行传播。

![](https://attach.52pojie.cn/forum/202505/21/180229oycbo0bkhkzck3sz.png)

Image-17.png (298.86 KB, 下载次数: 1) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM4OXw5NjNlNGU3MnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

壁纸引擎

病毒主体主要由Python编写，其中包含各种加载库文件用于伪装，真实的游戏文件保存在其中一个文件夹中。样本通过`AkaneChanToEcchi.exe`执行木马病毒和游戏程序。

![](https://attach.52pojie.cn/forum/202505/21/180232wiwawvwj7qnuu9qs.png)

Image-18.png (78.45 KB, 下载次数: 1) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5MHw2ZjYwNWMwYXwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

病毒主体

`AkaneChanToEcchi.exe`是一个Python程序加载器，通过读取`library.dat`文件执行Python类型的`pyc`文件。

![](https://attach.52pojie.cn/forum/202505/21/180234abv5ovsjo1l57s7t.png)

Image-19.png (78.03 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5MXw5NDk1ZTRiM3wxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

python loader

`library.dat`文件是一个`txt`文件，存储着内含大量`pyc`文件的压缩包`library.zip`。这些`pyc`文件由Python加载器加载执行，病毒主文件藏匿其中。

![](https://attach.52pojie.cn/forum/202505/21/180236rzmz3n4vaicngz4v.png)

Image-20.png (12.64 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5MnxhZTNhOWNlMXwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

`library.zip`

![](https://attach.52pojie.cn/forum/202505/21/180238f0zvoovvry00aj80.png)

Image-21.png (95.83 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5M3wxNDAwMDkzZnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

`pyc`文件

其中的`_main_2.pyc`文件为病毒主文件，该文件主要通过扫描内存中的`eya`密钥实施Steam盗号。

![](https://attach.52pojie.cn/forum/202505/21/180240g600q5rnqj2hq6ii.png)

Image-22.png (36.48 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5NHwwMDY3YTk1MHwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

获取Steam `PID`

样本利用获取到的Steam `PID`读取内存，并通过`PID`获取进程句柄。随后，匹配内存特征，获取`eya`密钥。

![](https://attach.52pojie.cn/forum/202505/21/180243e3j3wo5o9jhoy7iw.png)

Image-23.png (43.71 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5NXxiM2M4ZDdkM3wxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

![](https://attach.52pojie.cn/forum/202505/21/180245r2clic9lciccxxwn.png)

Image-24.png (48.04 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5NnwxYWMyYmM2ZHwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

内存获取`eya`密钥

之后，将从内存中获取到的`eya`密钥通过网络上传至`http://1.15.55.123/aerh.php`。

![](https://attach.52pojie.cn/forum/202505/21/180247xbqqnl0q08n8zvk8.png)

Image-25.png (40.2 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5N3xlYTE0MzMyMnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

上传`eys`密钥

样本文件夹还存在`inject.exe`与`inject.bin`两个PE文件类型的注入程序。它们采用`Ollvm`（包含虚假控制流、控制流平坦化）进行混淆，并实现进程注入。

![](https://attach.52pojie.cn/forum/202505/21/180249po584fdkfuffekff.png)

Image-26.png (135.79 KB, 下载次数: 1) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5OHw3NWEzNTVlNnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

`Ollvm`

通过`IDA-Python`对样本进行解混淆与分析发现，其主要是一个木马测试程序，它的注入方式以挂起进程为主。

![](https://attach.52pojie.cn/forum/202505/21/180251zqtrqp2uwk9k2zuk.png)

Image-27.png (41.47 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODM5OXxlNDYyN2U4YnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

木马测试程序

![](https://attach.52pojie.cn/forum/202505/21/180254ab184jnww8yu4qwj.png)

Image-28.png (106.57 KB, 下载次数: 3) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODQwMHwyNjM1N2E1N3wxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

进程注入

### 样本二

该样本在原有的基础上不断变种。早期变种通过更换不同版本的python加载器逃避检测，目前已发现其采用了Python 3.8、3.12、3.13等多个版本；最新变种则是利用`Nuitka`打包工具，将`py`文件转换成`c`文件，再将其编译打包成PE文件。

![](https://attach.52pojie.cn/forum/202505/21/180256irwq2w1q3gwosqgc.png)

Image-29.png (37.01 KB, 下载次数: 1) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODQwMXw0NTUzZWY0YnwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

`Nuitka`打包

该样本采用了与样本一相同的加载方式：通过`pyc`文件执行病毒和正常的游戏程序。

![](https://attach.52pojie.cn/forum/202505/21/180258l5epewkwampu7cof.png)

Image-30.png (105.85 KB, 下载次数: 2) · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODQwMnw1OTUyMjZiOXwxNzkzNTc2NjkzfDE2ODA4NnwyMDMzMTQy&nothumb=yes) · 2025-5-21 18:02 上传

启动病毒

对其盗号密钥进行分析发现，其获取密钥的方式也与样本一相同：通过创建`CMD`进程获取Steam进程的`PID`，接着通过打开进程句柄获取`eya`密钥。

![](https://attach.52pojie.cn/forum/202505/21/180300iw770033iam11owm.png)

Image-31.png (326.4 KB, 下载次数: 2)
