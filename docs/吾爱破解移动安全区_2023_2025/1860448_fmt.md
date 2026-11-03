
# [Android 原创] 新手入门，某作文应用 `sign` 参数分析

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1860448>
- 抓取时间：2026-11-01T21:38:00

## 抓包与初步观察

安装，开启 charles，打开 app。

![Screenshot_20231122-175708.png（1.15 MB）](https://attach.52pojie.cn/forum/202311/22/183502joxgmdrdjej0dgx0.png)

然后随机点击一篇文章。

![Screenshot_20231122-180014.png（355.1 KB）](https://attach.52pojie.cn/forum/202311/22/183543i3ypzzlwzpwbrwm3.png)

查看 charles，发现请求有 `sign` 参数（图片已脱敏处理）。

![微信截图_20231122175948.png（125.03 KB）](https://attach.52pojie.cn/forum/202311/22/190306hbh5b3qj57bel0ew.png)

## 查壳与脱壳

查壳。

![Screenshot_20231122-180032 (2).png（116.85 KB）](https://attach.52pojie.cn/forum/202311/22/183824u55ayst91it3s3zy.png)

壳代码。

![Screenshot_20231122-180137.png（101.46 KB）](https://attach.52pojie.cn/forum/202311/23/182309e4u4vz4h9ri4rceu.png)

（不是本文重点，这里略过）打开脱壳的 `dex`。

![Screenshot_20231122-180808.png（122.32 KB）](https://attach.52pojie.cn/forum/202311/22/183930vy9azyw2zg1xsxgb.png)

## 定位 `sign` 生成逻辑

搜索字符串 `sign`。

![Screenshot_20231122-180823.png（119.45 KB）](https://attach.52pojie.cn/forum/202311/22/184053d6jazjb4j78tbv6j.png)

结果很多。

![Screenshot_20231122-180832.png（196.8 KB）](https://attach.52pojie.cn/forum/202311/22/184124osa3cr7a5m5vv5s3.png)

继续搜索 `timestamp`。

![Screenshot_20231122-180904.png（194.41 KB）](https://attach.52pojie.cn/forum/202311/22/184155u11tteqmxi1qxmfh.png)

搜索结果只有几个，这里暂时不分析其他结果（与加密无关的类），直接打开最后那个类，转成 `java` 代码。

![Screenshot_20231123-181520.png（197.61 KB）](https://attach.52pojie.cn/forum/202311/23/182201nlcznzmdpd2co8zl.png)

![Screenshot_20231122-181055.png（296.58 KB）](https://attach.52pojie.cn/forum/202311/22/184403enqmdthopte6bukm.png)

可以看到 `sign` 参数由此方法生成。切换 `smail` 跳转过去。

![Screenshot_20231122-181127.png（268.66 KB）](https://attach.52pojie.cn/forum/202311/22/184706m9bm69ypk1m2lx2v.png)

标准 `MD5`，反编译有点问题，应为 `int`。

![Screenshot_20231122-181142.png（206.57 KB）](https://attach.52pojie.cn/forum/202311/23/182138mvtcjvd3vnnnnznn.png)

看看 `protected` 方法。

![Screenshot_20231122-181232.png（274.21 KB）](https://attach.52pojie.cn/forum/202311/23/182446hqkaiykju2jnaz0g.png)

把 `List` 转成了 `String`。

## 结束

分析结束，谢谢大家。

本文仅供学习交流，其他用途后果自负。

新人第一篇，如有不足，敬请指正。
