
# [Web逆向] 【JS逆向】某条signature逆向补环境

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2047209
- 抓取时间: 2026-11-01T22:06:54

本文中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，抓包内容、敏感网址、数据接口等均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关.本文章未经许可禁止转载，禁止任何修改后二次传播，擅自使用本文讲解的技术而导致的任何意外，作者均不负责

站点：aHR0cHM6Ly9zby50b3V0aWFvLmNvbS9zZWFyY2g/ZHZwZj1wYyZzb3VyY2U9dHJlbmRpbmdfY2FyZCZrZXl3b3JkPSVFNyVCRSU4RSVFNSU5QiVCRCVFNiVCNCU5QiVFNiU5RCU4OSVFNyU5RiVCNiVFNCVCOCU4MCVFOCVCRCVBNiVFOCVCRSU4NiVFNSU4NiVCMiVFNiU5MiU5RSVFNCVCQSVCQSVFNyVCRSVBNA==

视频：[//www.bilibili.com/video/BV1MhgTzrETQ" target="_blank" rel="noopener noreferrer nofollow"](https://www.bilibili.com/video/BV1MhgTzrETQ:rel="nofollow noopener" href="https)

## 1. 逆向目标

先访问站点，发现有2次请求，且第一次请求的响应内容空白。第1次请求后紧接的第2次请求就是渲染的页面。可见第1次请求必然做了什么事情，这也是典型的动态cookie场景。

![](https://attach.52pojie.cn/forum/202507/21/205455pk1zitdii412z7q0.png)

1.png (48.62 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3MHwwODQ0NDRmYnwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3MHwwODQ0NDRmYnwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:54 上传

观察第1次请求的响应头：有一些`setcookie`等字段

![](https://attach.52pojie.cn/forum/202507/21/205457wmffmxfuhizxfz7m.png)

2.png (58.29 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3MXw1NzM2MGNjN3wxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3MXw1NzM2MGNjN3wxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:54 上传

再看看第2次请求成功：

url、参数并没有区别，只不过带了个cookie，其中`__ac_nonce`来自于第1次请求返回，而多的这个`__ac_signature`就要处理了。

![](https://attach.52pojie.cn/forum/202507/21/205459nv7n8rzrxjn7ztzl.png)

3.png (35.21 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc5MDI3Mnw1OTMzZWQzZXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3Mnw1OTMzZWQzZXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:54 上传

此时利用`postman`发一次包看第1次请求响应内容究竟是什么：

![](https://attach.52pojie.cn/forum/202507/21/205501ztlfmqf5wyve5z5n.png)

4.png (130.71 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3M3wwOGJiODFjMXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3M3wwOGJiODFjMXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

是一大段JS，拿出来看：显然是个jsvmp，且能找到生成的函数是：

```
__ac_signature=window.byted_acrawler.sign("",__ac_nonce);
```

![](https://attach.52pojie.cn/forum/202507/21/205504o54jaq6zhj5h0hhf.png)

5.png (91.33 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3NHwxZjViMWZiZXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3NHwxZjViMWZiZXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

而`__ac_nonce`已经可以从第1次请求拿到，接下来就看怎么让这个调用能够完成。

## 2. 补环境

由于是个大jsvmp，简单做法直接补环境处理。

先尝试运行代码看看：报错防盗链。。。

![](https://attach.52pojie.cn/forum/202507/21/205506b5vjzayc2ydvaa12.png)

6.png (74.71 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3NXxmMTNiZDRkN7wxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3NXxmMTNiZDRkN7wxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

那常规操作，先补几个基础dom对象，再祭出`Proxy`直接挂！

提一嘴，`watch`方法来自于B站up主：无名的Coder，主页：[//space.bilibili.com/376969345" target="_blank" rel="noopener noreferrer nofollow"](https://space.bilibili.com/376969345:rel="nofollow noopener" href="https)

```
function watch(obj, name) {
    return new Proxy(obj,{
        get: function(target, property, receiver) {
            try {
                if (typeof target[property] === "function") {
                    console.table([`对象 => ${name} ,读取属性:`, property + `,值为:` + 'function' + `,类型为:` + (typeof target[property])]);
                } else {
                    console.table([`对象 => ${name} ,读取属性:`, property + `,值为:` + target[property] + `,类型为:` + (typeof target[property])]);
                }
            } catch (e) {}
            return target[property];
        },
        set: function(target, property, newValue, receiver) {
            try {
                console.table([`对象 => ${name} ,设置属性:`, property + `,值为:` + newValue + `,类型为:` + (typeof newValue)]);
            } catch (e) {}
            return Reflect.set(target, property, newValue, receiver);
        }
    });
}
window = global;
document= {
};
location= {
};
navigator= {
};
window = watch(window, "window");
location = watch(location, "location");
document = watch(document, "document");
navigator = watch(navigator, "navigator");
require("./bdm.js")
var sign = window.byted_acrawler.sign("","123");
console.log(sign);
```

运行打印看看：

- `sessionStorage`和`localStorage`

![](https://attach.52pojie.cn/forum/202507/21/205508uvuce2i0hhff8dhh.png)

7.png (35.91 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3NnxjYTUwYTE2ZHwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3NnxjYTUwYTE2ZHwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

- `cookie`

![](https://attach.52pojie.cn/forum/202507/21/205510o1m6ko1jw4mw41jd.png)

8.png (21.13 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3N3xkOTA1YzE0MnwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3N3xkOTA1YzE0MnwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

- `referer`

![](https://attach.52pojie.cn/forum/202507/21/205513g23rlwt1w1ztl843.png)

9.png (21 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3OHw1NDVkZmYxNXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3OHw1NDVkZmYxNXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

- `location`

![](https://attach.52pojie.cn/forum/202507/21/205515uvgutjt4ptgrp8dz.png)

10.png (20.72 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc5MDI3OXwzYzMwYmNkMXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDI3OXwzYzMwYmNkMXwxNzkzNTcwODEyfDE2ODA4NnwyMDQ3MjA5&nothumb=yes)

2025-7-21 20:55 上传

行，那就全补上
