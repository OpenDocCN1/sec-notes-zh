
# [分享] 新手教程-改头换面之修改版权和资源

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1989235](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1989235)
- 抓取时间：`2026-11-01T22:18:33`
- 讲师：Kido

## 总结

- 利用资源编辑器修改版权（上文介绍）：[上文教程](https://www.52pojie.cn/thread-1988677-1-1.html)
- 利用 API 修改版权（本文介绍）
- 利用 16 进制编辑器修改（上上文介绍）：[上上文教程](https://www.52pojie.cn/thread-1987049-1-1.html)

## 实战案例

首先尝试使用资源编辑器修改，找不到，改用 `OD`。将程序拖入 `OD` 并运行，使用中文搜索引擎搜索字符串，找到小广告对应内容 `0040282F`。

![image-20241207184624190.png](https://attach.52pojie.cn/forum/202412/07/191415fd1ve0gqi8vs1abv.png)

[image-20241207184624190.png (650.11 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjE1N3wyMDI1MDZkMnwxNzkzNTcxNTEwfDE2ODA4NnwxOTg5MjM1&nothumb=yes)

2024-12-7 19:14 上传

进入后发现使用了 API `SetDlgItemTextW/A`（给 `DialogBox` 类型控件设置文本）。

![image-20241207184838304.png](https://attach.52pojie.cn/forum/202412/07/191418sfzi4ffifa47arix.png)

[image-20241207184838304.png (435.36 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjE1OHw3NGRlN2JiZnwxNzkzNTcxNTEwfDE2ODA4NnwxOTg5MjM1&nothumb=yes)

2024-12-7 19:14 上传

复制 `push` 文本的地址 `00551A18`。

![image-20241207185018119.png](https://attach.52pojie.cn/forum/202412/07/192247th843a8b5hhehlbh.png)

[image-20241207185018119.png (765.22 KB, 下载次数: 3)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjE2Mnw0ZWYwYjUwM3wxNzkzNTcxNTEwfDE2ODA4NnwxOTg5MjM1&nothumb=yes)

2024-12-7 19:22 上传

在数据窗口跟到 `00551A18`，选择 Unicode 显示。

![image-20241207185423440.png](https://attach.52pojie.cn/forum/202412/07/192248pni3rz55zro3rqjg.png)

[image-20241207185423440.png (334.71 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjE2M3wzYzBmMjBiM3wxNzkzNTcxNTEwfDE2ODA4NnwxOTg5MjM1&nothumb=yes)

2024-12-7 19:22 上传

![image-20241207185508909.png](https://attach.52pojie.cn/forum/202412/07/192248wboz92924dtwmaw2.png)

[image-20241207185508909.png (454.81 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjE2NHw4ZDUxOTRiNXwxNzkzNTcxNTEwfDE2ODA4NnwxOTg5MjM1&nothumb=yes)

2024-12-7 19:22 上传

选中广告内容，右键-二进制-编辑，修改其中的 Unicode 内容即可。

![image-20241207185559657.png](https://attach.52pojie.cn/forum/202412/07/192249ig3soocc8sbx83b8.png)

[image-20241207185559657.png (513.32 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MjE2NXw4M2IwZTQyNXwxNzkzNTcxNTEwfDE2ODA4NnwxOTg5MjM1&nothumb=yes)

2024-12-7 19:22 上传

当然，你也可以用 16 进制编辑器 `C32Asm` 来编辑。
