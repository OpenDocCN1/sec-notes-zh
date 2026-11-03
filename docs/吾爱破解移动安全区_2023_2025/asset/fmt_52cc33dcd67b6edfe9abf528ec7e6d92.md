
# [Android 原创] android-pixel4a-刷机系列-(5)kernelsu编译问题解决及使用

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1805683
- 抓取时间: 2026-11-01T21:41:53

## android-pixel4a-刷机系列-(5)kernelsu编译问题解决及使用

上一节编译的`kernelSu`有点问题,本次就进行修复重新编译

### 重编译

上一篇出现 `savedefconfig` 不匹配的问题

```
....
--- private/msm-google/arch/arm64/configs/sunfish_defconfig        2023-06-27 15:56:28.068070234 +0800
+++ ~/aosp/android-kernel/out/android-msm-pixel-4.14/private/msm-google/defconfig        2023-06-27 22:37:59.857566122 +0800
@@ -685,7 +685,6 @@
 CONFIG_QUOTA_NETLINK_INTERFACE=y
 CONFIG_QFMT_V2=y
 CONFIG_FUSE_FS=y
-CONFIG_OVERLAY_FS=y
 CONFIG_INCREMENTAL_FS=m
 CONFIG_VFAT_FS=y
 CONFIG_TMPFS_POSIX_ACL=y
++ RES=1
++ '[' 1 -ne 0 ']'
++ echo ERROR: savedefconfig does not match private/msm-google/arch/arm64/configs/sunfish_defconfig
ERROR: savedefconfig does not match private/msm-google/arch/arm64/configs/sunfish_defconfig
++ return 1
```

之前我们的解决方案是 注释掉 `KernelSu` 的 `Kconfig` 中的 `select OVERLAY_FS` 选项

```
menu "KernelSU"
config KSU
        tristate "KernelSU function support"
        #select OVERLAY_FS
        default y
        help
        Enable kernel-level root privileges on Android System.
```

这样确实 ok 了。但总感觉这样不是正确的解决办法。

下面提供了解决方案

#### 4> 去掉 `check_defconfig`

这个参考了`debug_cat`的解决方案,亲测,确实方便有效！

