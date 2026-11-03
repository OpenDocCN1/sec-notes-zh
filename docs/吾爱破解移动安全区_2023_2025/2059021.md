
# [Android 原创] Mac上AS模拟器安装面具

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=2059021](https://www.52pojie.cn/forum.php?mod=viewthread&tid=2059021)
- 抓取时间：2026-11-01T13:59:14.119Z

## 前提条件

- Mac电脑，M芯片，我的是M4
- Android Studio模拟器，下面简写AS
- 参考链接：[//xz.aliyun.com/news/11922](https://xz.aliyun.com/news/11922)
- 参考里的步骤大体相同，下面所用版本更新一些

## 准备资源

- rootAVD库：[//gitlab.com/newbit/rootAVD](https://gitlab.com/newbit/rootAVD)
    - 可`git`下载到本地
    - github上的库 [https://github.com/newbit1/rootAVD](https://github.com/newbit1/rootAVD)，不如这个gitlab上的版本新
- LSPosed-zygisk的zip文件
    - 《安卓逆向这档事》课件资源里有
    - 参考链接下载：[//magiskcn.com/lsposed-install.html](https://magiskcn.com/lsposed-install.html)
    - 我这里用的版本是`LSPosed-v1.9.2-7024-zygisk-release.zip`
- 算法助手Pro apk安装包：主要来验证LSPosed是否可用
    - 下载链接：[//www.123pan.com/s/7G8aTd-OU34H](https://www.123pan.com/s/7G8aTd-OU34H)
    - 《安卓逆向这档事》里有提到算法助手2.1.2，但在使用时会遇到文件夹权限问题，在github的库：[//github.com/Xposed-Modules-Repo/com.junge.algorithmaide](https://github.com/Xposed-Modules-Repo/com.junge.algorithmaide)
    - 算法助手Pro解决了Android高版本时的文件夹权限问题，建议直接使用
- `wuaipojie` apk安装包
    - 《安卓逆向这档事》第四课资源里有，用前几课里的也可以
    - 这里用其他APP也可以，主要为了配合算法助手APP使用看功能是否正常可用

## 操作步骤

### 1. root模拟器

1. 打开AS后，点击菜单，打开Virtual Device Manager，选择一个已建好的模拟器打开。
    - 建议模拟器选择带play store版本，更符合正常使用。
    - 我这里选择的是Pixel 7 API 33，带play store版本。
    - 不同机型可选择的API版本不同。
    - 我在使用Pixel 9 API 35时遇到了zygisk无法开启的问题。

    ![](https://attach.52pojie.cn/forum/202509/09/150930pgzsyshjfcsnrlac.png)

    ![](https://attach.52pojie.cn/forum/202509/09/151337g5ub5q0yn43bwmd2.png)

2. 进入到rootAVD工程后，执行命令：`./rootAVD.sh ListAllAVDs`，会展示命令行可用选项。
    - 下面有当前电脑上所有已建模拟器的可使用命令，注意版本型号。

    ![](https://attach.52pojie.cn/forum/202509/09/151457z4iy6wpyyxxbbwjo.png)

    ![](https://attach.52pojie.cn/forum/202509/09/151524g27lo0sr7458d7d2.png)

3. 根据刚才打开的模拟器版本选择执行命令：`./rootAVD.sh system-images/android-33/google_apis_playstore/arm64-v8a/ramdisk.img`。
    - 执行后终端里会显示执行内容，执行完毕后会显示成功提示，最后一行内容是Huge Credits and big Thanks to topjohnwu, shakalaca, vvb2060 and HuskyDG。
    - 命令执行完后，当前打开的模拟器会自动关闭，如果没有自动关闭，点击一下模拟器也应该会自动关闭。
    - 关闭时如果提示是否保存当前快照的话，可以选择YES。

4. 从Device Manager里选择刚才的模拟器，用Cold Boot的选项重启打开。
    - 一定注意！要选Cold Boot，否则直接点击启动按钮会导致模拟器打开后卡死。

    ![](https://attach.52pojie.cn/forum/202509/09/151800fcsxc9cu8tacoc30.png)

5. 打开后可上划模拟器页面看到很多应用，找到Magisk(也就是面具) APP打开。同意通知权限后，会弹出Requires Additional Setup为标题的弹窗，点击OK，模拟器会重启。

    ![](https://attach.52pojie.cn/forum/202509/09/154301d1s8bw7a8jdsbodo.png)

6. 打开Magisk，稍微等待几秒钟，首页会显示Magisk已被安装，Zygisk已被打开。点击右上角设置按钮左边的重启按钮，选择第一个选项Reboot，模拟器会重启。
    - 如果Zygisk显示未被打开，点击右上角的设置按钮，在设置页面里找到Zygisk选项打开。回到Magisk主页面选择重启。

    ![](https://attach.52pojie.cn/forum/202509/09/152046g8gzbfmafbcf78g6.png)

### 2. 安装LSPosed

1. 将`LSPosed-v1.9.2-7024-zygisk-release.zip`拖到模拟器里。
2. 打开Magisk，点击底部菜单栏里的Modules，点击Install from storage按钮，在新页面里点击左上角的菜单按钮，点击Downloads，会看到刚才拖进来的`LSPosed-v1.9.2-7024-zygisk-release.zip`。点击zip后会弹窗提示安装，点击OK安装。

    ![](https://attach.52pojie.cn/forum/202509/09/154315lu4modjvpnozn1pz.png)

3. LSPosed的zip安装完毕后，回到首页去点击重启。
    - 这时候如果在安装完毕页面点击下方出现的Reboot按钮也可以重启模拟器，但速度比较慢。

4. 模拟器重启后，打开Magisk，点击Modules，会看到Zygisk-LSPosded安装成功。
    - 这时候可以点击模拟器右边的菜单栏里的菜单按钮，选择Snapshots，点击TAKE SNAPSHOT保存当前手机快照。后面可以随时从Snapshots恢复到这个状态。
    - 这个Snapshot可以在你任何想保存手机当前状态的时候使用，以便恢复使用。

    ![](https://attach.52pojie.cn/forum/202509/09/154326e09u5asupyshek90.png)

    ![](https://attach.52pojie.cn/forum/202509/09/152401dg33gq4yz63zn68q.png)

### 3. 安装算法助手

在上面安装LSPosed成功后就可以结束了，这里安装算法助手是进一步验证下环境是否正常可用。

1. 将`wuaipojie` apk安装包拖到模拟器里安装。
2. 将算法助手Pro apk安装包拖到模拟器里安装，在安装完毕时，模拟器会弹出算法助手模块需要激活的提示弹窗，点击弹窗进入配置算法助手Pro页面。
    - 如果没来得及点击弹窗，可以下划顶部的通知栏，在LSPosed提示下面依然有相同提示可以点击进入配置。

    ![](https://attach.52pojie.cn/forum/202509/09/154338oiog9mp04yd0izpu.png)

3. 选中系统框架和wuaipojie。

    ![](https://attach.52pojie.cn/forum/202509/09/152534eva3erfaqra9r1cr.png)

4. 打开算法助手Pro，授予各种权限，点几个弹窗后会看到提示服务未激活。

    ![](https://attach.52pojie.cn/forum/202509/09/154357yqv31kh0ah5dcnvz.png)

5. 到Magisk app里点击重启。
6. 重启后，打开算法助手Pro，不再提示服务未激活了。点击wuaipojie右侧对应的按钮选中，然后点击wuaipojie进入到设置页面。

    ![](https://attach.52pojie.cn/forum/202509/09/154406ial23r553v8bf9z9.png)

7. 在设置页面可以看到算法分析、隐藏root等，这里进行一个简单的，在Ui相关区域打开弹窗定位选项，然后点击右上角的三角开始按钮就会启动wuaipojie。

    ![](https://attach.52pojie.cn/forum/202509/09/153035w5abjb363x5j3hs8.png)

8. 在wuaipojie里填写名字后，进入到挑战里的第三关去广告，让页面弹一些弹窗。
9. 点Home键回到模拟器主页，重新进入算法助手Pro，从设置页面后退到首页，点击Log栏，可以看到wuaipojie选项，点击进入日志页面，可以看到刚才hook到的在wuaipojie app里的几个弹窗。

    ![](https://attach.52pojie.cn/forum/202509/09/154418nqzlquweqfb33zdi.png)

    ![](https://attach.52pojie.cn/forum/202509/09/154426jddvvdy9tbnzcidt.png)

到此结束了。Magisk和算法助手的其他功能可自行探索或看论坛里的《安卓逆向这档事》课程学习。例如可以利用算法助手的拦截弹窗功能拦截掉某些软件的强制版本更新。
