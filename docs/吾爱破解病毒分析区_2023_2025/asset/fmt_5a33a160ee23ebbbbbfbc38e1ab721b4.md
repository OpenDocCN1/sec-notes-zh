
[href="forum.php?mod=attachment&aid=MjcxNzEzOXwzYzNkYjc3OXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzEzOXwzYzNkYjc3OXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:38 上传

![](https://attach.52pojie.cn/forum/202408/13/163835k85jt4l49jo29csa.png)

Image-17.png (50.72 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0MHw5NWRiNWZhOHwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0MHw5NWRiNWZhOHwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:38 上传

## 文件复制展示

然后拼接字符串 `WScript.exe //B "C:\Users\Administrator\AppData\Roaming\Monetary_Funding_Sheet_2024.js"` 写入到注册表 `"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"` 中，进行第二个持久化操作。其中 `/B` 参数将启用“批处理模式”，并禁用执行期间可能发生的任何潜在警告或警报：

![](https://attach.52pojie.cn/forum/202408/13/163839gy0dpg0orpptrpdu.png)

Image-18.png (133.21 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0MXxkNjU2YTE0MXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0MXxkNjU2YTE0MXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:38 上传

### 写入开机自启动注册表

最后继续用命令 `WScript.exe //B "C:\Users\Administrator\AppData\Roaming\Monetary_Funding_Sheet_2024.js"` 执行复制的文件，退出当前进程：

![](https://attach.52pojie.cn/forum/202408/13/163844uv0hl7etb3cg6zm0.png)

Image-19.png (34.42 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0Mnw2OGViZjJiNXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0Mnw2OGViZjJiNXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:38 上传

### 启动子进程

`"C:\Users\Administrator\AppData\Roaming\Monetary_Funding_Sheet_2024.js"` 被执行后才会进入最后的 `while` 循环，该循环负责与 C2 IP `45.88.91.57` 的通信操作：

![](https://attach.52pojie.cn/forum/202408/13/163850t2iz0yw5w92zpc2l.png)

Image-20.png (24.8 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0M3xjMTIwMGZlM3wxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0M3xjMTIwMGZlM3wxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:38 上传

### 通信循环

其会先解密出请求路径 `"/is-ready"` 拼接到前面获取的 C2 中，接着把收集到的本地相关信息作为 `user-agent` 字段数据回传给 C2：

![](https://attach.52pojie.cn/forum/202408/13/163856u12znhuw0inwjhgc.png)

Image-21.png (30.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0NHwwZTkzOGJkNXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0NHwwZTkzOGJkNXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:38 上传

![](https://attach.52pojie.cn/forum/202408/13/163901zkhpsxu2kfkwwikf.png)

Image-22.png (46.36 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0NXxhNjcxYjNlMnwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0NXxhNjcxYjNlMnwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### 构造数据包

![](https://attach.52pojie.cn/forum/202408/13/163905jfl26lssjd7244dj.png)

Image-23.png (24.87 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0Nnw2NzQ2YjdiYXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0Nnw2NzQ2YjdiYXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### 流量包截图

`user-agent` 字段中包含了 `Monetary_Funding_Sheet_2024.js` 在运行过程中收集的相关信息，整理格式如下：

![](https://attach.52pojie.cn/forum/202408/13/163910scsacyqdxqclmblt.png)

Image-24.png (17.2 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0N3xlMWYzY2NlYXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0N3xlMWYzY2NlYXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### 格式整理

随后根据 C2 回传的指令，在 `switch` 中进行匹配以执行指定的操作，从而实现远程控制。

`"Monetary_Funding_Sheet_2024.js"` 涉及的操作指令包括 `execute`、`update`、`uninstall`、`send`、`site-send`、`recv`、`enum-driver`、`enum-faf`、`enum-process`、`cmd-shell`、`delete`、`exit-process`、`sleep`：

![](https://attach.52pojie.cn/forum/202408/13/163915xx4mxo05l5g4brqm.png)

Image-25.png (21.64 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0OHxlYjE0MGRjZnwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0OHxlYjE0MGRjZnwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### 操作指令匹配

下图是国外研究人员在 `Houdini is Back Delivered Through a JavaScript Dropper` 捕获到的不带混淆的 Houdini RAT，这是 WSHRAT 的前身，可以看到控制命令基本相同：

![](https://attach.52pojie.cn/forum/202408/13/163920qsdqefkmlvsnmsnm.png)

Image-26.png (320.39 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE0OXwxZjc4NGRmMHwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE0OXwxZjc4NGRmMHwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### Houdini RAT 功能代码

此外，在分析 `"Monetary_Funding_Sheet_2024.js"` 时根据其 C2 IP `45.88.91.57` 关联到另一篇出自 JAMESWT 写的关于 Twitter 威胁情报，该研究人员列出了其它同源的样本。目前，火绒对这些关联样本均支持查杀：

![](https://attach.52pojie.cn/forum/202408/13/163924ob0vc8cg2z22wwuo.png)

Image-27.png (220.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE1MHxjYzRjNzJlN3wxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE1MHxjYzRjNzJlN3wxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### Twitter 情报

## 二、附录

### C&C

![](https://attach.52pojie.cn/forum/202408/13/163932yzxqvxl969eq1o64.png)

Image-29.png (9.74 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE1MnxkOTQxYTE3ZXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE1MnxkOTQxYTE3ZXwxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传

### HASH

![](https://attach.52pojie.cn/forum/202408/13/163928mh0lt2ofs46xseit.png)

Image-28.png (13.43 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcxNzE1MXxiYmE2MmYwY3wxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcxNzE1MXxiYmE2MmYwY3wxNzkzNTc2ODM1fDE2ODA4NnwxOTU0MDcy&nothumb=yes)

2024-8-13 16:39 上传
