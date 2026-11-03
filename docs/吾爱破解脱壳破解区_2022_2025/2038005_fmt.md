
# [Web逆向] [原创] 网洛者 `AAEncode` 加密 - 初体验 逆向分析

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=2038005](https://www.52pojie.cn/forum.php?mod=viewthread&tid=2038005)
- 抓取时间：2026-11-01T22:08:16

本次分析网站为 aHR0cHM6Ly93YW5nbHVvemhlLmNvbS9jaGFsbGVuZ2UvMw==

![image.png](https://attach.52pojie.cn/forum/202506/11/112051bjtas77ar7ustvsr.png)

`image.png` (60.39 KB, 下载次数: 1) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjA5NXwzZDUzMWU0Y3wxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 11:20 上传

## 开撸!!!

打开网站打开 F12 抓包，然后进行翻页。发现请求参数 `_signature` 是加密参数，然后直接全局搜索 `_signature`。

![image.png](https://attach.52pojie.cn/forum/202506/11/112633x8liqqmsu8fvqjw3.png)

`image.png` (17.87 KB, 下载次数: 1) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjA5N3wwMDkyZTBmOXwxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 11:26 上传

发现一个名为 `get_sign` 的函数入口，应该就是加密函数了。

![image.png](https://attach.52pojie.cn/forum/202506/11/112821pssmjm26qmmmcmm7.png)

`image.png` (14.5 KB, 下载次数: 1) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjA5OHw4NDUzNDI0YnwxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 11:28 上传

进去之后发现一大堆进行 `AAEncode` 的代码，处理方法也很简单，复制下来删除最后一对括号然后在 `console` 运行一下即可，`JJEncode` 处理方法是一样的。

处理完之后发现一堆代码就只有这一部分的代码是有用的，他首先以时间戳为 `key`，构建了一个 `encrypted` 对象，然后再用 `message + "|" + 13位时间戳` 的字符串进行 `DES` 加密去，`message` 为请求的链接。

![image.png](https://attach.52pojie.cn/forum/202506/11/113526ewy4b41b8xexjzhx.png)

`image.png` (133.16 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjA5OXxhZGZmODMyNnwxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 11:35 上传

## 直接撸代码

先导入包然后构建 `crypto` 对象，`key` 那边切片是因为 `des` 的 `key` 是八位数的所以要切片。

```python
from Crypto.Cipher import DES
from Crypto.Util.Padding import pad

_time = int(time.time()) * 1000
crypto = DES.new(key=str(_time)[:8].encode(), mode=DES.MODE_ECB)
```

然后进行加密对比一下跟网站的结果是否一样。

```python
_signature = crypto.encrypt(pad(f'要加密的内容'.encode(), DES.block_size)).hex()
```

![image.png](https://attach.52pojie.cn/forum/202506/11/114741qhy0f8nzzl6ryiy6.png)

`image.png` (15.55 KB, 下载次数: 1) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjExNXwyYjQ5MDg3MXwxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 11:47 上传

![image.png](https://attach.52pojie.cn/forum/202506/11/114803a0lwxb0t9w0kd25t.png)

`image.png` (9.06 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjExNnw4ZGZiOGUwNHwxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 11:48 上传

结果一样就行了。

## 打完收工!!!!

核心代码已经贴出来了，其余代码自己撸哦。

![image.png](https://attach.52pojie.cn/forum/202506/11/122630gej7e3yecrpanore.png)

`image.png` (40.82 KB, 下载次数: 1) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4MjEyOHw4YmRkODI0MXwxNzkzNTcwODk0fDE2ODA4NnwyMDM4MDA1&nothumb=yes) 2025-6-11 12:26 上传