![](https://attach.52pojie.cn/forum/202307/07/035622cev8viu4mdaluudz.png)
image01.png (148.06 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjYyNTY0N3xhZTUwMGRhOXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY0N3xhZTUwMGRhOXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

首先我们看下 `private/msm-google/build.config` 文件

```
KERNEL_DIR=private/msm-google
. ${ROOT_DIR}/${KERNEL_DIR}/build.config.common.clang
POST_DEFCONFIG_CMDS="check_defconfig"
```

发现他调用了 `check_defconfig` 方法,这个方法在哪里呢？

回到 `build` 目录下你会发现一个叫 `_setup_env.sh` 的文件

```
echo

echo "PATH=${PATH}"

echo

# verifies that defconfig matches the DEFCONFIG

function check_defconfig() {

    (cd ${OUT_DIR} && \
     make "${TOOL_ARGS[@]}" O=${OUT_DIR} savedefconfig)

    [ "$ARCH" = "x86_64" -o "$ARCH" = "i386" ] && local ARCH=x86

    echo Verifying that savedefconfig matches ${KERNEL_DIR}/arch/${ARCH}/configs/${DEFCONFIG}

    RES=0

    diff -u ${KERNEL_DIR}/arch/${ARCH}/configs/${DEFCONFIG} ${OUT_DIR}/defconfig ||

      RES=$?

    if [ ${RES} -ne 0 ]; then

        echo ERROR: savedefconfig does not match ${KERNEL_DIR}/arch/${ARCH}/configs/${DEFCONFIG}

    fi

    return ${RES}

}
```

它里面就有 `check_defconfig` 方法,可以看到它通过 `make ... savedefconfig` 从 `.config` 提取的 `defconfig`。
并使用 `diff` 进行比较差异。
这就难受了,因为我修改 `arch/arm64/configs/xxx_defconfig` 后,他会和默认值进行比较,无论如何都不相等的。
当然,砸门可以去掉这个。

查看 `build_sunfish.sh` 脚本

![](https://attach.52pojie.cn/forum/202307/07/035624xgag22zzgce2cm2a.png)
image02.png (42.13 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYyNTY0OHw1YTBmNWIyNXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY0OHw1YTBmNWIyNXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

它里面的配置指向了 `build.config.sunfish_no-cfi`,所以我们改这个

```
BUILD_CONFIG=private/msm-google/build.config.sunfish_no-cfi build/build.sh "$@"
```

改成如下

```
KERNEL_DIR=private/msm-google
. ${ROOT_DIR}/${KERNEL_DIR}/build.config.sunfish.common.clang
#去掉check_defconfig
#POST_DEFCONFIG_CMDS="check_defconfig && update_nocfi_config"
POST_DEFCONFIG_CMDS="update_nocfi_config"
function update_nocfi_config() {
  # Disable clang-specific options
  ${KERNEL_DIR}/scripts/config --file ${OUT_DIR}/.config \
    -d LTO \
    -d LTO_CLANG \
    -d CFI \
    -d CFI_PERMISSIVE \
    -d CFI_CLANG
  (cd ${OUT_DIR} && \
   make ${CC_LD_ARG} O=${OUT_DIR} olddefconfig)
}
```

只不过构建的时候不是调用 `build/build.sh` 了,而是直接调用 `build_sunfish.sh`

#### 4> kprobe 配置

`kernelSu` 通过 `kprobe` 去 `hook` 内核 实现的功能。所以我们得让编译的内核支持 `kprobe`

在 `arch/arm64/configs/xxx_defconfig` 添加如下配置

```
CONFIG_KPROBES=y
CONFIG_HAVE_KPROBES=y
CONFIG_KPROBE_EVENTS=y
```

![](https://attach.52pojie.cn/forum/202307/07/035627au6okoq8kyt8oequ.png)
image03.png (119.83 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjYyNTY0OXxhYWNjNWI3OXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY0OXxhYWNjNWI3OXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

#### 4> xxx_defconfig 正确配置方式

前面说的虽然也可以完成编译工作,但是我在浏览 `defconfig` 的时候发现了个不错的文章:

[//www.adtxl.com/index.php/archives/124.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.adtxl.com/index.php/archives/124.html:rel="nofollow noopener" href="https)

按照文章所指示,我们应该这样操作:

- `make ARCH=arm64 sunfish_defconfig` 生成 `.config` 文件
- `make ARCH=arm64 menuconfig` 根据 `.config` 中 `arm64` 进行配置

此时您应该进入了配置界面,输入 `/`,搜索 `KPROBE`。
这里你要记住几个地方,`KPROBES` 以及 `KPROBE_EVENTS` 这几个是没有开的:

![](https://attach.52pojie.cn/forum/202307/07/035637unjqi27pb6cbg12j.png)
image08.png (60.68 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjYyNTY1NHwzZDhmNTk1ZHwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY1NHwzZDhmNTk1ZHwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

![](https://attach.52pojie.cn/forum/202307/07/035646ynhnhth9upnekuik.png)
image09.png (70.77 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjYyNTY1NXw1ZjM3MDI4ZHwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY1NXw1ZjM3MDI4ZHwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

以此找到地方后,按空格进行选中:

![](https://attach.52pojie.cn/forum/202307/07/035648c7oybkaj39m55za9.png)
image10.png (88.12 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYyNTY1NnwxMTlhMmNmNHwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY1NnwxMTlhMmNmNHwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

对于 `KPROBE_EVENTS` 只要选择 `KPROBE` 就自动选中了:

![](https://attach.52pojie.cn/forum/202307/07/035650e8izxei5exy5udgz.png)
image11.png (63.95 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjYyNTY1N3xiYTQ2NWM1YnwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY1N3xiYTQ2NWM1YnwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

选择 `save` 进行保存

![](https://attach.52pojie.cn/forum/202307/07/035653o25kd7kk4svksvz5.png)
image12.png (27.28 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYyNTY1OHw4NTM1NTc3OXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyNTY1OHw4NTM1NTc3OXwxNzkzNTY5MzExfDE2ODA4NnwxODA1Njgz&nothumb=yes)
2023-7-7 03:56 上传

接下来,我们需要生成修改过的 `defconfig`,将 `sunfish_defconfig` 进行覆盖

- `make ARCH=arm64 savedefconfig` 根据 `.config` 生成 `defconfig`
- `cp defconfig arch/arm64/configs/sunfish_defconfig` 覆盖 `sunfish_defconfig`

> 

这样确实就没问题了,`check_defconfig` 也不用去掉,但是我也没搞明白为什么这样生成的 `defconfig` 为什么可以一直保持,
而手动修改的 `sunfish_defconfig` 确会导致 `check_defconfig` 不同

接下来重新进行构建

#### 4> 构建时候强调点问题

如果构建的时候出现了下面的错误:

```
aosp/android-kernel/private/msm-google is not clean, please run 'make mrproper'
```

这代表某些配置没有删除干净。

这段错误在 `/private/msm-google/Makefile` 中

```
# prepare3 is used to check if we are building in a separate output directory,
# and if so do:
# 1) Check that make has not been executed in the kernel src $(srctree)
prepare3: include/config/kernel.release
ifneq ($(KBUILD_SRC),)
        @$(kecho) '  Using $(srctree) as source for kernel'
        $(Q)if [ -f $(srctree)/.config -o -d $(srctree)/include/config ]; then \
                echo >&2 "  $(srctree) is not clean, please run 'make mrproper'"; \
                echo >&2 "  in the '$(srctree)' directory.";\
                /bin/false; \
        fi;
endif
```

意思大概是说,您需要保证 `.config` 和 `include/config` 是空的
使用 `rm -f .config` 和 `rm -rf include/config`。
如果发现你从未见过的错误,那代表你可能乱改了某些地方。可通过 `git checkout .` 重新检出覆盖下内容,那么你将会需要重新配置。
实在不行,全部清理重构,参考我下面的代码:
```
