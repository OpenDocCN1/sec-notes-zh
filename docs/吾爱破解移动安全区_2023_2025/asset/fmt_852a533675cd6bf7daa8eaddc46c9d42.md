
```
Rinitcall_debugLinux version 4.14.295-g84b42e6a786c-ab9578266 (android-build@abfarm-2004-0229) (A!� (7284624, based on r416183b) clangv�12.0.5 (https://d�H.googlesource.com/toolchain/llvm-project c935d99d7cf2016289302412d708641d52d2f7ee), LLDo/�tbot/srcuZi9outmO/lldq�!) #1 SMP PREEMPT Wed Feb 8 04:47:36 UTC 2023
```

其中`4.14.295`在 `Markfile`中

![](https://attach.52pojie.cn/forum/202307/07/042821bqq0rd8989j8gnj9.png)

image05.png (143.52 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYyNTY2M3xhMGY4NmI0OHwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY2M3xhMGY4NmI0OHwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

而 `84b42e6a786c`是提交的id,可以进入`private/msm-google`使用 `git log 84b42e6a786c`进行查看:

![](https://attach.52pojie.cn/forum/202307/07/042823onm41fhiqdgiuvnh.png)

image06.png (54.26 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYyNTY2NHw5MGY2ZDRkNnwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY2NHw5MGY2ZDRkNnwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

如果你想完全一致,可以使用`git checkout 84b42e6a786c`来检出这个提交。

所以现在,你应该会带着骂人的话,重新再刷一遍手机,以达到 手机系统的内核与你编译的内核一致

> 

如果刷机出现了Cannot load Android system. Your data may be corrupt....提示数据损坏等消息,

它会让你 `erase`（擦除） `userdata` 选择一下就可以。这可能是跟降级更新有关系。好吧,这又是我踩到的坑

## 测试自编译内核

测试的内核不包含`kernelSu`,没错,我删掉重新编译了,你们可以不删除直接测。

按照下面命令进行测试内核:

```
adb reboot fastboot

fastboot reboot bootloader

fastboot boot Image.lz4-dtb
```

但是此时你会发现,卡在启动log页面了

## 解决内核模块与内核对应问题

通过 `dmesg` 查看内核日志发现了大量的:

```
...
[   28.861688] init: Control message: Could not find 'android.hardware.sensors@2.0::ISensors/default' for ctl.interface_start from pid: 652 (/system/bin/hwservicemanager)
[   28.861963] adsprpc: bad ioctl: -1072934383
[   28.871840] adsprpc: bad ioctl: -1072934383
[   28.883564] adsprpc: bad ioctl: -1072934383
[   28.892785] adsprpc: bad ioctl: -1072934383
[   28.899394] adsprpc: bad ioctl: -1072934383
[   28.915591] adsprpc: bad ioctl: -1072934383
[   28.924410] adsprpc: bad ioctl: -1072934383
[   28.927512] adsprpc: bad ioctl: -1072934383
[   28.947871] adsprpc: bad ioctl: -1072934383
```

跟踪源码得知某个 case条件没有匹配上。

![](https://attach.52pojie.cn/forum/202307/07/042825tq7r1kqwqq7i4ny7.png)

image07.png (72.19 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYyNTY2NXxkN2MyZGVkM3wxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY2NXxkN2MyZGVkM3wxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

那么肯定是 vendor的内核模块 与 kernel启动内核不匹配,供应商修改了 内核代码。

这应该就是内核碎片话问题。 下游 vendor 调用上游 kernel代码不匹配

我想过几种方案啊:

- 
修改vendor分区

(尝试 `mount` 进行修改,可惜是只读的,放弃)

- 
手动编译vendor (看不懂 `Markfile` 配置,放弃)

- 
修改版本检测代码(尝试修改 `modversions` 和 `module magic` 校验,失败,原因不明,但是错误日志确实少了很多)

以上全部失败,耗费好几天时间,把我折磨的欲仙欲死,一度想放弃

最后研究如何将内核模块整合在 `boot`里面,可算有了思路。

两种方案:

- 将内核模块直接整合到启动内核中

- `ramdisk` 整合

当然这两种思路并不来自于我

## 将内核模块直接整合到启动内核中

这一种来自于 debug_cat 的 [//mp.weixin.qq.com/s?__biz=MzU1NjYyNjA3MQ==&mid=2247485043&idx=1&sn=f0c7f008d27851ee0d88780711eebb21&chksm=fbc37f8bcc4f69d666182c9a22b97dc7776c62fc67303bb08dcf75944e2637649aa2ee66028&mpshare=1&scene=23&srcid=0706c2VKK7bFY5X50Qc7aQEU&sharer_sharetime=1688653779744&sharer_shareid=f3dd87dafbf3fa6c2772d66af5f1c719#rd" target="_blank" rel="noopener noreferrer nofollow"](https://mp.weixin.qq.com/s?__biz=MzU1NjYyNjA3MQ==&mid=2247485043&idx=1&sn=f0c7f008d27851ee0d88780711eebb21&chksm=fbc37f8bcc4f69d666182c9a22b97dc7776c62fc67303bb08dcf75944e2637649aa2ee66028&mpshare=1&scene=23&srcid=0706c2VKK7bFY5X50Qc7aQEU&sharer_sharetime=1688653779744&sharer_shareid=f3dd87dafbf3fa6c2772d66af5f1c719#rd:rel="nofollow noopener" href="https)

其原理是像`kernelsu`那样直接将 驱动模块 直接打包进内核中

![](https://attach.52pojie.cn/forum/202307/07/042848edvudpe7oq7euwe5.png)

image17.png (150.06 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYyNTY3NXw0MDRiNjg5MHwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY3NXw0MDRiNjg5MHwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

其`M`代表打包成内核模块,在运行时候进行加载和卸载,将`M`改成`*`号打包进内核中。

由于文章写的很详细了,我就不重复叙述了,亲测是可行的,具体可以参考: [//mp.weixin.qq.com/s?__biz=MzU1NjYyNjA3MQ==&mid=2247485043&idx=1&sn=f0c7f008d27851ee0d88780711eebb21&chksm=fbc37f8bcc4f69d666182c9a22b97dc7776c62fc67303bb08dcf75944e2637649aa2ee66028&mpshare=1&scene=23&srcid=0706c2VKK7bFY5X50Qc7aQEU&sharer_sharetime=1688653779744&sharer_shareid=f3dd87dafbf3fa6c2772d66af5f1c719#rd" target="_blank" rel="noopener noreferrer nofollow"](https://mp.weixin.qq.com/s?__biz=MzU1NjYyNjA3MQ==&mid=2247485043&idx=1&sn=f0c7f008d27851ee0d88780711eebb21&chksm=fbc37f8bcc4f69d666182c9a22b97dc7776c62fc67303bb08dcf75944e2637649aa2ee66028&mpshare=1&scene=23&srcid=0706c2VKK7bFY5X50Qc7aQEU&sharer_sharetime=1688653779744&sharer_shareid=f3dd87dafbf3fa6c2772d66af5f1c719#rd:rel="nofollow noopener" href="https)

## ramdisk 整合

这种来自另外一个大佬 ,seeFlowerX的:https://blog.seeflower.dev/archives/174,写的也非常详细。

我对这种方式抽出了脚本,具体需要先了解一些知识:

### 了解供应商ramdisk模块加载位置

参考资料: https://source.android.com/docs/core/architecture/kernel/kernel-module-support?hl=zh-cn

通用内核映像 (`GKI`) 可能不包含使设备能够装载分区所需的驱动程序支持。为了使设备能够装载分区并继续启动，增强了第一阶段 `init`，用于加载 `ramdisk` 上的内核模块。`ramdisk` 被拆分为通用 `ramdisk` 和供应商 `ramdisk`。供应商内核模块存储在供应商 `ramdisk` 中。内核模块加载的顺序可以配置。

注意一些主要内容: `ramdisk` 拆分成 通用 `ramdisk` 和 供应商 `ramdisk`。 供应商内核模块存储在供应商 `ramdisk` 中

关于模块位置的描述:

（作为供应商 `ramdisk` 存储在供应商启动分区中）包含以下组件：

- 第一阶段 `init` 供应商内核模块，位于 `/lib/modules/`

- `modprobe` 配置文件，位于 `/lib/modules/`：`modules.dep`、`modules.softdep`、`modules.alias`、`modules.options`

- 一个 `modules.load` 文件，用于指示在第一阶段 `init` 期间加载的模块及相应的加载顺序，位于 `/lib/modules/`

- 供应商恢复内核模块，用于 A/B 和虚拟 A/B 设备，位于 `/lib/modules/`

- `modules.load.recovery`，用于指示要加载的模块及相应的加载顺序，用于 A/B 和虚拟 A/B 设备，位于 `/lib/modules`

第二个 `cpio` 归档（作为 `boot.img` 的 `ramdisk` 随 `GKI` 提供，应用在第一个 `cpio` 归档之上）包含 `first_stage_init` 及其依赖的库。

问题来了!!

`ramdisk` 是啥？它在哪里？

首先,关于`ramdisk`我并没有在 `android sources` 中搜索到满意的描述,可能是属于`linux`内容？但是按照查找的资料来说 `ramdisk` 是种基于

内存的磁盘,用于提升固定的访问速度。 在 `android`中将`ramdisk`做为 开机时候的启动项,里面包含了`init` 作为用户进程第一个执行程序。

`ramdisk` 可以看官方的描述:

```
https://source.android.com/docs/core/architecture/partitions/generic-boot?hl=zh-cn#boot-images-contents
```

通用 `ramdisk` 包含以下组件。

`init`

`system/etc/ramdisk/build.prop`

`ro. PRODUCT .bootimg.* build道具`

`debug_ramdisk/` 、 `mnt/` 、 `dev/` 、 `sys/` 、 `proc/` 、 `metadata/`

`first_stage_ramdisk/`

`debug_ramdisk/` 、 `mnt/` 、 `dev/` 、 `sys/` 、 `proc/` 、 `metadata/`

官方说了一段话 （作为 `boot.img` 的 `ramdisk` 随 `GKI` 提供，应用在第一个 `cpio` 归档之上）包含 `first_stage_init` 及其依赖的库。,有点绕口,但是抓住核心问题

通用 `ramdisk` 是在 `boot.img` 中,使用 `android image kitchen` 解包后确实有这个文件:

![](https://attach.52pojie.cn/forum/202307/07/042827rsnfcn1n9nh9onee.png)

image08.png (81.64 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYyNTY2Nnw4ZjQ4Yzk3M3wxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY2Nnw4ZjQ4Yzk3M3wxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

打开看看:

![](https://attach.52pojie.cn/forum/202307/07/042829f7egpeeoes4ezsea.png)

image09.png (145.95 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYyNTY2N3w2YzllN2Q3OXwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY2N3w2YzllN2Q3OXwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

发现它跟我们`cd /`查看手机根目录的内容差不多。 并且按照网上一些零散的资料来看,通用 `ramdisk`会挂载到根目录

但是供应商 `ramdisk`在哪里呢？

`ramdisk`一般采用 `cpio`归档,我们查看 `android kernel` 的 `build/build.sh` 脚本 搜索`.cpio`,你会发现:

![](https://attach.52pojie.cn/forum/202307/07/042831sfjjfjlr80f9o968.png)

image10.png (144.27 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYyNTY2OHwwYmZjMTNlMnwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY2OHwwYmZjMTNlMnwxNzkzNTY5MzA5fDE2ODA4NnwxODA1Njg0&nothumb=yes)

2023-7-7 04:28 上传

它会将`initramfs.cpio`打包成`initramfs.img`。

`initramfs.cpio`就是供应商的`ramdisk`,那么 `initramfs.cpio`在哪里呢?
