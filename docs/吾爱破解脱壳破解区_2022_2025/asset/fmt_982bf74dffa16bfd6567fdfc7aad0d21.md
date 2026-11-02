
# [CTF] xctf逆向新手区第三题

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1590163>
- 抓取时间：2026-11-01T23:35:45

## 1. 先查看题目要求

![image-20220219102734776.png](https://attach.52pojie.cn/forum/202202/19/110256vy2zzme3end2ed71.png)

附件：[image-20220219102734776.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NjE4NXw5YjBjZTU2N3wxNzkzNTc2MTQyfDE2ODA4NnwxNTkwMTYz&nothumb=yes)（40.51 KB，下载次数：2）· 2022-2-19 11:02 上传

## 2. 直接告诉我们是个被加壳的64位二进制文件，那我们先下载下来查壳脱壳

![image-20220219102856203.png](https://attach.52pojie.cn/forum/202202/19/110357rbrbriry7obzp9ec.png)

附件：[image-20220219102856203.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NjE4Nnw1ZWE1ZDJmN3wxNzkzNTc2MTQyfDE2ODA4NnwxNTkwMTYz&nothumb=yes)（128.61 KB，下载次数：0）· 2022-2-19 11:03 上传

## 3. `upx` 的壳，由于是二进制文件，那我们就用 kali 去脱壳吧

> [tip] 脱壳命令：`upx -d 脱壳文件路径`

![image-20220219105121991.png](https://attach.52pojie.cn/forum/202202/19/110407ukgzxgnpxx3bodjn.png)

附件：[image-20220219105121991.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NjE4N3xmZmE1ZDcyMnwxNzkzNTc2MTQyfDE2ODA4NnwxNTkwMTYz&nothumb=yes)（69.77 KB，下载次数：0）· 2022-2-19 11:04 上传

## 4. 脱壳结束后我们放入 ida 中分析

一丢进 ida 就直接看到了 flag

![image-20220219105450481.png](https://attach.52pojie.cn/forum/202202/19/110416b3u2aup0p4w9azxf.png)

附件：[image-20220219105450481.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NjE4OHw3MDUyM2JlNnwxNzkzNTc2MTQyfDE2ODA4NnwxNTkwMTYz&nothumb=yes)（34.29 KB，下载次数：0）· 2022-2-19 11:04 上传

## 5. 这样多没意思，还是 F5 看看伪代码吧

![image-20220219105607976.png](https://attach.52pojie.cn/forum/202202/19/110429yilys33b75lv77p3.png)

附件：[image-20220219105607976.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjQ5NjE5OXwzNDRhNjA3OXwxNzkzNTc2MTQyfDE2ODA4NnwxNTkwMTYz&nothumb=yes)（23.32 KB，下载次数：0）· 2022-2-19 11:04 上传

代码不难，流程就是让用户输入用户认为的 flag，如果和作者给的 flag 对比相等那么就提示 `Congratulations`，否则提示 `Try again!`

## 总结

综合下来这题只是想让新手们知道有 `upx` 这种壳的存在以及最基本脱壳方法，那什么是 `upx` 壳呢？直接附上介绍链接：[https://www.cnblogs.com/iBinary/p/7764483.html](https://www.cnblogs.com/iBinary/p/7764483.html)
