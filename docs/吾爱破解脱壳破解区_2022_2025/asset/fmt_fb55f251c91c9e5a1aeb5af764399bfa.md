
# [CTF] 吾爱破解2025红包题WriteUP（除高级）

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=2002919>
- 抓取时间：2026-11-01T22:15:36

## 目录
- Windows部分：
  - [解题领红包之二 {Windows 初级题}](<https://www.52pojie.cn/thread-2002919-1-1.html#52304896_%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E4%BA%8C-{windows-%E5%88%9D%E7%BA%A7%E9%A2%98}>)
- 安卓部分：
  - [解题领红包之三 {Android初级题}](<https://www.52pojie.cn/thread-2002919-1-1.html#52304896_%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E4%B8%89-{android%E5%88%9D%E7%BA%A7%E9%A2%98}>)
  - [解题领红包之四 {Android中级题}](<https://www.52pojie.cn/thread-2002919-1-1.html#52304896_%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E5%9B%9B-{android%E4%B8%AD%E7%BA%A7%E9%A2%98}>)
- Web部分：
  - [【春节】解题领红包之九 {番外篇 一}](<https://www.52pojie.cn/thread-2002919-1-1.html#52304896_%E3%80%90%E6%98%A5%E8%8A%82%E3%80%91%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E4%B9%9D-{%E7%95%AA%E5%A4%96%E7%AF%87-%E4%B8%80}>)
  - [【春节】解题领红包之十 {番外篇 二}](<https://www.52pojie.cn/thread-2002919-1-1.html#52304896_%E3%80%90%E6%98%A5%E8%8A%82%E3%80%91%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E5%8D%81-{%E7%95%AA%E5%A4%96%E7%AF%87-%E4%BA%8C}>)
  - [【春节】解题领红包之十一 {番外篇 三}](<https://www.52pojie.cn/thread-2002919-1-1.html#52304896_%E3%80%90%E6%98%A5%E8%8A%82%E3%80%91%E8%A7%A3%E9%A2%98%E9%A2%86%E7%BA%A2%E5%8C%85%E4%B9%8B%E5%8D%81%E4%B8%80-{%E7%95%AA%E5%A4%96%E7%AF%87-%E4%B8%89}>)

## Windows与安卓CM部分

### 前言
第二年参加红包题目，初级和中级题目还是挺得心应手（大费周折）的（高级题还是不会做，嗯。）

### 解题领红包之二 {Windows 初级题}
新手题：送分题完成啦来试试新手题吧，点击下方“立即申请”任务，即可获得本题Windows CrackMe题目下载地址，通过分析CrackMe获得本题正确口令的解题方法。

#### 查壳
嗯，很好，还是没有壳。

![](https://attach.52pojie.cn/forum/202501/29/193117kawrbbbwqlp5auzr.png)

屏幕截图 2025-01-29 094052.png (15.7 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjMwM3wwYzJlYWZlZnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-29 19:31 上传

#### IDA 静态分析
略过，代码太长太复杂了，还是直接动态调试吧......

#### `x64dbg` 调试
在字符串区域找到了作者留下的提示：

"Hint: Debug the LCG sequence!"

![](https://attach.52pojie.cn/forum/202501/29/193224ez4vvs7zr4ls569v.png)

屏幕截图 2025-01-29 101139.png (20.93 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjMwNHw3NDA2Y2I3Y3wxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-29 19:32 上传

继续分析，找到关键跳1，下断点：

![](https://attach.52pojie.cn/forum/202501/29/193356umu1ulej7y1seeut.png)

屏幕截图 2025-01-29 102538.png (69.97 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjMwNXwxNTgyNmI4ZHwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-29 19:33 上传

这里的意思是比较 `esi` 和 `edi` 两个寄存器中的数值是否一致，不是的话就输出 "Error, please try again." 退出

`esi` 是你输入字符串的长度，我这里输入了 "111"，所以为 3，它这边要求的是 `1B[H]`，也就是 `27[D]`，即密码长度为 27 位。

此时提示变为了 "Wrong password, please try again." 说明前面的分析没问题。

![](https://attach.52pojie.cn/forum/202501/29/102900zquizx9icitqqiii.png)

image.png (19.23 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI2NHwwMzJmNDRiYnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-29 10:29 上传

使用 `F8` 跟进，即可找到位于 `ECX` 中的 `Flag`

![](https://attach.52pojie.cn/forum/202501/29/104626pjeoete3tvcz13dd.png)

image.png (69.77 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI2OXwyZmJjMTc5YnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-29 10:46 上传

![](https://attach.52pojie.cn/forum/202501/29/104753f2yg7c3sple57s7z.png)

image.png (10.92 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjI3MHxmM2VhOTk3YnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-29 10:47 上传

> `Flag`: `fl@g{52pOj1E_2025#Fighting}`

### 解题领红包之三 {Android初级题}
题目简介：程序员小王和测试员老李是"相爱相杀"的好基友，最近小王发现老李破解CM的速度突飞猛进，他百思不得其解。经过三天三夜的暗中观察，终于在某次午休时撞见老李把手机折成S形——上折看反编译代码，下折调动态注入参数，中间还夹着个实时日志监控屏。"好家伙！原来你用的遥..."话音未落，老李邪魅一笑把手机折成三截："年轻人，听说过三折叠吗？"

（PS:试试左滑屏幕）

好家伙，这下真·遥遥领先了......

![](https://attach.52pojie.cn/forum/202501/31/161049blwroiliawavz4of.png)

image.png (6.93 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjQ1MXxlMzliYjk4Y3wxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-31 16:10 上传

#### 分析
程序本身看不出来有什么，直接拖进 `jadx` 分析一下：

代码很清楚，创建 `View` 组件，获取 `adapter`，然后根据 `position` 创建分片。

![](https://attach.52pojie.cn/forum/202501/31/161420z74znxlzkuoblxb2.png)

image.png (50.25 KB, 下载次数: 0)  
[下载附件](<https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1MjQ1MnxlY2Q0NmI3ZnwxNzkzNTcxMzMzfDE2ODA4NnwyMDAyOTE5&nothumb=yes>)  
2025-1-31 16:14 上传

`FoldFragment1` 没有什么东西，就创建了一个媒体播放器然后播放，重点看 `FoldFragment2`
