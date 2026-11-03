
# [原创] 记录一下 160 个 crackme 练手--5

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1626969
- 抓取时间：2026-11-01T23:27:52

第 5 个我也卡了好久，卡好久也没思路，就只会个爆破了，硬磨了挺长时间，然后放弃去看大佬的帖子去了。看完之后只想说：“太复杂，太变态了！”

## 查壳

`PIED` 查壳发现有壳，是 `upx` 的壳。

![Snipaste_2022-04-24_14-59-04.png](https://attach.52pojie.cn/forum/202204/24/145836q6ct27tx46vp6x2z.png)
（17.23 KB，下载次数：0）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg0NXw0YjJjYzQxYXwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 14:58 上传）

我们直接用工具[链接](https://www.52pojie.cn/forum-5-1.html)就可以。

![Snipaste_2022-04-24_15-01-48.png](https://attach.52pojie.cn/forum/202204/24/150121m115l163x2282lz3.png)
（50.09 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg0Nnw3Yjc0MDEwZHwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:01 上传）

## 运行程序

然后我们运行程序，看看要解决什么问题。

发现它有个提示，还有个框框，框框里面有 4 张图片在循环移动。点击注册没有任何反应。

![Snipaste_2022-04-24_15-03-19.png](https://attach.52pojie.cn/forum/202204/24/150255mhvnhdyn9oh5yj47.png)
（49.42 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg0N3xkM2IzNDYxZHwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:02 上传）

## 搜索字符串

我们去搜字符串吧。

![Snipaste_2022-04-24_15-07-15.png](https://attach.52pojie.cn/forum/202204/24/150646drp4mmn9sv6m8lse.png)
（28.5 KB，下载次数：0）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg1M3xiYTBhODQyZXwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:06 上传）

发现有 4 个“注册尚未成功，同志仍需努力！8）”。这个字符会在每次单击 4 个图片时出现，4 个图片应该对应了 4 个字符。

![Snipaste_2022-04-24_15-09-35.png](https://attach.52pojie.cn/forum/202204/24/150912niu14u16r3do7tpz.png)
（18.36 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg1NXxmYjdkOGE4M3wxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:09 上传）

总之，这些东西就不考虑了！我搞了那么长时间，真的越看越懵逼，还是来爆破吧！

## 爆破修改

双击跟到这个位置。

![Snipaste_2022-04-24_15-21-57.png](https://attach.52pojie.cn/forum/202204/24/152123zz2au22vufgvis9g.png)
（41.77 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg2N3xiZjk4MDkxNnwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:21 上传）

跟过去发现有 5 个跳转都有可以跳过成功。

![Snipaste_2022-04-24_15-22-57.png](https://attach.52pojie.cn/forum/202204/24/152245v55ue8doeuu5zce8.png)
（30.49 KB，下载次数：0）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg3MHw4M2YyNmVmMnwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:22 上传）

把他们通通 `nop` 掉！

![Snipaste_2022-04-24_15-25-44.png](https://attach.52pojie.cn/forum/202204/24/152526i2si4sqxxi5734vs.png)
（21.08 KB，下载次数：2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg3M3w1YzkwNmMzOHwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:25 上传）

![Snipaste_2022-04-24_15-26-34.png](https://attach.52pojie.cn/forum/202204/24/152558iaafsvlgbsezbgd6.png)
（19.39 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg3NHwxNTZmMmMzZHwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:25 上传）

然后保存所有修改，打开修改后的文件，点注册等个几秒。

![Snipaste_2022-04-24_15-28-00.png](https://attach.52pojie.cn/forum/202204/24/152801kzivubbq6zd11bik.png)
（33.99 KB，下载次数：2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNjg3OHxjYmRmYjQxZnwxNzkzNTc1NjcwfDE2ODA4NnwxNjI2OTY5&nothumb=yes)（2022-4-24 15:28 上传）

就成功了。

## 参考

最后给你们贴个帖子，让你们看看这太变态了！

[`CrackMe005` 全破详解（逆向分析部分已补充完整）](https://www.52pojie.cn/thread-855172-1-1.html)

（出处：[吾爱破解论坛](https://www.52pojie.cn)）

这才第 5 个啊，唉。但如果是 CM 的话，只爆破也是可以的吧……

![cry.gif](https://static.52pojie.cn/static/image/smiley/default/cry.gif)
