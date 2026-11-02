
[href="forum.php?mod=attachment&aid=MjUzMDg4MXwyOGNmZTcwNXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4MXwyOGNmZTcwNXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

11、`plugins`文件夹复制完成后，输入`net stop "Tenable Nessus"`、`netstart "Tenable Nessus"`重启服务。

![](https://attach.52pojie.cn/forum/202206/17/170820ljdqjmmne8qhdyvq.png)

图片18.png (13.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUzMDg4MnxiM2Y0MDFjZXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4MnxiM2Y0MDFjZXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

12、刷新浏览器界面，等待插件编译完成。

![](https://attach.52pojie.cn/forum/202206/17/170822ggtubhr6rttci6xt.png)

图片19.png (95.22 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUzMDg4M3w4MWEwNWQ1N3wxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4M3w4MWEwNWQ1N3wxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

13、进入应用后，发现还是不能进行扫描（正常）。

![](https://attach.52pojie.cn/forum/202206/17/170824y0z7ktzkj7jyqsw2.png)

图片20.png (43.03 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUzMDg4NHxmNzRkOTIyZHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4NHxmNzRkOTIyZHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

## 三、破解

1、停止服务`net stop "Tenable Nessus"`

![](https://attach.52pojie.cn/forum/202206/17/170826c668owlqkw6u1zvj.png)

图片21.png (6.31 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUzMDg4NXw0NjAzOWJlZHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4NXw0NjAzOWJlZHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

2、将`plugin_feed_info.inc`和`plugins`拷贝到安装目录`nessus\nessus`路径下进行替换。

![](https://attach.52pojie.cn/forum/202206/17/170828mnrfzghf62x8jhoh.png)

图片22.png (98.82 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUzMDg4NnwyYzcyMTM0MHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4NnwyYzcyMTM0MHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

3、按顺序输入以下 3 个命令（实际路径需根据实际安装目录进行调整）。

```
attrib +s +r +h “E:\software\nessus\nessus\plugins\*.*”

attrib +s +r +h “E:\software\nessus\nessus\plugin_feed_info.inc”

attrib -s -r -h “E:\software\nessus\nessus\plugins\plugin_feed_info.inc”
```

![](https://attach.52pojie.cn/forum/202206/17/170830fzqjj72l3333gqqc.png)

图片23.png (12.19 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUzMDg4N3wzNTRhNDgyNXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4N3wzNTRhNDgyNXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

4、启动服务`net start "Tenable Nessus"`。

![](https://attach.52pojie.cn/forum/202206/17/170832dnwryzwqwwswany4.png)

图片24.png (5.55 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUzMDg4OHw5ZDk2NDNmZnwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4OHw5ZDk2NDNmZnwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

5、打开浏览器`https://127.0.0.1:8834`等待插件编译完成。

![](https://attach.52pojie.cn/forum/202206/17/170835xpuv244vqfu7u24f.png)

图片25.png (95.25 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUzMDg4OXwzMjZlYTVlY3wxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg4OXwzMjZlYTVlY3wxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

6、编译完成后，可以看到已经出现扫描功能。

![](https://attach.52pojie.cn/forum/202206/17/170837aj3g257p4mb22r74.png)

图片26.png (32.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUzMDg5MHw5YzU4ODc1NHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg5MHw5YzU4ODc1NHwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传

7、扫一扫试一下，完全没问题。

![](https://attach.52pojie.cn/forum/202206/17/170839ty05ns0n1ris650r.png)

图片27.png (29.03 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUzMDg5MXw3NWYxMjlmOXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzMDg5MXw3NWYxMjlmOXwxNzkzNTc1MTkxfDE2ODA4NnwxNjUwNjY3&nothumb=yes)

2022-6-17 17:08 上传
