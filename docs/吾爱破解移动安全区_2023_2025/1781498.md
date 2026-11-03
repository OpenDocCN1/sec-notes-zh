
# [iOS 原创] 某IOS小说vip破解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1781498
- 抓取时间: 2026-11-01T21:44:34

时间紧迫，直接说重点

工具: `HTTP Catcher`

PS:乘着百度网盘在下视频，所以无聊找了个ios小说软件玩玩，有侵权的话联系我删即可

## 1. 抓包分析业务逻辑
通常`vip`都属于`User`或者`vip`之类的，所以，得到以下包

![](https://attach.52pojie.cn/forum/202305/04/222857fa9mb99i9k8i6lf1.png)

`1.png` (413.26 KB, 下载次数: 0)

2023-5-4 22:28 上传

原本图中的`true`全为`false`的，一看变量就知道作用了，所以直接重写响应体全改`true`;

## 2. 重写之后效果

![](https://attach.52pojie.cn/forum/202305/04/223104rovdz2irlxoxrsay.png)

`2.png` (540.21 KB, 下载次数: 0)

2023-5-4 22:31 上传

## 3. 抓包主要是对开发者的业务逻辑逆向分析，对`json`的含义的理解，哎，没啥说的，看上面图自己盲猜是哪款小说吧，曝光怕被敲警钟
