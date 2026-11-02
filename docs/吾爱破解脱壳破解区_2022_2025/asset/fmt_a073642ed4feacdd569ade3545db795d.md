
# [Web逆向] [验证码逆向]某手势验证码逆向分析

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1888845>
- 抓取时间：2026-11-01T22:38:13

## [验证码逆向]某手势验证码逆向分析

> **本文章所有内容仅供学习和研究使用，本人不提供具体模型和源码。若有侵权，请联系我立即删除！维护网络安全，人人有责。**

## 前言

手势验证码是一种绘制轨迹进行人机的验证码，我曾经遇到过几次。

查了查网上没有较好的教程，便打算自己写一篇文章。

目标网址：

```
aHR0cHM6Ly93d3cudmFwdGNoYS5jb20vI2RlbW8=
```

手势验证码长这样

![1.png](https://attach.52pojie.cn/forum/202402/08/005139xwgk0f0s6vn6hpwb.png)

`1.png`（246.65 KB，下载次数：1）；附件 1，2024-2-8 00:51 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA1NXw4MjY0N2VlMXwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

我将该验证码的协议逆向分析和验证码识别分为 2 篇文章，本文主要研究该验证码的逆向分析。

## 目录

- 请求分析
  - 准备工作
  - 4种请求
  - `config` 请求
  - `get` 请求
  - `validate` 请求
- 获取验证码
  - `en` 参数分析
  - `fp` 指纹分析
  - `fp` 指纹分析2
  - 构造 `en` 参数
  - 获取数据
- 图片还原
  - 还原代码分析
  - 获取图片还原顺序
  - python 还原图片
- 图片识别
  - 关于图片识别
- 构造请求
  - 请求分析
  - 轨迹分析
  - 轨迹加密
  - 请求验证
- 总结

## 请求分析

### 准备工作

首先，每个验证码平台都有唯一的业务标识，用于区分不同网站。我们应该养成看文档的好习惯，一般情况下直接在该验证码的官网找到 Web 端开发文档

![2.png](https://attach.52pojie.cn/forum/202402/08/005141wnd0n07szht68sjl.png)

`2.png`（101.16 KB，下载次数：2）；附件 2，2024-2-8 00:51 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA1NnxkZTBhNWMwNXwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

可以看到可以看到验证码初始化的代码，`vid` 就是唯一标识，然后在验证码出现的页面全局搜索 `vaptcha(`

![3.png](https://attach.52pojie.cn/forum/202402/08/005145yxj6lxnerrgjnxrw.png)

`3.png`（165.06 KB，下载次数：2）；附件 3，2024-2-8 00:51 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA1N3w0YmM5Nzg2MnwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

然后记住该 `id` 为 `59b252ed57f5a2111xxxxxxx`（为了脱敏，部分真实数据我将用 `xxx` 代替）

### 4种请求

该验证码没有 `debugger` 反爬措施，直接打开浏览器开发者工具抓包即可。

![4.png](https://attach.52pojie.cn/forum/202402/08/005150lorbf3mmforl4sz6.png)

`4.png`（30.97 KB，下载次数：2）；附件 4，2024-2-8 00:51 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA1OHwyYzZiZDY4MHwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

可以看到一共有 4 种请求，其中 `59b252ed57f5a2111xxxxxxx` 接口就是用户的 `vid`，用于获取验证服务器。

![5.png](https://attach.52pojie.cn/forum/202402/08/005152t5gvbcb08rl3fozk.png)

`5.png`（40.94 KB，下载次数：1）；附件 5，2024-2-8 00:51 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA1OXxhMGE2N2FiM3wxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

返回的 `api` 就是下面 3 个请求的域名。

### `config` 请求

该请求是获取验证码配置的，即一些 SDK 地址和版本。

**请求数据**

![6.png](https://attach.52pojie.cn/forum/202402/08/005226ezjuzpjllua3pf3l.png)

`6.png`（23.44 KB，下载次数：3）；附件 6，2024-2-8 00:52 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2MXw1MmFlMmJiZnwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

**响应数据**

![7.png](https://attach.52pojie.cn/forum/202402/08/005228qitodyt6oopdawt9.png)

`7.png`（34.09 KB，下载次数：0）；附件 7，2024-2-8 00:52 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2Mnw2NDRiN2MxZXwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

`vi` 参数就是 `vid`

其中我们只需要响应数据的 `knock` 值

那么 `u` 值是怎么生成的呢？

我们开始跟栈

![8.png](https://attach.52pojie.cn/forum/202402/08/005230eckvzvmjkun4zc2v.png)

`8.png`（34.24 KB，下载次数：2）；附件 8，2024-2-8 00:52 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2M3w3NjM4ZTMwYXwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

`getConfig` 这个函数名那么明显，我们进去看看

![9.png](https://attach.52pojie.cn/forum/202402/08/005232mm0uykt8bb85ym05.png)

`9.png`（34.54 KB，下载次数：1）；附件 9，2024-2-8 00:52 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2NHw0MWQ0OTE4ZHwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

可以明显看到，`u` 值的来源是 `localStorage` 中的 `vaptchanu`，那么意味着可以是空值。

当把该页面的缓存清空后，可以发现 `u` 值为空，这里我就不放图了，有兴趣大家试试。

### `get` 请求

顾名思义，该请求是获取验证码数据的。

**请求数据**

![10.png](https://attach.52pojie.cn/forum/202402/08/005224wh1839gql4zr1zi0.png)

`10.png`（25.32 KB，下载次数：1）；附件 10，2024-2-8 00:52 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2MHxkOTJkNTUzZXwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

**响应数据**

![11.png](https://attach.52pojie.cn/forum/202402/08/005322zf15y9kirkz0zyn0.png)

`11.png`（31.68 KB，下载次数：1）；附件 12，2024-2-8 00:53 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2NnxiMzhhODVjMnwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

可以看到 `k` 参数就是 `config` 接口返回的 `knock`

而 `en` 参数包含一些环境检测，我将在下文分析

### `validate` 请求

该请求用于验证是否正确。

**请求数据**

![12.png](https://attach.52pojie.cn/forum/202402/08/005324x8oz8hwomwmvobhi.png)

`12.png`（41.08 KB，下载次数：1）；附件 13，2024-2-8 00:53 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2N3xkMjE4ZDZlY3wxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

**响应数据**

![13.png](https://attach.52pojie.cn/forum/202402/08/005327yce2sacs7amgeeee.png)

`13.png`（17.29 KB，下载次数：2）；附件 14，2024-2-8 00:53 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2OHxmYjc1MDhjZnwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

返回的 `token` 值就是我们的目标，我们携带该值请求需要验证的接口就可以返回数据了。

## 获取验证码

### `en` 参数分析

我们进入 `post` 的前面的栈

![14.png](https://attach.52pojie.cn/forum/202402/08/005329smgzj8gq08l8669l.png)

`14.png`（38.83 KB，下载次数：0）；附件 15，2024-2-8 00:53 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2OXwyZjg2ZTNhM3wxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

![16.png](https://attach.52pojie.cn/forum/202402/08/005357itlcwqtfzfqgucxk.png)

`16.png`（112.48 KB，下载次数：2）；附件 16，2024-2-8 00:53 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA3MXxlYjRlODFhOHwxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

由于 `switch` 的存在，代码是分步执行的，我们在 `switch` 打上记录点，记录执行顺序

![15.png](https://attach.52pojie.cn/forum/202402/08/005320t9iooq4noty0oi48.png)

`15.png`（122.37 KB，下载次数：1）；附件 11，2024-2-8 00:53 上传

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NTA2NXw0MmVkYzUwY3wxNzkzNTcyNjkwfDE2ODA4NnwxODg4ODQ1&nothumb=yes)

可以看到，获取了一些 `localStorage` 的内容，

`hex_md5` 其实就是 `md5` 加密（严格来说不算加密，因为 `md5` 无法解密）

我们清空缓存后重新请求可以发现执行顺序。

```
6
0
1
2
5
```

我们只需要把执行过的函数扣下来即可。

我只说几个重点

- 获取了 `localStorage` 的变量都是三元运算符，当 `localStorage` 里的值为空时就会设置默认值
- `_0x502684['GenerateFP']` 和 `_0x5d0164['GenerateFP']` 都是浏览器指纹，但是后者是 `Promise` 异步任务，我接下来会分析这两个的区别。
- `_0x4f54b3['secretC']` 是一个固定值。

![17.png](https://attach.52pojie.cn/forum/202402/08/005400mh39o2ei2ogxdhoo.png)

`17.png`（48.13 KB，下载次数：1）
