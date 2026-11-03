
# [原创] Seer v4.0.1，PinStack v1.10.0 注册机制分析记录

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2034373
- 抓取时间: 2026-11-01T22:09:03

## 前排提醒

本文不包含任何成品或注册机，但内容应足够分析`Seer v4.0.1`注册机制（后续 v4.X 也应适用）。
本文对部分细节流程做出省略（尤其是加解密部分），如有需求可自行分析。
宝藏小众软件，分析过程仅供学习，请勿广泛传播破解成品，请多多支持正版，谢谢。
若无法负担，可通过站内信或留言等方式留下你的机器码，获取方式：`cmd`运行如下命令（如安装路径不一致请自行修改，因楼主个人原因可能巨久才会回复一次，如有其他需求可考虑直接搜寻成品使用）

Seer:
```
"C:\Program Files (x86)\Seer\seerhelper.exe" 0 100 | more
```

PinStack:
```
"C:\Program Files (x86)\PinStack\PinStackHelper.exe" 0 100 | more
```

## 正文

`Seer`是一款强大的快速窗口预览工具，具备高度的定制化和出色的运行速度，可让你在 Windows 上获得类似于 macOS 的快速查看文件的功能。支持100多种文件格式，包括视频、音频、图像、文本、Markdown 和 PDF 等，无需打开文件即可高效预览。

### 软件官网
`Seer`: https://1218.io/
`PinStack`: https://1218.io/pinstack.html

### 购买链接
数码荔枝 `Seer`: https://lizhi.shop/products/seer
数码荔枝 `PinStack`: https://lizhi.shop/products/pinstack

下载安装后，在 `C:\Program Files (x86)\Seer` 文件夹下拿到主程序 `Seer.exe`，提示未注册版本只能试用7天

![](https://attach.52pojie.cn/forum/202505/26/232807nnjo1bpjdm2xynol.png)

从官网得知此处激活码为联网激活，且一台激活码能激活10台设备，因此此处输入的码应该不为本地激活的最终产物。

将 `Seer.exe` 拖入 `Binary Ninja`，搜索关键词 `Seer will stop running`：

![](https://attach.52pojie.cn/forum/202505/26/233025iae5xpzhrxxeprpg.png)

根据上面的if判断翻译，给光标所指的变量的来源的函数标记名字，其作用应为获取试用类型。

进入该函数，可见其使用Qt自带的`QSettings`对一些配置进行读写。

![](https://attach.52pojie.cn/forum/202505/26/233351k5zm1yccm4wtwkcl.png)

![](https://attach.52pojie.cn/forum/202505/26/233223ntokss4p05ooqowy.png)

Qt的`QSettings`要么在INI文件中，要么在注册表中，要么在某二进制中，经查找注册表确定其存储路径：

`HKEY_CURRENT_USER\Software\Corey\Seer`

![](https://attach.52pojie.cn/forum/202505/26/233650fcclzxhi3mhaem6i.png)

注意到此处有 `reg_id` 和 `reg_key` 两个疑似与注册相关的项，在二进制中搜索字符串，可得其中一个读取+验证处，其使用了与上一步一样的设置读取函数：

![](https://attach.52pojie.cn/forum/202505/26/233824mz313jj530hrrrp3.png)

分析代码，其将 `reg_id` 读取并解密，若其中含有 `","` 则将 `reg_key` 返回。根据论坛其他文章（有关`Seer v1.x`，`Seer v3.x`），猜测 `reg_key` 为旧版激活码，在之后的分析中更关注 `reg_id` 这个涉及加解密操作的值。

查找上述函数的交叉引用，只有一处，进入可得：

![](https://attach.52pojie.cn/forum/202505/26/234137qbvrrnubnr65mb4b.png)

其在验证成功后调用了`seerhelper.exe`，传入参数0与100，使用`cmd`手动调用，可见程序返回了当前硬件的一些信息：

![](https://attach.52pojie.cn/forum/202505/26/234353rmccvbdqbm9vtjmq.png)

读取这些信息后，`Seer.exe`对信息做了一次Hash并存入 `QStringList` 中：

![](https://attach.52pojie.cn/forum/202505/27/001710td3lioaddd3dck34.png)

继续分析，程序将从 `reg_id` 解密得到的文本经过分割后，依次与返回的硬件信息Hash进行比对，在多个硬件信息Hash中，若有三个比对成功，则注册成功。

![](https://attach.52pojie.cn/forum/202505/27/001539gss5qkpprx7dh5mz.png)

因此可编写注册机，主要函数如下：

![](https://attach.52pojie.cn/forum/202505/27/001832maa5qwwyrrsxqa2s.png)

在本电脑上运行可得命令行如下：

![](https://attach.52pojie.cn/forum/202505/27/002748q22v91267g2vwhzh.png)

运行后重启 `Seer`，可见注册按钮已被移动到 `About`，打开后可见已注册成功

![](https://attach.52pojie.cn/forum/202505/27/002656no8j1jj27pqjj398.png)

![](https://attach.52pojie.cn/forum/202505/27/002909bs0vtjst348133tj.png)

![](https://attach.52pojie.cn/forum/202505/27/004233o61e5utjoque6ko1.png)
