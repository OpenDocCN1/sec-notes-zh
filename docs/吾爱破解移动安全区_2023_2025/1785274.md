
# [Android 原创] 某查查`sign`和`access_token`分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1785274
- 抓取时间: 2026-11-01T21:43:10

## 一、抓包查看数据

1、在第一次打开app的时候，会自动生成一个`deviceId`、`sign`。并且服务器返回对应的`access_token`

2、多次清除app数据，重新打开app抓包。

①`deviceId`是会变化的，可以猜想是随机生成

②`sign`的格式均为32位数字后拼接`||d`,看起来像一个md5值

## 二、Jadx定位

1、通过地址中的关键字`/getAccessToken`进行搜索，可以成功定位到

![](https://attach.52pojie.cn/forum/202305/14/124752kxr2a4oz42bvx1jf.png)

[`2.png` (29.55 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYwMHw1MmJmOThjZnwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 12:47 上传

2、其中参数2是我们要找的`deviceId`、参数8是`sign`值。向上查找调用源头

![](https://attach.52pojie.cn/forum/202305/14/125426v9w9kdcewll3z9kv.png)

[`4.png` (23.17 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYwMnw2MWJlOTg3MXwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 12:54 上传

3、其中：`bIv`就是`deviceId`、`sign`是由一个方法的返回值，其中参数1：`deviceId`、参数2：毫秒时间戳、参数三：向上查找发现如下是一个固定值：`3b09g2ae9c34bb5af5d37d7c13ed033f645f50a4d`

![](https://attach.52pojie.cn/forum/202305/14/130314cqn1vao4ko89kkva.png)

[`5.png` (56.91 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYwM3wxZGExYzVhOXwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:03 上传

## 三、加密方法定位

参数分析完毕后，我们继续向上寻找这个方法的调用源头：

1、发现是一个抽象的方法

![](https://attach.52pojie.cn/forum/202305/14/130608rk6kje6zijarur9j.png)

[`6.png` (16.68 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYwNHxjYjY4NmUzMXwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:06 上传

2、我们继续分析，可以看到，有一个是叫`EncryptImpl`的类实现了这个方法，比较可疑

![](https://attach.52pojie.cn/forum/202305/14/130909vwciydiihiaphaae.png)

[`7.png` (50.13 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYwN3xkMzBhZWE0Y3wxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:09 上传

3、进入后发现是一个`MD5`加密，并且有日志输出，我们hook他的参数部分

![](https://attach.52pojie.cn/forum/202305/14/132419pz3dh4a9al2799u9.png)

[`8.png` (40.9 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYxMHxlNDQxYzU5ZXwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:24 上传

（`deviceId + 毫秒时间戳 + 3b09g2ae9c34bb5af5d37d7c13ed033f645f50a4d`）

![](https://attach.52pojie.cn/forum/202305/14/132118d18f4i48nize0zge.png)

[`9.png` (6.58 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYwOXxjYWJjNzRjYXwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:21 上传

md5验证后，也是正确的，没有魔改

![](https://attach.52pojie.cn/forum/202305/14/135119hsbzdim1pizvu9di.png)

[`666.png` (77.88 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYyN3wyNDFhMDMzNnwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:51 上传

## 四、模拟请求

我们用`python`生成代码并且请求。其中设备ID随机生成即可。成功拿到`access_token`

![](https://attach.52pojie.cn/forum/202305/14/133249ceg0ts9astszv1ag.png)

[`13.png` (52.96 KB, 下载次数: 2)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNDYxOHwyYzRhYTVjZnwxNzkzNTY5Mzg4fDE2ODA4NnwxNzg1Mjc0&nothumb=yes)

2023-5-14 13:32 上传
