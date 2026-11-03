
# [CTF] 2025腾讯游戏安全技术竞赛初赛-题解(有源码)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2020132
- 抓取时间: 2026-11-01T22:11:21

## 源码

开源地址: [//github.com/DejavuSecure/2025-tencent-game-security-race-pre" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/DejavuSecure/2025-tencent-game-security-race-pre:rel="nofollow noopener" href="https)

## 第一部分：阅读题目要求

根据题目要求 :

选择 Windows 11 22H2 系统，关闭 Windows Defender, VBS, HyperV 后，以管理员权限执行解题程序。首先打开`ACEFirstRound.exe`，提示请输入flag。

![](https://attach.52pojie.cn/forum/202504/01/171845id5x5544gl40kzph.png)

0.png (45.59 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc2NzkzM3wwMGRjNWNkM3wxNzkzNTcxMDc4fDE2ODA4NnwyMDIwMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NzkzM3wwMGRjNWNkM3wxNzkzNTcxMDc4fDE2ODA4NnwyMDIwMTMy&nothumb=yes)

2025-4-1 17:18 上传

解题开始:

## 第二部分：Ring3 程序调试

### 主模块地址获取

在进程启动的初期输入: `lm` 获得主模块的地址。

![](https://attach.52pojie.cn/forum/202504/01/171848kffly6f6y0oqkmfe.png)

1.png (24.02 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NzkzNHxkYWNhNzJiNnwxNzkzNTcxMDc4fDE2ODA4NnwyMDIwMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NzkzNHxkYWNhNzJiNnwxNzkzNTcxMDc4fDE2ODA4NnwyMDIwMTMy&nothumb=yes)

2025-4-1 17:18 上传

共需绕过两处检测，第一处为 R3 的基础反调试函数`sub_1400010D0`，第二处为驱动加载后对 R3 的修补检测。具体表现为

- `windbg` 调试程序无法正常执行
- 绕过 R3 调试检测后在驱动加载时对程序下断， 程序会直接被结束。

这里的做法是直接在 `sub_140001370` 处阻止驱动加载。

![](https://attach.52pojie.cn/forum/202504/01/171851josibs6i6bomdnzb.png)

2.png (29.69 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NzkzNXwzMGY3Y2Q2OHwxNzkzNTcxMDc4fDE2ODA4NnwyMDIwMTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NzkzNXwzMGY3Y2Q2OHwxNzkzNTcxMDc4fDE2ODA4NnwyMDIwMTMy&nothumb=yes)

2025-4-1 17:18 上传
