
# [Web逆向] 记一次简单解析某免费影视站播放地址的全过程（适合小白练手）

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2033927
- 抓取时间: 2026-11-01T22:09:09

## 一、免责声明
本文章中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关。

## 二、目标地址
`aHR0cHM6Ly93d3cuZGFtaXlzLmNjL3ZvZHBsYXkvMjItMy0xLmh0bWw=`

## 三、解析思路
### 1、对于一般的免费影视站点，真实播放地址一般都可以通过`Ctrl + U`查看HTML源码找到。
#### ① 在打开的源码标签页（即`view-source:`网址）中，通过`Ctrl + F`查找`.m3u8`或`.mp4`。
>
如果能找到`.m3u8`或`.mp4`URL地址并通过验证，那么恭喜你，证明这网站基本没有对视频地址加密.
比如以下这种：
![](https://attach.52pojie.cn/forum/202505/24/171329kkolylunlln2x26i.jpg)
m3u8.jpg (28.96 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTA0OXxkMmIyN2JlZXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTA0OXxkMmIyN2JlZXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
2025-5-24 17:13 上传
#### ② 如果没找到`.m3u8`或`.mp4`相关明码视频地址，尝试搜索`aaa=`。
>
（为什么？问就是见多了，很多免费影视站点都是`var player_aaaa=`声明载入明文或加密的播放地址）
### 2、通过HTML源码搜索`aaa=`找到的是加密的播放地址，则需要`F12` 打开开发工具分析
本次目标网站就是通过`aaa=`关键词找到相关加密地址.
这种就需要找出解密算法。

## 四、解密加密地址分析
### 1、先从HTML查找
对于本次目标站点地址，`Ctrl + U`的时候，遇到反调试检测机制。
没关系，咱们直接（`view-source:aHR0cHM6Ly93d3cuZGFtaXlzLmNjL3ZvZHBsYXkvMjItMy0xLmh0bWw=`）打开HTML源码搜索
尝试搜索`aaa=`得到这段疑似被加密的播放地址："JTY4JTc0JTc0JTcwJTczJTNBJTJGJTJGJTc2JTJFJTcxJTcxJTJFJTYzJTZGJTZEJTJGJTc4JTJGJTYzJTZGJTc2JTY1JTcyJTJGJTMyJTc3JTMyJTZDJTY1JTY3JTc0JTMwJTY3JTM4JTdBJTMyJTM2JTYxJTZDJTJGJTc1JTMwJTMwJTMyJTM5JTM2JTcwJTM2JTc0JTM1JTMwJTJFJTY4JTc0JTZEJTZD"
![](https://attach.52pojie.cn/forum/202505/25/024108mi4msjeiwwwtzvdk.jpg)
jiami.jpg (164.49 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTE3OHxhODAzYjEwMXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE3OHxhODAzYjEwMXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
jiema
2025-5-25 02:41 上传
根据经验，这应该是`Base64`加密了，直接在当前窗口`F12`打开控制台使用`atob()`，比如输入`atob("JTY4JTc0JTc0JTcwJTczJTNBJTJGJTJGJTc2JTJFJTcxJTcxJTJFJTYzJTZGJTZEJTJGJTc4JTJGJTYzJTZGJTc2JTY1JTcyJTJGJTMyJTc3JTMyJTZDJTY1JTY3JTc0JTMwJTY3JTM4JTdBJTMyJTM2JTYxJTZDJTJGJTc1JTMwJTMwJTMyJTM5JTM2JTcwJTM2JTc0JTM1JTMwJTJFJTY4JTc0JTZEJTZD")`，得到明显是被转码的URL地址，再通过`unescape()`反转码，得到明文地址，但这个明显是某TX的播放地址，并非可以直接下载的真实地址。
![](https://attach.52pojie.cn/forum/202505/25/024358p1dyjiz10jzhghwi.jpg)
atob.jpg (183.53 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTE3OXw0NjQwNjg4NXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE3OXw0NjQwNjg4NXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
2025-5-25 02:43 上传
### 2、HTML找到的并非真实地址，需要`F12`打开浏览器开发工具调试解密
遭遇`F12`快捷键打不开（即反调试检测），我们直接手动打开。
![](https://attach.52pojie.cn/forum/202505/25/024457zd2djaql1masp349.jpg)
你知道.jpg (25.42 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTE4MHw2ZGQ4MmIwZHwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE4MHw2ZGQ4MmIwZHwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
2025-5-25 02:44 上传
发现反调试`debugger`代码是硬编码在HTML网页中的，咱们直接将相关代码注释掉（暴力破解方法），很幸运跳过反调试检测，没有进一步保护机制。
![](https://attach.52pojie.cn/forum/202505/25/024533g5q33qljvj2cjl32.jpg)
debugger.jpg (219.24 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc3OTE4MXw2NjRkMDU0ZnwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE4MXw2NjRkMDU0ZnwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
d1
2025-5-25 02:45 上传
比如修改成`// debugger;`，或者删除相关`debugger`代码然后保存。
![](https://attach.52pojie.cn/forum/202505/25/024620orj5ln5wl6wr4z25.jpg)
debugger2.jpg (21.08 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTE4Mnw2MmYzYTU2ZXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE4Mnw2MmYzYTU2ZXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
d2
2025-5-25 02:46 上传
刷新网页发现注释掉`debugger`生效，不再阻塞我们调试。
这时打开开发工具“网络”标签，刷新当前播放页重载，发现可疑的API入口`https://jiexi.shipinbofang.net:7898/api.php`，而且名称就是请求地址包含“`jiexi`”（解析）字符串。
![](https://attach.52pojie.cn/forum/202505/25/024659lrbl3jjjnlh7kbfp.jpg)
api.jpg (147.83 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=Mjc3OTE4M3xiNThmY2U4YXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE4M3xiNThmY2U4YXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
api1
2025-5-25 02:46 上传
这个时候，不管它是什么，我们都需要验证它的响应数据是不是真实地址数据包。
```
from DrissionPage import SessionPage
page = SessionPage()
def fetch_video_data():
    """
    发送 POST 请求到指定的 URL，并返回响应数据
    """
    url = "https://jiexi.shipinbofang.net:7898/api.php"
    headers = {
        "accept": "application/json, text/javascript, */*; q=0.01",
        "accept-language": "zh-CN,zh;q=0.9,en;q=0.8",
        "cache-control": "no-cache",
        "content-type": "application/x-www-form-urlencoded; charset=UTF-8",
        "pragma": "no-cache",
        "priority": "u=1, i",
        "sec-ch-ua": "\"Chromium\";v=\"136\", \"Microsoft Edge\";v=\"136\", \"Not.A/Brand\";v=\"99\"",
        "sec-ch-ua-mobile": "?0",
        "sec-ch-ua-platform": "\"Windows\"",
        "sec-fetch-dest": "empty",
        "sec-fetch-mode": "cors",
        "sec-fetch-site": "same-origin",
        "sec-fetch-storage-access": "active",
        "x-requested-with": "XMLHttpRequest",
        "User-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36 Edg/136.0.0.0"
    }
    body = "url=https://v.qq.com/x/cover/2w2legt0g8z26al/u00296p6t50.html&time=1748103938&key=5655407d56b0b456686eb2da7b620b21"

    page.post(url, headers=headers, data=body)
    return page.json

response_data = fetch_video_data()
print(response_data)
```
打印出的结果数据：
```
{'code': 200, 'msg': '请求成功', 'url': 'n8DB0IMAq13Am1RbWthn17mGLk1OnfZv2xHDeWUzc2QlBpQ1dkmNAgpC3FBy8bQCV4jb47ZB3uvKp2Eq8hVrHnrF+jQkGkTf9HOId/3bJzgPwfXKn4h2g5fVTVJZZHu39+Q5OnoUstVa8w5jLufcp896FJ/FUlxYQa3tWIzOhiZgC8nDaBN/Y+txr2b5u3ZVWnUkM7AB6wBDoDCjhc2OX/jRdBDArq6tOPu6iXb8emtUiNNlcsSggPprYlhLfrXXDGNeqkFjPqIqhkXFWBDHOCoEkMPQ8S91iZm47Q1p95Bk8F2VfcoPqkrsziEq7S91', 'type': 'm3u8', 'success': 1, 'time': '2025-05-25 00:44:01', 'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36 Edg/136.0.0.0', 'ip': '192.168.1.1'} # 为了保护隐私，IP地址已修改
```
一目了然，返回的数据包里面就包含了加密的真实地址，地址类型`type`：`m3u8`。
整理解码流程，发现就差找到`key`和`time`的生成方法，然后再找到解密被加密的真实地址`url`字符串，应该就大功告成了。
即：
- 通过播放详情页目标地址`aHR0cHM6Ly93d3cuZGFtaXlzLmNjL3ZvZHBsYXkvMjItMy0xLmh0bWw=`，使用`request`或`DrissionPage`库的`SessionPage`，`XPath`或正则或`BeautifulSoup`解析器，找到请求的播放地址，再通过`Base64`方法解密 ->URL反编码成明文（比如上述某TX的https://v点qq点com/x/cover/XXX.html
- 把请求播放地址https://v点qq点com/x/cover/XXX.htm l与`time`和`key`组成API负载，通过API请求真实播放地址（可下载的地址）数据包。

## 五、寻找解密真实地址`url`的算法或方法
如何寻找？既然真实地址数据包是由`https://jiexi.shipinbofang.net:7898/api.php` 请求的发送的，那就看看它的上一级发起进程，即“发起程序”是什么。
在“发起程序”请求调用堆栈里面，看到了这个匿名的栈方法，点击进去。
![](https://attach.52pojie.cn/forum/202505/25/024841yzl73fm7f4fclsl4.jpg)
匿用.jpg (79.92 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTE4NHxmOTYxNzY0OHwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE4NHxmOTYxNzY0OHwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
匿用
2025-5-25 02:48 上传
真是太流畅了，基本没有做防护混淆的机制，直接就看到了`key`和`time`。
![](https://attach.52pojie.cn/forum/202505/25/030453xqo9c1t1odt1r9m7.jpg)
key.jpg (104.66 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTE5NXw0M2EyMmRkNXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTE5NXw0M2EyMmRkNXwxNzkzNTcwOTQ2fDE2ODA4NnwyMDMzOTI3&nothumb=yes)
key
2025-5-25 03:04 上传
而且发现它的请求地址是`https://jiexi.shipinbofang.net:7898/?url=`[//v点qq点com/x/cover/XXX.html" target="_blank" rel="noopener noreferrer nofollow"](https://v点qq点com/x/cover/XXX.html:rel="nofollow noopener" href="https)。
