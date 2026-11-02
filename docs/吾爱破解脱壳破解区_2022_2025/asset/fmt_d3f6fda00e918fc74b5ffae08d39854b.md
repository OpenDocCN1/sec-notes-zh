
# [Web逆向] 【原创】 猿人学第一题解析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1726459
- 抓取时间: 2026-11-01T23:03:34

本文适合像我一样得小白，大牛请略过，主要是记录，哈哈哈。

## 一、 发现一个练手的好地方

昨日在浏览论坛的时候，发现大牛 发了一个帖子 猿人学第二题 手把手补环境

链接地址：[https://www.52pojie.cn/thread-1639885-1-1.html](https://www.52pojie.cn/thread-1639885-1-1.html)

![image.png](https://attach.52pojie.cn/forum/202212/13/165756zhyyhu9pmb2y2d0a.png)

[`image.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3ODk3MXxjMzBhODA4MHwxNzkzNTc0MjExfDE2ODA4NnwxNzI2NDU5&nothumb=yes) (76.3 KB, 下载次数: 2) 2022-12-13 16:57 上传

依照Anekys 大牛提供的地址，进入之后返现者的是个练手的好地方，哈哈哈，js解密过程中，能遇到的问题，几乎在这边全部遇到了。并且还有解题排名，膜拜各种大牛的机会。

## 二、 正式开始

### 2.1 过掉debugger 分析页面

首先看第一题

![image.png](https://attach.52pojie.cn/forum/202212/13/194057wu3707n727qqg25q.png)

[`image.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAxN3xiMjc1YzdjNnwxNzkzNTc0MjExfDE2ODA4NnwxNzI2NDU5&nothumb=yes) (32.73 KB, 下载次数: 1) 2022-12-13 19:40 上传

要求是获取机票的平均价格，填入答案。

依照国际管理，F12，开发者工具，`ctrl+shift+r` 重新加载页面，不出意外的界面停止在`debugger`

![image.png](https://attach.52pojie.cn/forum/202212/13/194528vzfofvolfrbsf5l8.png)

[`image.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAxOHw1N2Y0NDBkOHwxNzkzNTc0MjExfDE2ODA4NnwxNzI2NDU5&nothumb=yes) (11.37 KB, 下载次数: 2) 2022-12-13 19:45 上传

直接在第二行除邮件编辑断点，值填写`false`，过掉`debugger`。

### 2.2 依次点击每一页的值查看，请求值变化

- 第一页 `/api/match/1?m=5c70fb1948228fdf84f74568b037fa11|81671008953`
- 第二页 `/api/match/1?page=2&m=922c0993d9df7bc038a09470c17d5901|81671009628`
- 第三页 `api/match/1?page=3&m=e9526008c0fde5ba4e4499ec78289239|81671032379`

不难看出请求不能的页面出了添加了`page`的参数外，`m`的值发生了变化，如果在全局搜索里面搜索`m`的话，这个值太多了直接去启动器中查看堆栈调用

![forum.png](https://attach.52pojie.cn/forum/202212/13/195947q9lzrk4hdw4w6kdw.png)

[`forum.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAyMHxkNGQxZWQzOXwxNzkzNTc0MjExfDE2ODA4NnwxNzI2NDU5&nothumb=yes) (37.66 KB, 下载次数: 1) 2022-12-13 19:59 上传

排除掉ajax 后，第一个就是`request()`函数了，点击函数进入源代码区查看。

![image.png](https://attach.52pojie.cn/forum/202212/13/200211wyiiboygz5h4y5u4.png)

[`image.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTAyMXwwYjFjNDY3NXwxNzkzNTc0MjExfDE2ODA4NnwxNzI2NDU5&nothumb=yes) (102.36 KB, 下载次数: 1) 2022-12-13 20:02 上传

### 2.3 解密混淆和加密得js代码

js代码被加密混淆了，在线搜索js解密工具，其实不解密也能看，但是阅读的难度就比较高了。

解码后得到

[JavaScript] 纯文本查看 复制代码
