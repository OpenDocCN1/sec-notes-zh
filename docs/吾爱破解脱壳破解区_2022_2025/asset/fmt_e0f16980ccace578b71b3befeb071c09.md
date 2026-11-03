
# [原创] 去除新版 SanDisk PrivateAccess 设备限制和自动更新并打包回原版格式

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2049710
- 抓取时间: 2026-11-01T22:06:12

去除 SanDisk PrivateAccess 6.4.13 设备限制和自动更新并打包回原版格式。过程细节与旧版稍有不同故开贴记录，方便新手学习。

## 准备工作

访问 [//support-cn.sandisk.com/app/products/downloads/softwaredownloads"](https://support-cn.sandisk.com/app/products/downloads/softwaredownloads:rel="nofollow noopener" href="https)，下载 SanDisk PrivateAccess，得到 `privateaccess-win.exe`

## 使用工具

- `x32dbg`
- Detect It Easy (`DIE`)
- `010 Editor` 或其它十六进制编辑器
- `Resource Hacker`

## 1. 寻找主程序

打开 `privateaccess-win.exe`，跳出提示 This application only runs on a SanDisk flash drive。用 `DIE` 查壳可知这个程序嵌套有两个资源文件。同时运行目录下多了一个 `WDPA` 文件夹，里面有一个 241 KB 的文件 `PrivateAccessMonitor.exe`。由此判断这可能是一个自解压文件。还有可能有一个文件夹叫做 `Temp`，里面有个文件 `PrivateAccess_win.exe`，这显然就是真正的主程序。如果没有也没关系，我们直接用 `DIE` 提取，如图：

![](https://attach.52pojie.cn/forum/202507/30/102518qwhp6fh9cbwz4eei.png)

1.png (201.85 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5MzIyM3w1YzMxYTQ2M3wxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzIyM3w1YzMxYTQ2M3wxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes)

`DIE` 提取主程序

2025-7-30 10:25 上传

提取可知第一个 `PE32` 文件就是主程序。

## 2. 脱壳

这个程序加了原版 `UPX` 壳，理论上直接用 `upx -d` 脱壳即可……

然后脱出来的程序打不开，原因是我自己的 `upx` 版本太老导致入口点改不回去……

更新一下就能正常脱壳了，程序也能正常启动。

打开 `x32dbg`。把 `xAnalyzer` 插件或者别的什么分析插件的 自动分析 (`Automatic analysis`) 关掉，否则无法开始调试。原因是这个程序小函数很多，分析时间巨长无比（`IDA 8.3` 分析了半个小时），并且 `xAnalyzer` 自动分析完成前无法开始调试。

拖入程序，但是 `x32dbg` 提示无法启动程序！错误原因是“访问无效的地址”……

所以我们还是要动用我们在吾爱破解第一课中学习的 `ESP` 定律手工脱壳大法！

拖入原始程序，按 `F7` 单步运行，在寄存器区找到 `ESP` 右击，选择“在内存窗口中转到”，在内存窗口右击左边的地址，选择断点、硬件访问、4 字节，`F9` 运行，此时 CPU 窗口跳转到另一处代码。再次右击刚才的地址，断点，删除硬件断点。在 CPU 窗口中选中下面的 `jmp` 指令，按 `F4` 运行到此，再按 `F7` 单步。此时就来到了 `OEP`。选择 `Scylla`（在工具栏上），看图脱壳（图中的 `Scylla` 是我汉化的，英文原版的看按钮位置一样就对了）：

![](https://attach.52pojie.cn/forum/202507/30/102521uek77qk76cackmzc.png)

2.png (120.2 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MzIyNHxmNjNmZTc1NHwxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzIyNHxmNjNmZTc1NHwxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes)

用 `Scylla` 脱壳

2025-7-30 10:25 上传

最终得到 `PrivateAccess_win_dump_SCY.exe`，脱壳完成，结束程序。

## 3. 破解

特别感谢 [//www.52pojie.cn/thread-1925640-1-1.html"](https://www.52pojie.cn/thread-1925640-1-1.html:href="https) 给的思路！

将脱壳得到的最终程序拖入运行，出现弹窗后点击暂停。在 CPU 窗口右击，搜索，所有用户模块，字符串。在搜索页最下面的过滤框中输入 `SanDisk` 即可找到提示字符串。双击提示字符串转到 CPU 窗口，选中上面的 `jne` 语句，按 `F2` 下断点，重启程序。此时可以看到 `jne` 下面就是那句报错，且此时“跳转不会执行”，因此按空格修改，将 `jnz` (`jnz=jne`) 改成 `jmp` 强制跳转，剩余字节是否以 `NOP` 填充其实无所谓，反正已经跳转了，不过为了下面指令好看我还是勾了。

![](https://attach.52pojie.cn/forum/202507/30/102523qax42ar3o54zr3jq.png)

3.png (41.31 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5MzIyNXwyYWQxMTAwM3wxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzIyNXwyYWQxMTAwM3wxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes)

修改指令

2025-7-30 10:25 上传

继续运行程序，程序正常启动说明修改正确。然后在 CPU 窗口右键，补丁，修补文件，保存为 `PrivateAccess_win2.exe`。关闭 `x32dbg`，直接运行修改后的程序，没有问题。

## 4. 移植

我们能不能把这个壳重新加回去呢？直接用 `UPX` 对手动脱壳的程序加壳是不行的，会报错。我们必须把刚才的补丁移植回 `UPX` 直接脱出的原始文件。

首先打开终端，定位刚才补丁的位置。执行：

```
fc /b PrivateAccess_win_dump_SCY.exe PrivateAccess_win2.exe
```

得到：

```
正在比较文件 PrivateAccess_win_dump_SCY.exe 和 PRIVATEACCESS_WIN2.EXE

00061476: 0F E9

00061477: 85 A1

00061478: A0 00

0006147B: 00 90
```

运行 `upx -d PrivateAccess_win.exe` 脱掉主程序的壳得到原始文件，把文件拖入 `010 Editor`。找到 `6:1470h:` 行，选择第 6 列，发现和文件对比给出的结果一致，因此从这里开始输入 `E9 A1 00 00 00 90`，如图：

![](https://attach.52pojie.cn/forum/202507/30/102525hbdd4lzo13l2j589.png)

4.png (24.82 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MzIyNnw5NzYwZWE4Y3wxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzIyNnw5NzYwZWE4Y3wxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes)

移植补丁字节码

2025-7-30 10:25 上传

另外软件可能会自动更新导致需要重新破解，所以我们要修改更新地址以去除更新，搜索 `privateaccess.stonehenge.sandisk.com`，从头开始用 `0.0.0.0/` 覆盖，如图：

![](https://attach.52pojie.cn/forum/202507/30/102527pooa3wmshyaoit8p.png)

5.png (49.12 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc5MzIyN3w5Y2Y4ZWE1NXwxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzIyN3w5Y2Y4ZWE1NXwxNzkzNTcwNzY5fDE2ODA4NnwyMDQ5NzEw&nothumb=yes)

去除更新

2025-7-30 10:25 上传

按 `Ctrl+S` 保存。`PrivateAccess_win.exe` 可以正常运行。再运行 `upx PrivateAccess_win.exe`，重新加壳，程序仍然正常。

## 5. 打包

把 `PrivateAccess_win.exe` 重命名为 `PrivateAccess_win.bin`。打开 `Resource Hacker`，拖入 `privateaccess-win.exe`，展开 `RCData`，选择第一项，右击，Replace，Select，选择 `PrivateAccess_win.bin`，按 `Ctrl+S` 保存文件，得到新的 `privateaccess-win.exe`，可以正常使用，但数字签名不再有效。可以删掉无效的签名证书防止部分杀软报毒。

打包后文件大小与原版完全相同。
