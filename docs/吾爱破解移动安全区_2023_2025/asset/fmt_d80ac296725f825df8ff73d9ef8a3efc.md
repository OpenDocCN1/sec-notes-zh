
# [Android 原创] 某乖生活app桶自洁接口加密参数分析

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=2069979](https://www.52pojie.cn/forum.php?mod=viewthread&tid=2069979)
- 抓取时间：2026-11-01T23:54:33

## [Android逆向] 某乖生活app桶自洁接口加密参数分析

## 0x00 概述

针对某乖生活桶自洁下单问题，本文研究了app的下单接口并编写了自动在每日早上6:00批量对洗衣房机器桶自洁的实现。阅读本文你将会学习到如何抓包，脱壳，反编译，加密还原等逆向技术。本文仅用于学习交流，严禁用于非法用途。

关键词：`frida`、360加固、`jadx`、脱壳、`reqable`、Android逆向

## 0x01 缘起

在上海实习三个多月，回学校宿舍后，闻到柜子里一股霉味，于是将衣服拿出，走进宿舍的洗衣房发现，稍微干净一点的洗衣机都在工作中，只剩下脏乱差的洗衣机敞开着门，经过两遍桶自洁效果依旧微乎其微，心里反问，为什么供应商不在每日固定时间实现以下桶自洁，这样不是更干净吗？于是一怒之下，供应商不干的，我来干。

## 0x02 环境

- `frida` 16.2.1
- `jadx-gui` 1.5.3
- 某乖生活 1.96.1
- 红米note9 (root+magisk+lsposed)

其中，`frida` 网上的教程很多，不再赘述，只需要注意的是，使用魔改过的 `frida`，以便绕过检测，例如 [https://github.com/taisuii/rusda](https://github.com/taisuii/rusda)。

## 0x03 抓包

工具：`reqable`，官网地址：[https://reqable.com/zh-CN/](https://reqable.com/zh-CN/)。

![](https://attach.52pojie.cn/forum/202511/05/122039hmmw7spwpp8p8wr9.png)

`image.png` (123.13 KB, 下载次数: 0) · 2025-11-5 12:20 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5MHxmZDUzY2IxYnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

使用方法也相对简单，采用电脑端+手机端结合的方式，方便分析，手机端需要刷入面具模块，如下。

![](https://attach.52pojie.cn/forum/202511/05/122128ufol5gxhzgjweomr.png)

`image.png` (100.41 KB, 下载次数: 0) · 2025-11-5 12:21 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5MXxhYzliYWIxNHwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

直接开始抓包。

![](https://attach.52pojie.cn/forum/202511/05/122146pfsigvk22v591v3r.png)

`image.png` (302.16 KB, 下载次数: 0) · 2025-11-5 12:21 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5MnxhZDUzZDYyMnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

有如下结果：

- `/goods/scan/v2`：扫描机器上的二维码，获取设备信息
- `/goods/normal/details`：获取洗衣机详细信息
- `/goods/normal/skus`：获取洗衣方式（包括桶自洁）
- `/trade/create`：订单创建，待支付状态
- `/pay/unify/pay`：订单支付，启动洗衣机桶自洁

再看请求头，`Authorization` 为登录的账号 `token`，`imei` 为接口返回，`cookie` 不带即可，其中 `sign` 和 `orderRiskSign` 是检验参数。

![](https://attach.52pojie.cn/forum/202511/05/122158kbntrgkgnnn2alpl.png)

`image.png` (56.32 KB, 下载次数: 0) · 2025-11-5 12:21 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5M3w1MzAzM2QwM3wxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

再看请求体，`goodsId` 为设备 `id`，`skuId` 为洗衣方式的 `id`，`orgId` 可能是学校 `id`。

![](https://attach.52pojie.cn/forum/202511/05/122210rh252hh2l6mm522j.png)

`image.png` (19.97 KB, 下载次数: 0) · 2025-11-5 12:22 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5NHw1Njk1OGU0NXwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

对此，对app的逆向任务也有了小小的收获，接下来就是脱壳，反编译，还原加密参数。

## 0x04 脱壳 反编译

说到脱壳，首先想到的一定是大名鼎鼎的 `frida-dexdump`，它的核心逻辑是：

`attach` 到目标进程 → `hook` `dex2oat` 或 `LoadDex` → 从内存中读取 DEX 数据 → 保存到本地。

使用也相对简单，直接执行 `pip install frida-dexdump` 即可。

### 手机端启动 `frida-server`

在终端执行 `adb shell`，再执行 `su` 提权。把刚才下载的魔改 `frida` 放到 `/data/local/tmp` 下，执行 `chmod 777 ./fida` 授权，再执行 `./fida`。这里终端阻塞就是在运行。

![](https://attach.52pojie.cn/forum/202511/05/122245kgj0n1s7yt0001yw.png)

`image.png` (10.68 KB, 下载次数: 0) · 2025-11-5 12:22 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5NXwyNzVkMDkwMXwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

### `frida-dexdump` 脱壳

#### 安装

终端执行 `pip install frida-dexdump`，即可安装（推荐使用 `conda` 创建完虚拟环境，在虚拟环境中安装，因为在逆向任务中，可能会使用到不同版本的 `frida`）。

#### 脱壳

执行 `frida-dexdump -U -f com.qiekj.user`。

这里就遇到问题了，报错：

![](https://attach.52pojie.cn/forum/202511/05/122256ejj6sb1ee15bjeza.png)

`image.png` (318.45 KB, 下载次数: 0) · 2025-11-5 12:22 上传 · [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMTU5NnxhZGE2MGM2NnwxNzkzNTc3MjcxfDE2ODA4NnwyMDY5OTc5&nothumb=yes)

经分析，发现目标 app 采用 360 加固，而这种加固方式，通过反调试、内存保护、动态解密与映射机制阻止 `Frida` 或内存读访问，进而使逆向人员难以达到拿到关键 `dex`。
