
# [Web逆向] 在大佬指明的方向上再迈一步——雨课堂刷课

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1578071](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1578071)
- 抓取时间：2026-11-01T23:39:50

单位要求用雨课堂学习，在帮领导手动看课的时候发现已经有人学完并且满分，于是尝试电动看课。看了 quals 的帖子（[链接](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1512683&highlight=%D3%EA%BF%CE%CC%C3)），尝试着做了一下，成功了，跟大家分享一下。

## 刷视频

![刷课](https://attach.52pojie.cn/forum/202201/19/204850jjgocric2q2gbtba.png)

`刷课.png`（189.3 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NzExNnw4MjRmMGNkMnwxNzkzNTc2Mzg3fDE2ODA4NnwxNTc4MDcx&nothumb=yes)  
2022-1-19 20:48 上传

上图是刷视频的代码，多看几个包就知道了，没啥可注意的。核心是发`heartbeat`的`payload`，然后根据学的课的树形结构在哪层，给扒开构造成`list`，循环。

## 刷讨论和刷图

![刷讨论](https://attach.52pojie.cn/forum/202201/19/205335vj77oxx444g70txx.png)

`刷讨论.png`（120.67 KB，下载次数：1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NzExN3xjZmY3Njk1N3wxNzkzNTc2Mzg3fDE2ODA4NnwxNTc4MDcx&nothumb=yes)  
2022-1-19 20:53 上传

讨论和刷图主要是这几个`request`，刷图只要知道你的图片课程的`id`就行，在鼠标操作上就是点进去就算完成。刷讨论需要一个`to_user`和`topic_id`，通过分析上一个请求就能拿到。

## 搜答案与做题

![搜答案](https://attach.52pojie.cn/forum/202201/19/205804n9q66r11s671206q.png)

`搜答案.png`（49.85 KB，下载次数：3）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NzExOXwyMTE1OGI3N3wxNzkzNTc2Mzg3fDE2ODA4NnwxNTc4MDcx&nothumb=yes)  
2022-1-19 20:58 上传

涉及做题的话因为不在浏览器这端判定，所以找一个做完题的，扒出来正确答案的字典，按照填空判断单选多选。

![答案的kv](https://attach.52pojie.cn/forum/202201/19/210636zi65ye0vxmgkricn.png)

`答案的kv.png`（41.84 KB，下载次数：2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NzEyMnxjMmNiMDk4N3wxNzkzNTc2Mzg3fDE2ODA4NnwxNTc4MDcx&nothumb=yes)  
2022-1-19 21:06 上传

做过的题的答案的`json`格式如上图

![做题](https://attach.52pojie.cn/forum/202201/19/210935x4r5w4cgt1w4rgzg.png)

`做题.png`（147.96 KB，下载次数：0）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjM2NzEyM3w0N2E3OWUwN3wxNzkzNTc2Mzg3fDE2ODA4NnwxNTc4MDcx&nothumb=yes)  
2022-1-19 21:09 上传

然后就可以刷了，要注意的是刷题太快会有请求过快的提示，每做一道题要等待一下，所以时间长，最好做个题是否做过的判定，没做过的再做，还有就是填空的`payload`和其他题不一样。
