
# 数值搜索修改游戏内树苗

## 搜索并定位树苗数值

[附件：xTree](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM1NXxhZDVmNjM5MHwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:22 上传）

使用 `js` 内存漫游，可以通过搜索数值来定位树苗存放位置。`js` 内存漫游网址根据树苗数量在控制台搜索：

![xTree](https://attach.52pojie.cn/forum/202311/26/002446t5a6j5d68638wd8w.png)

`image.png`（5.53 KB，下载次数：2）

[附件：valueS](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM1NnxkMTBkZmY0MHwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:24 上传）

![valueS](https://attach.52pojie.cn/forum/202311/26/002516o99b1b911ro53d92.png)

`image.png`（104.76 KB，下载次数：1）

[附件：result](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM1N3xlMDZmOTZhYXwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:25 上传）

![result](https://attach.52pojie.cn/forum/202311/26/002527j3l7s9dd76b7ak66.png)

`image.png`（34.14 KB，下载次数：1）

[附件：r2](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM1OHw0NzVjZTczZXwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:25 上传）

搜索出的结果可以看到变量值设置位置和类信息，`js` 中跟进这个 `updatePlayerResource` 里面看看。

![r2](https://attach.52pojie.cn/forum/202311/26/002542lmut8lm2ludie2h9.png)

`image.png`（38.33 KB，下载次数：2）

[附件：r3](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM1OXw3NjBkODE5NHwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:25 上传）

![r3](https://attach.52pojie.cn/forum/202311/26/002600sh20c0bq01zcl0zm.png)

`image.png`（327.05 KB，下载次数：2）

## 跟进更新逻辑

[附件：update](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM2MHxiZWM0MzRkNHwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:26 上传）

可以看出这里是界面增加树苗时 ui 变动的代码，而这里：

```javascript
d.mainManger().fightModule.getResourceNumByType(l.EResourceType.GOLD)
```

可能就是真实树苗数量。对这里下断点，然后控制台输出 `d.mainManger().fightModule.getResourceNumByType(l.EResourceType.GOLD)` 看看。

![update](https://attach.52pojie.cn/forum/202311/26/002720cyj6m0cb6yizqyer.png)

`image.png`（27.95 KB，下载次数：2）

[附件：r4](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM2MnwwNDJhOTM4MnwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:27 上传）

![r4](https://attach.52pojie.cn/forum/202311/26/002730abzriqr55iq5erzd.png)

`image.png`（45.96 KB，下载次数：1）

## 验证并修改数值

[附件：trees](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM2M3xkY2Q0ODkxMXwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:27 上传）

可以看到两个值恰好相等，在输出 `d.mainManger().fightModule` 看看类的具体信息。

![trees](https://attach.52pojie.cn/forum/202311/26/002753xc448fpb4xynw3y8.png)

`image.png`（134.6 KB，下载次数：1）

[附件：r5](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1OTM2NHwwMGI2ODhiYXwxNzkzNTcyOTUxfDE2ODA4NnwxODYxNzYw&nothumb=yes)（2023-11-26 00:27 上传）

这里就可以很方便修改其他数值。
