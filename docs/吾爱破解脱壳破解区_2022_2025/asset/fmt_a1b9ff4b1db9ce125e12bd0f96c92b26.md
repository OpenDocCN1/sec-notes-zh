
# [Web逆向] QTranslate翻译器接口修复(10月18日更新)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1695613
- 抓取时间: 2026-11-01T23:08:29

非常感谢大家的支持！很惊喜还有这么多朋友在坚持喜欢并使用着这款软件，那我们应该做点力所能及的事

我在gitee上新建了仓库，专用于分享QTranslate的`Services`，为了让这个翻译神器继续流传，我们一起加油

QTranslate的`Services`分享

https://gitee.com/linpure/qtranslate-services

”作为一名工科生我才20几岁，我只想找一款好用的翻译软件，我有什么错！“ ——我不是译神

今天故事的主角就是大名鼎鼎的QTranslate，软件小巧，功能专业，可谓麻雀虽小五脏俱全，从取词翻译、朗读到全文翻译、再到图片OCR翻译，QTranslate 无所不能。

而且其强大的粘手性，让你用过QTranslate之后，再面对其他五花八门的翻译软件，总感觉差强人意。

![](https://attach.52pojie.cn/forum/202210/04/150804sd0izb9zvhpobzwy.png)
`image.png` (40.31 KB, 下载次数: 5) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1ODM2NHw0NThlOTRlZHwxNzkzNTc0NTA3fDE2ODA4NnwxNjk1NjEz&nothumb=yes) 2022-10-4 15:08 上传

然而就是这样一款堪称完美的神器，却也面临着不少问题：

如

![](https://attach.52pojie.cn/forum/202210/04/145505n1n1hzkoohjd730x.png)
`image.png` (12.68 KB, 下载次数: 5) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1ODM2MHxkZTc0ZTEyNHwxNzkzNTc0NTA3fDE2ODA4NnwxNjk1NjEz&nothumb=yes) 2022-10-4 14:55 上传

如

![](https://attach.52pojie.cn/forum/202210/04/145554l4x8mtpm0p8c080m.png)
`image.png` (11.41 KB, 下载次数: 5) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1ODM2MnxmOWU1NTI0NnwxNzkzNTc0NTA3fDE2ODA4NnwxNjk1NjEz&nothumb=yes) 2022-10-4 14:55 上传

再如

![](https://attach.52pojie.cn/forum/202210/04/145620eiilj9nnqnx6lmll.png)
`image.png` (10.79 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1ODM2M3wwNGI5NjhhOXwxNzkzNTc0NTA3fDE2ODA4NnwxNjk1NjEz&nothumb=yes) 2022-10-4 14:56 上传

此情此景，可谓”世间无限丹青手，一片伤心画不成“，更让人绝望的是这款优秀的软件未开源，也暂未找到前辈大神们的修改，唯一的办法就是等待作者的更新......

可认真学习代码两年半，喜欢唱歌跳舞和rap，又热爱学习的我怎么能轻易让被困难打败呢

接下来就来亲自打怪吧，简单总结一下我的思路，没有深入研究，只算是抛砖引玉，还请各位大佬指点。

安装版本和便携版的文件基本一致，如图所示

我们可以看到的接口代码主要存放在`Services`文件夹下，供主程序去调用

![](https://attach.52pojie.cn/forum/202210/04/151215aq985qq5yezy1h8y.png)
`image.png` (33.22 KB, 下载次数: 3) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1ODM2OHxiOGMxNzUwNnwxNzkzNTc0NTA3fDE2ODA4NnwxNjk1NjEz&nothumb=yes) 2022-10-4 15:12 上传

各个文件夹对应存放的不同接口处理程序，`common.js`则是一些公共函数。

每一个文件夹里都有一个`server.js`的同名文件，以Google翻译为例

[JavaScript] 纯文本查看 复制代码
