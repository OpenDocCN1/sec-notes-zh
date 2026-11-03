
```
cd scripts
python3 ndk_builder.py
.\adb_execute_arm64.bat
```

接下来打开我们的 `ida pro` 然后选择 `go`

进去后选择 `Debugger->Attach->Remote ARM Linux/Android debugger`

![](https://attach.52pojie.cn/forum/202304/26/073840cxt44441k14v1g5v.png)

05.png (14.51 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYxMDQ2N3w5M2JjZWNiYnwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2N3w5M2JjZWNiYnwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

2023-4-26 07:38 上传

我们找到我们得 `hello2` 程序。然后选择 `OK`

![](https://attach.52pojie.cn/forum/202304/26/073842easssya6fmytsp3a.png)

06.png (29.67 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMDQ2OHw1ZjU1ZDdhYnwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2OHw1ZjU1ZDdhYnwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

2023-4-26 07:38 上传

进去后注意右下角,双击我们的模块

![](https://attach.52pojie.cn/forum/202304/26/073844jgjvvgags1aqvaqp.png)

07.png (91.22 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxMDQ2OXwyNTg5NDgyY3wxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ2OXwyNTg5NDgyY3wxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

2023-4-26 07:38 上传

进去后注意一点,点击 `main` 方法,如果你是流程图 按 `空格` 切换下,然后再 `hello world` 字符串前面按 `F2` 下个断点

![](https://attach.52pojie.cn/forum/202304/26/073846bg5i3qrhx0gq10xh.png)

08.png (93.66 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMDQ3MHwwMjM5Nzc1YXwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ3MHwwMjM5Nzc1YXwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

2023-4-26 07:38 上传

接下来按 `F9` 或者点击上面的 绿三角 `△` 运行程序。

运行后,在 `vscode` 终端 您启动 `.\adb_execute_arm64.bat` 的地方 按任意键 运行下去

![](https://attach.52pojie.cn/forum/202304/26/073848dssny329sydidyfs.png)

09.png (13.33 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYxMDQ3MXw1ZjQwNWI3NHwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDQ3MXw1ZjQwNWI3NHwxNzkzNTY5NDk2fDE2ODA4NnwxNzc4Njkz&nothumb=yes)

2023-4-26 07:38 上传

接下来您应该就能断下来了。 您可以尝试看下右边的寄存器和状态标志位

然后按 `F8` 步过 观察变化

后面的章节会分析下 `arm64` 的指令 以及 寄存器
