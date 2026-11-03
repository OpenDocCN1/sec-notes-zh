
# [Web逆向] B站素材库平台下载按钮【tampermonkey脚本】

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1623614](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1623614)
- 抓取时间：2026-11-01T23:28:49

## B站素材库平台 视频下载

### 介绍

目标网站，B站素材库平台

```
https://cool.bilibili.com/square/video
```

B站素材库平台，有比较多质量还不错的素材，比如 nice、黑人流泪、土拨鼠尖叫，还有很多不错的绿幕素材，但是网页端竟然不允许下载，这就不方便在 PR 中使用。可是给我预览，不给我下载，好像不太可能吧，写了一个 tampermonkey 脚本，使用后就可以出现下载按钮了。

![](https://attach.52pojie.cn/forum/202204/18/104005i1g85mu8se1imv1s.png)

`image-20220418100843176.png`（302.48 KB，下载次数：1）

附件：[下载链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxNDY0N3xmY2RlOWM2ZXwxNzkzNTc1NzI2fDE2ODA4NnwxNjIzNjE0&nothumb=yes)

2022-4-18 10:40 上传

### 脚本主页地址与安装地址

tampermonkey 脚本已经发布。

再安装 tampermonkey 脚本管理器后，就可以去以下地址安装：

```
https://greasyfork.org/zh-CN/scripts/443506-b%E7%AB%99%E7%B4%A0%E6%9D%90%E5%BA%93%E5%B9%B3%E5%8F%B0%E4%B8%8B%E8%BD%BD%E6%8C%89%E9%92%AE
```

### 实现原理与注意事项

如果对实现原理不感兴趣，可以不用往下看了。

这个脚本非常简单，去掉样式的代码，核心代码就十几行，但感觉还是有地方可以讲的，比如：

- [chrome 阻止下载不安全的内容](https://blog.chromium.org/2020/02/protecting-users-from-insecure.html)，以及绕过的方法。
- 在 tampermonkey 的执行时间设置成 `@run-at document-idle` 仍然过早，使用 `@run-at context-menu` 时发现该网页屏蔽了鼠标右键，这个时候使用 `setTimeout()` 延迟执行基本可以满足要求。
- 最后即使设置了 `a` 标签的 `download` 属性，仍然无法改变下载文件的名字，这很奇怪。
- 总之脚本不难，因为一些规则或保护遇到了不少困难。

### 2022.4.28 更新

总体来说我写 tampermonkey 脚本很少，还是个新手，如果发现我有讲的不当的地方，欢迎在评论区指正。

首先我的目的就是可以下载视频，可以想到的方法有：

| 方法 | 特点 |
| --- | --- |
| 1. 直接按 `F12` 打开浏览器开发者工具审查元素得到视频的链接 |  |
