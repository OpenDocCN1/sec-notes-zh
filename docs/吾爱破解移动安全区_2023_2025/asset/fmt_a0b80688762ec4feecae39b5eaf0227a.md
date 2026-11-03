
# [Android 脱壳] X-plore 4.38.34 Google加固（`pairip_utility.py` 已更新）

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1956850>
- 抓取时间：2026-11-01T16:39:53.180Z

mt论坛发布相同内容的也是我

## 操作步骤

**1.** 将 `apks` 转为 `apk`。

**2.** 在 `termux` 中运行 `python pairip.py`，选择 `y` > 选择 `a`，等待运行完成。

> 注：`termux` 安装 `python`，`pkg install python`。

**3.** 将第 2 步执行完成后得到的 `apk` 中的 `dex` 添加到第 1 步得到的 `apk` 中，以下称这个 `apk` 为 `Xplore.apk`。

**4.** 翻译模式将 `mtd` 应用到 `Xplore.apk` 中的 `classes3.dex`（通用性补充：最后一个 `dex`）。

> 注：将 `mtd` 复制到 `/storage/emulated/0/MT2/dictionary/`。

![](https://attach.52pojie.cn/forum/202408/22/115253rtwkqhksxkecsl10.jpg)

[得到 `mtd` 的方法，需要光速[虚拟机](https://www.52pojie.cn/thread-661779-1-1.html)，lucky patcher。打开 lucky patcher，应用 android 核心[破解](https://www.52pojie.cn/)，应用前两个，重启虚拟机，应用第三个，导入安装 `Xplore.apk`，打开运行，导出 `/storage/emulated/0/MT2/dictionary/`。]

**5.** 将 `AndroidManifest.xml` 中的 `com.pairip.application.Application` 改为 `com.lonelycatgames.Xplore.App`。

通用方法：`dex++` 打开全部 `dex`，按照以下步骤操作：

![](https://attach.52pojie.cn/forum/202408/22/113709oawyhyahaada6hw6.jpg)

![](https://attach.52pojie.cn/forum/202408/22/113754wljrwwrobvomrzs2.jpg)

![](https://attach.52pojie.cn/forum/202408/22/115735ck2pkmmqmhk8hk8b.jpg)

![](https://attach.52pojie.cn/forum/202408/22/113911l4f1rx1w0q1rpr7x.jpg)

**6-1.** `dex++` 打开全部 `dex`，删除 `pairip`、`mt`。

![](https://attach.52pojie.cn/forum/202408/21/210229jpzn8x8uhhegzn15.jpg)

**6-2.** 进行以下操作：

**6-2-1.** 正则搜索（代码），替换为以下：

```asm
(# direct methods\n.method public static )appkiller\(\)V([\s\S]*?.end method)[\w\W]*
```

替换为：

```asm
$1constructor ()V$2
```

**6-2-2.** 正则搜索（代码），替换为以下：

```asm
sget-object.*\s+.*const-string v1,(.*\s+).*.line.*\n+.+.*\n.*invoke-static \{v0\}, Lmt/Objectlogger;->logstring\(Ljava/lang/Object;\)V
```

替换为：

```asm
const-string v0,$1
```

**6-2-3.** 正则搜索（代码），替换为以下：

```asm
(\.method public.*onReceive\(Landroid/content/Context;Landroid/content/Intent;\)V\n    \.registers) .[\s\S]*const-string/jumbo.*\s+.*\s+.*\s+(return-void)
```

替换为：

```asm
$1 3\n    $2
```

**6-2-4.** 搜索（代码）6-2-4-1，在结果中正则搜索（代码）6-2-4-2，替换为空。

**6-2-4-1.** 搜索（代码）：

```asm
pairip
```

**6-2-4-2.** 正则搜索（代码）：

```asm
.*pairip.*
```

**7.** 删除加固特征，`libpairipcore.so`（不同架构都有这个动态链接库）。

![](https://attach.52pojie.cn/forum/202408/22/143559kekqmr6246aem6jk.jpg)

`com.lonelycatgames.Xplore-20240821194038306.mtd`

`pairip_utility.py` https://wwqd.lanzoul.com/iV7UG2e4y50f
