
# [.NET逆向] [原创]少前2逆向经验分享

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1874512](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1874512)
- 抓取时间：2026-11-01T22:39:41

## 免责声明

本篇文章仅作个人逆向分析经验记录与学习之用，所获得的一切资产恕不作任何分享。实践本文章记录的任何操作所产生的一切后果均与本人无关。侵删。

本文的撰写离不开 [52pojie 用户主页](https://www.52pojie.cn/home.php?mod=space&uid=1719928) 前辈开路，也是前辈的帖子带我认识了 Process Monitor 这一神器。前辈帖子指路：[bbs.kanxue.com/thread-279118.htm](https://bbs.kanxue.com/thread-279118.htm) 和 [www.52pojie.cn/thread-1840843-1-1.html](https://www.52pojie.cn/thread-1840843-1-1.html)。

## 1. `global-metadata.dat` 及 `AssetBundle`

跟着前辈的文章一步步走就行了，具体怎么操作就请诸位大佬各显神通了，在此略过。

## 2. 植入 UnityExplorer

简单介绍一下：[UnityExplorer](https://github.com/sinai-dev/UnityExplorer) 是一个十分方便且开源的 Unity 游戏内调试辅助工具，提供了 Inspector、Object Explorer、Console 和 Hook 等诸多强大功能。（可惜的是，该项目不知出于何种原因已经 archived 不再更新。）

- 在上一步中得到解密的 `global-metadata.dat` 后，先将 BepinEx 或者 MelonLoader 框架安装到游戏上。这里根据我的经验，il2cpp 最好选择 MelonLoader 0.5.7 版。

MelonLoader 官方安装教程：[github.com/LavaGang/MelonLoader?tab=readme-ov-file#how-to-use-the-installer](https://github.com/LavaGang/MelonLoader?tab=readme-ov-file#how-to-use-the-installer)

![Snipaste_2023-12-28_10-08-18.png](https://attach.52pojie.cn/forum/202312/29/112939chzmlmvpfh1pmtmk.png)  
Snipaste_2023-12-28_10-08-18.png（99.87 KB，下载次数：1） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk4OXxjOGUyYjg5NHwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

安装 MelonLoader 0.5.7  
2023-12-29 11:29 上传

（这里我已经安装过了，所以显示为 RE-INSTALL，正常安装应该只有一个 INSTALL 按钮。）

- 安装 MelonLoader 后，依照 UnityExplorer 的 Readme 安装 UnityExplorer。

[UnityExplorer MelonLoader 安装说明](https://github.com/sinai-dev/UnityExplorer?tab=readme-ov-file#melonloader)

![Snipaste_2023-12-28_10-08-26.png](https://attach.52pojie.cn/forum/202312/29/113003bs10j2fvyatfow3b.png)  
Snipaste_2023-12-28_10-08-26.png（189.45 KB，下载次数：0） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk5MHwwMzhjZDBmZnwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

选择 UnityExplorer 版本  
2023-12-29 11:30 上传

（其实就是将解压出来的 `Mods` 文件夹和 `UserLibs` 文件夹复制到游戏对应目录就行了。）

## 3. 替换 `global-metadata.dat`

安装 MelonLoader 启动后，会在终端发现 Cpp2IL 报错，导致游戏无法正常启动。这是因为原版加密的 `global-metadata.dat` 依然在 `il2cpp_data\Metadata` 文件夹中，需要凭借单身多年的手速将其替换。

替换之前记得将 dump 出来的 `global-metadata.dat` 的第 4-8 字节的版本号 patch 正确。经过一遍遍在 il2cppDumper 中试错后，确认为 `0x18`。

偷梁换柱具体步骤如下：

- 在 `Game` 文件夹中直接启动游戏，不通过启动器启动。
- 立即切换到终端窗口，当 `Loading Plugins from ...` 字样出现时，立即选中终端内任意范围冻结进程。此时游戏窗口应刚好出现并冻结于如图所示白屏状态。

![Snipaste_2023-12-28_10-10-12.png](https://attach.52pojie.cn/forum/202312/29/113024c919iocw9qwf9bos.png)  
Snipaste_2023-12-28_10-10-12.png（152.73 KB，下载次数：2） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk5MXw0MmY2MTg4NXwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

通过终端选中冻结进程  
2023-12-29 11:30 上传

- 复制并备份好原版 `global-metadata.dat`。（这一步和接下来的几步理应不会出现“文件被占用”“无法启动”、游戏崩溃之类的任何错误；如果遇到了，说明上一步冻结时机过晚、过早。）
- 将 dump 并 patch 好版本（再强调一次，十分甚至九分重要）的 `global-metadata.dat` 替换过去。
- 切换至终端窗口，按 `Esc` 取消选中，解冻进程。
- 退出游戏，将第 3 步备份好的原版 `global-metadata.dat` 替换回来。之后在遇到游戏更新时，重新走一遍以上步骤即可。

## 4. 解密其它未封装进 `AssetBundle` 的资源

在成功安装 UnityExplorer 之后，我们应当已经拥有了 il2CppDumper 提供的 `dump.cs` 文件，以及 UnityExplorer 提供的强大 hook 功能。至此我们终于可以进入正题，解密其它资源。

### 4.1 解密 `.acb`、`.awb` 音频

根据以往解 galgame 的经验，特殊格式必有特殊工具。通过万能的搜索引擎，我最终找到了 [vgmstream](https://github.com/vgmstream/vgmstream) 这个强大工具。

通过阅读 [vgmstream USAGE.md#decryption-keys](https://github.com/vgmstream/vgmstream/blob/master/doc/USAGE.md#decryption-keys) 这节可以发现，解码 `.acb` 和 `.awb` 需要 `.hcakey`。经过测试发现确实如此，没有 `.hcakey` 解码出来的 `.wav` 文件就是一坨答辩。

![Snipaste_2023-12-28_10-10-25.png](https://attach.52pojie.cn/forum/202312/29/113049bu9z8zyp9yhujhj4.png)  
Snipaste_2023-12-28_10-10-25.png（119.47 KB，下载次数：1） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk5Mnw0OGNjYjA3NnwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

解密 `.awb`/`.acb` 可能需要 `.hcakey`  
2023-12-29 11:30 上传

再次借助万能搜索引擎の抛瓦，我找到了 [blog.mottomo.moe](https://blog.mottomo.moe/) 大佬的这篇分析 `.hca` 的顶级文章。

[New-HCA-Encryption](https://blog.mottomo.moe/categories/Tech/RE/zh/2018-10-12-New-HCA-Encryption/)

这里我就直奔主题，跳到 [初步分析密钥设置流程](https://blog.mottomo.moe/categories/Tech/RE/zh/2018-10-12-New-HCA-Encryption/#4-1-%E5%88%9D%E6%AD%A5%E5%88%86%E6%9E%90%E5%AF%86%E9%92%A5%E8%AE%BE%E7%BD%AE%E6%B5%81%E7%A8%8B) 这一节，发现了 `criWareUnity_SetDecryptionKey()` 这一关键函数。然而在 IDA 或者 il2CppDumper 的 `dump.cs` 中都搜索不到 `SetDecryptionKey` 这一函数。

![Snipaste_2023-12-28_10-10-35.png](https://attach.52pojie.cn/forum/202312/29/113118pijg9jmniqxhzj3j.png)  
Snipaste_2023-12-28_10-10-35.png（33.1 KB，下载次数：0） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk5M3wxYTAwZTNlMnwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

搜索 `SetDecryptionKey` 无结果  
2023-12-29 11:31 上传

因此我退而求其次，搜索 `Decrypt` 关键词。这次在 IDA 中发现了一些跟 `CriWare` 有关的目标函数。

![Snipaste_2023-12-28_10-10-46.png](https://attach.52pojie.cn/forum/202312/29/113148f335smcsw2tzyss2.png)  
Snipaste_2023-12-28_10-10-46.png（209.92 KB，下载次数：2） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk5NHxiOWUzMzYzNHwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

IDA 函数搜索 `Decrypt`  
2023-12-29 11:31 上传

其中包括两个跟 `.acb` 有关的目标函数：

- `CriWare.CriAtomExAcb::Decrypt()`
- `CriWare.CriAtomPlugin::DecryptAcb()`

重启游戏，在点击开始屏幕进入游戏之前的界面，尝试用 UnityExplorer hook 住这两个函数。

![Snipaste_2023-12-29_11-34-45.png](https://attach.52pojie.cn/forum/202312/29/113457dmga4lq6fl6lczan.png)  
Snipaste_2023-12-29_11-34-45.png（70.01 KB，下载次数：1） [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2Njk5NXwyODhlMjk4MHwxNzkzNTcyNzc5fDE2ODA4NnwxODc0NTEy&nothumb=yes)

UnityExplorer hook  
2023-12-29 11:34 上传

点击开始屏幕，切换到终端查看打印输出，发现成功打印出 `key`，调用的函数是 `CriWare.CriAtomExAcb::Decrypt()`。
