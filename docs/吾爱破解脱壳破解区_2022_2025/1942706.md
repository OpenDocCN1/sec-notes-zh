
# [Web逆向] 某供水公司请求签名算法分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1942706
- 抓取时间：2026-11-01T22:27:38

## 1、抓包获取接口

手机上开启 `http catcher`，微信上打开日用水量，返回 `http catcher`，找到 `/api/waterdrip/Pay/BillSubDetailWx` 接口，复制 `Raw` 到电脑。

![](https://attach.52pojie.cn/forum/202407/11/091106j6ekdtevvlsrtkds.png)

`image.png` (98.02 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjcwNzYxMHw0NjY0ZGY0ZHwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNzYxMHw0NjY0ZGY0ZHwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes)

2024-7-11 09:11 上传

```
POST https://【已隐藏】/api/waterdrip/Pay/BillSubDetailWx HTTP/1.1
x-tif-loginUserid: 【已隐藏】
Sec-Fetch-Site: same-site
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
x-tif-appid: 【已隐藏】
x-tif-nonce: 【已隐藏】
x-tif-token: 【已隐藏】
x-tif-timestamp: 【已隐藏】
x-tif-sign: 【已隐藏】
Accept: application/json, text/plain, */*
Content-Type: application/json
User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 17_4_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E148 MicroMessenger/8.0.49(0x18003137) NetType/WIFI Language/en
Referer: 【已隐藏】
Host: 【已隐藏】
Content-Length: 61
Expect: 100-continue
Connection: Keep-Alive

{"ksrq":"2024-06-09","jsrq":"2024-07-09","jsdbh":"【已隐藏】"}
```

## 2、浏览器打开地址

从手机端复制地址，在浏览器打开。

![](https://attach.52pojie.cn/forum/202407/10/150120flvzfcfrodlpwy47.png)

`image.png` (155.73 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcwNzM1OHwxZWU5ZDJkNXwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNzM1OHwxZWU5ZDJkNXwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes)

2024-7-10 15:01 上传

## 3、初部分析算法

如下图，基本可得出算法为：`Object(c["a"])(C + g + _ + C)`

![](https://attach.52pojie.cn/forum/202407/10/151623j1vzbbuxjlviulit.png)

`image.png` (244.16 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcwNzM2NnwyNjE5ODE3YXwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNzM2NnwyNjE5ODE3YXwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes)

2024-7-10 15:16 上传

## 4、问 GPT，得出具体算法

![](https://attach.52pojie.cn/forum/202407/10/152111s3r161rfjxkjjzrx.png)

`image.png` (151.68 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcwNzM4N3xjYTBiNTg5ZnwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwNzM4N3xjYTBiNTg5ZnwxNzkzNTcyMDU2fDE2ODA4NnwxOTQyNzA2&nothumb=yes)

2024-7-10 15:21 上传

`token` 是根据是从 `POST /BasicsApi/Weixin/BaseCallback` 接口动态获取，`http catcher` 中搜索 `x-tif-token` 的值，懒得写过程了。
