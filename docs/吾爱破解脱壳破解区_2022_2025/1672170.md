
# [原创] 植物大战僵尸之植物被咬反增阳光

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1672170
- 抓取时间：2026-11-01T23:11:40

找地址可以看这一篇文章：[//www.52pojie.cn/thread-1671905-1-1.html](https://www.52pojie.cn/thread-1671905-1-1.html)

我们利用思维，可以一个地址实现好多个功能。

我们大概的流程是：种植植物 → 植物被僵尸咬 → 利用阳光地址增加阳光。

## 关键代码

植物找阳光的基址，论坛有很多，这儿就不做详细的介绍了。

同样的地址，申请内存跳过去。

![ass.jpg](https://attach.52pojie.cn/forum/202208/07/094543bzgrzzmrml3malrl.jpg)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0Mzc3MXwzMzA0NTFmNnwxNzkzNTc0Njk4fDE2ODA4NnwxNjcyMTcw&nothumb=yes)

`ass.jpg`（44.45 KB，下载次数：1） 2022-8-7 09:45 上传

![asss.jpg](https://attach.52pojie.cn/forum/202208/07/094619m6gg557rr354ju9j.jpg)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0Mzc3Mnw3Y2E1ZDUwMnwxNzkzNTc0Njk4fDE2ODA4NnwxNjcyMTcw&nothumb=yes)

`asss.jpg`（55.73 KB，下载次数：1） 2022-8-7 09:46 上传

```asm
mov ecx, dword [esi+0x40]
mov dword [esi+0x000000B4], 0x00000032  // 上面两句还原被HOOK的代码

’//我们知道阳光的基地址是：[6a9ec0+768+5560]
mov eax, dword [0x006A9EC0] ' 把基址传递给eax
mov eax, dword [eax+0x00000768] '这儿看作 [6a9ec0+768]
cmp dword [eax+0x00005560], 0x00002706 '判断当前的阳光数 是否 2706  十进制为：9990
jc label1 小于则跳到label1代码执行
jnbe label2  否则跳到label2代码执行
label1:
add dword [eax+0x00005560], 0x19  把19传递给 [eax+5560]  我们读写内存都是  [6a9ec0+768+5560] 之后在写入，这也是一杨
label2:
mov edx, 0x0052FD01  跳回HOOK原代码
jmp edx
```

![458.jpg](https://attach.52pojie.cn/forum/202208/07/095844ukxlmuubm3gpph4b.jpg)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0Mzc3M3w5MjhiMTVhMnwxNzkzNTc0Njk4fDE2ODA4NnwxNjcyMTcw&nothumb=yes)

`458.jpg`（126.71 KB，下载次数：1） 2022-8-7 09:58 上传

## 最终效果

每当植物被咬一口的代码触发，就每次增加 `20` 的阳光值。可以把 `20` 修改得更多，两口的话基本就满了。

大家可以找找僵尸的来实现一些别的实现方法。

[//www.52pojie.cn/thread-1671405-1-1.html](https://www.52pojie.cn/thread-1671405-1-1.html)

源码里这些没写，因为是即兴发挥。

## 源码

看到有人要成品，我还是直接索性直接上源码。成品哪有源码香。

![123.jpg](https://attach.52pojie.cn/forum/202208/07/112329xk902dad4takm5fo.jpg)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0Mzc5M3w3ZDBlNmU5NHwxNzkzNTc0Njk4fDE2ODA4NnwxNjcyMTcw&nothumb=yes)

`123.jpg`（98.01 KB，下载次数：1） 2022-8-7 11:23 上传

就只写了这一个，之前两篇的写法和这个一样。

![zip](https://static.52pojie.cn/static/image/filetype/zip.gif)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0Mzc5N3xiNGFkOTc4ZXwxNzkzNTc0Njk4fDE2ODA4NnwxNjcyMTcw)

（2.03 MB，下载次数：282） 2022-8-7 11:28 上传

点击文件名下载附件

下载积分：吾爱币 -1 CB
