
# [Web逆向] 某美验证码及风控浅析一

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2069214
- 抓取时间: 2026-11-01T21:59:52

## 声明

本文章中所有内容仅供学习交流使用，不用于其他任何目的，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！若有侵权，请联系作者删除。

## 前言

记录一下逆向某登陆的总结!关键部分已做打码处理，网址敏感，暂不分享(笔者可不想再来一发！)，后续会提供分析样本，感谢理解。emmm，这次登录总共遇到了某美滑块验证码,某美指纹风控。接下来就进入正题吧老铁们！

## 接口分析

提交登录触发事件之后总共出现n个数据包。其中有用的就是验证码接口，风控接口和校验接口。先简单说一下吧，接口返回的顺序就是首先就是`register`接口，`web`接口，`fverify`接口，最后是`verify`接口。

### `register`接口

- `organization`：某美产品唯一标识
- `callback`：回调ID
- `sdkver`：标识版本
- `captchaUuid`：32位随机字符串
- `model`：模式，也就是这次要处理的验证码类型是滑块验证码
- 其他：都是固定的，不过有些值后续加密的时候会使用到。但是是固定的没必要深究，不过后续会稍微说一下的勒。

![](https://attach.52pojie.cn/forum/202510/31/194921qderw57wedlw0x48.png)

2025-10-26-18-19-13-image.png (76.52 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg1MnwyN2VmNTI2NHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg1MnwyN2VmNTI2NHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:49 上传

![](https://attach.52pojie.cn/forum/202510/31/195106s878s0778dm0ts6d.png)

2025-10-26-18-17-59-image.png (77.11 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg1NHw2NGM4MTUxYXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg1NHw2NGM4MTUxYXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:51 上传

### `web`接口

指纹的第一个加密的接口。

- `callback`：回调ID
- `organization`：某美产品唯一标识
- `smdata`：指纹加密
- `os`：他写的web，我估计是这个产品使用的是网站或是app。
- `version`：版本

![](https://attach.52pojie.cn/forum/202510/31/195129y7dm1447060ygp4c.png)

2025-10-26-18-34-59-image.png (139.98 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg1NXxlOTE0ZWZjNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg1NXxlOTE0ZWZjNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:51 上传

返回值是一个JSONP，看着参数不咋多勒，`smCB_*` 函数后续会被调用，并且接收相应的 JSON 数据，完成跨域数据获取。换句话说就是这些返回值后续会直接当作指纹第二次加密的值。为什么前面接口的jsonp不说，这个接口的JSONP我会专门拿来说，就是因为遇到坑了后续会细说，特别是`sign`跟`timestamp`和`len`这三个值特别重要，卡了我很久。

### `fverify`接口

`fverify`接口就是验证码校验接口

![](https://attach.52pojie.cn/forum/202510/31/195239g2eoh6yzx0bbqnze.png)

2025-10-26-18-48-19-image.png (25.15 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg1OXxkZWIwODc2NHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg1OXxkZWIwODc2NHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:52 上传

验证码返回值是`pass`就是通过，没通过就是`REJECT`。

### `verify`接口是校验接口

![](https://attach.52pojie.cn/forum/202510/31/195311nraaee0x1ye8bm89.png)

2025-10-26-18-55-39-image.png (89.07 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2MHxjZDlkNjUzZXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2MHxjZDlkNjUzZXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:53 上传

携带的参数有`account`，`clientId`，`deviceId`，`keyPairId`，`pageToken`，`password`，`promotionId`，`rid`，`smDeviceId`，`state`，`type`，`uiaType`。

`account`是网站加密之后的账号

`password`是网站加密之后的密码

其余参数除了固定之外的都在后续验证码跟指纹那块了。

![](https://attach.52pojie.cn/forum/202510/31/195332fjvbi2rj2wrmgmmk.png)

2025-10-26-18-54-50-image.png (22.64 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2MXxlMDI5N2E4Y3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2MXxlMDI5N2E4Y3wxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:53 上传

## 验证码

验证码相关的接口就只有`register`接口，`fverify`接口和`verify`接口。话不多说，直接开始逆向分析。

### `register`接口

![](https://attach.52pojie.cn/forum/202510/31/195419q0kctm8rcz98pbcz.png)

2025-10-26-20-15-59-image.png (54.19 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2Mnw2ZTU5N2FhNnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2Mnw2ZTU5N2FhNnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:54 上传

`register`就只有一个`captchaUuid`，不去刷新好像不变，参考了b站韭菜何某的教程说是固定的，但是刷新之后又是会变化的，依着知其然知其所以然的原则，我决定深究下去，于是我开始找各种文章，终于参考k佬的文章找出来了加密点，就浅析一下吧！

#### `captchaUuid`

![](https://attach.52pojie.cn/forum/202510/31/195437muxxhg7ekypvvhpx.png)

2025-10-31-09-23-37-image.png (51.2 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2M3w4Y2JmYjg1ZHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2M3w4Y2JmYjg1ZHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:54 上传

跟栈进去打断点刷新验证码获取，断点断住了

![](https://attach.52pojie.cn/forum/202510/31/195448kik5sjcajafjjdib.png)

2025-10-31-09-26-39-image.png (170.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2NHwyYmFkYTk5NnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2NHwyYmFkYTk5NnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:54 上传

搜索`captchaUuid`，可以看到`captchaUuid`在此文件总共出现了6次。全部打断点逐一分析。

![](https://attach.52pojie.cn/forum/202510/31/195514hj3jh3tq7c3qlnll.png)

2025-10-31-09-37-20-image.png (115.47 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2NXxiY2VmZTYyMnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2NXxiY2VmZTYyMnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:55 上传

这是所有断点唯一断住的地方，可以看到他是被`_0xda8467`赋值的，直接搜索`_0xda8467`，经过断点调试，此时`captchaUuid`早就被生成了，是从`_0x2b2937`取的值。

![](https://attach.52pojie.cn/forum/202510/31/195534un7rmwn8zlv72gwv.png)

2025-10-31-09-43-50-image.png (217.13 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2NnxjMzBhZjczNnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2NnxjMzBhZjczNnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:55 上传

经此断定，`captchaUuid`生成的地方不在这个文件，想找到生成点，跟栈回溯也好，搜索也罢。我的习惯是先全局搜索，实在找不到再回溯跟栈。全局搜索之后，`captchaUuid`还出现在`smcp.min.js`。

![](https://attach.52pojie.cn/forum/202510/31/195546on0hgrafan6n6z0z.png)

2025-10-31-10-22-31-image.png (31.39 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2N3wzMDQ5MTU0YnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2N3wzMDQ5MTU0YnwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:55 上传

在`smcp.min.js`搜索`captchaUuid`，发现他是由`_0x4954a5`赋值的。此处只截图了一个赋值点，代码里面有蛮多的地方`captchaUuid`都是被`_0x4954a5`赋值的。

![](https://attach.52pojie.cn/forum/202510/31/195600ijwdzxw02rruj2wr.png)

2025-10-31-10-29-22-image.png (175.19 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2OHwxYzM3ZDFiNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2OHwxYzM3ZDFiNHwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:56 上传

搜索`_0x4954a5`很容易就找到了生成点，但是代码混淆了。咱们直接手动解混淆看看代码究竟写了什么。

![](https://attach.52pojie.cn/forum/202510/31/195620u2bjvqrrtmubbb21.png)

2025-10-31-10-36-18-image.png (179.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgxMDg2OXwxNDY1YzZhMXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDg2OXwxNDY1YzZhMXwxNzkzNTcwMzg5fDE2ODA4NnwyMDY5MjE0&nothumb=yes)

2025-10-31 19:56 上传

这里有个或运算，它会从左到右求值，返回第一个真值表达式的结果，如果从`userConfig`取不到`captchaUuid`，就是`undefined`，则运行后面的函数。如果`userConfig`里面有`captchaUuid`则一直没生成`captchaUuid`。这也就是为什么过验证码的时候`captchaUuid`一直没变，刷新之后就变了。而后面的函数就是生成`captchaUuid`的地方。
