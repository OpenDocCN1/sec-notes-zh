
# [Web逆向] 微软tts 下载音频按钮（tampermonkey脚本）的实现思路

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1606432
- 抓取时间: 2026-11-01T23:32:16

## 微软tts 下载音频按钮（tampermonkey脚本）的实现思路

### 目标网站

```
https://azure.microsoft.com/zh-cn/services/cognitive-services/text-to-speech/#features
```

- 目的是解决微软官方的网页版demo，不能直接下载转换后的MP3文件

### 效果

![](https://attach.52pojie.cn/forum/202203/19/103253i2et3h33fwzp53gf.png)

image-20220319101032760.png (132.54 KB, 下载次数: 254)

[href="forum.php?mod=attachment&aid=MjUwNTAwM3w0M2FlZGJmM3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAwM3w0M2FlZGJmM3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:32 上传

### 3> 适合阅读的人群

- 有一定的JS逆向基础的人群
- 会使用chrome开发者工具进行抓包调试和断点调试
- 如果只是想要实现微软tts 下载音频可以直接安装`tampermonkey`扩展后安装脚本即可,安装完成后会在声音合成后出现下载音频的按钮，脚本地址`https://greasyfork.org/zh-CN/scripts/441531-%E5%BE%AE%E8%BD%AFtts-%E4%B8%8B%E8%BD%BD%E6%8C%89%E9%92%AE`

### 3> 理解本篇教程你将会对以下知识点有更深刻的理解

- `tampermokey` 脚本的注入时机，需要尽可能早的注入时使用`@run-at` `document-start`
- `hook` 的简单使用
- 函数或变量的导出，比如导出到`window`方便后续使用，JS扣代码常用
- 使用`tampermokey` 脚本 下载二进制文件的方法
- `websocket`的简单了解

### 3> 0x01

首先在目标网站`https://azure.microsoft.com/zh-cn/services/cognitive-services/text-to-speech/#features`点击几次播放抓几次包发现以下的请求是用来传输合成的音频的

```
wss://eastus.tts.speech.microsoft.com/cognitiveservices/websocket/v1?Authorization=bearer%20eyJhbGciOiJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGRzaWctbW9yZSNobWFjLXNoYTI1NiIsInR5cCI6IkpXVCJ9.eyJyZWdpb24iOiJlYXN0dXMiLCJzdWJzY3JpcHRpb24taWQiOiI2MWIxODBlMmJkOGU0YWI2OGNiNmQxN2UxOWE5NjAwMiIsInByb2R1Y3QtaWQiOiJTcGVlY2hTZXJ2aWNlcy5TMCIsImNvZ25pdGl2ZS1zZXJ2aWNlcy1lbmRwb2ludCI6Imh0dHBzOi8vYXBpLmNvZ25pdGl2ZS5taWNyb3NvZnQuY29tL2ludGVybmFsL3YxLjAvIiwiYXp1cmUtcmVzb3VyY2UtaWQiOiIvc3Vic2NyaXB0aW9ucy9jMjU1ZGYzNi05NzRjLTQ2MGEtODMwYi0yNTE2NTEzYWNlYjIvcmVzb3VyY2VHcm91cHMvY3MtY29nbml0aXZlc2VydmljZXMtcHJvZC13dXMyL3Byb3ZpZGVycy9NaWNyb3NvZnQuQ29nbml0aXZlU2VydmljZXMvYWNjb3VudHMvYWNvbS1zcGVlY2gtcHJvZC1lYXN0dXMiLCJzY29wZSI6InNwZWVjaHNlcnZpY2VzIiwiYXVkIjoidXJuOm1zLnNwZWVjaHNlcnZpY2VzLmVhc3R1cyIsImV4cCI6MTY0NzU5NDk1MCwiaXNzIjoidXJuOm1zLmNvZ25pdGl2ZXNlcnZpY2VzIn0.PjYuGtrhh9RgkaH3iPYwPxBwtUDaWCjIBT_iQbPJf7g&X-ConnectionId=F4D719576F88485997E628643E905B76
```

有两个参数`Authorization`和`X-ConnectionId`前者在网页的源码中，后者是`uuid4`，因为我们用`tampermonkey`脚本所以不需要搞定这两个参数的生成

![](https://attach.52pojie.cn/forum/202203/19/103308xesaieaevvirtrxi.png)

image-20220318171347035.png (255.31 KB, 下载次数: 253)

[href="forum.php?mod=attachment&aid=MjUwNTAwNHw0ZjM2MzJlOHwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAwNHw0ZjM2MzJlOHwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:33 上传

这个还是`get`请求，但该请求后改变传输协议为`websockets`,发送需要转换的文本后，服务器返回了一堆Binary Message（二进制消息），这些就是合成的声音。

![](https://attach.52pojie.cn/forum/202203/19/103326h7jk7q777n6ukl22.png)

image-20220318171615746.png (305.14 KB, 下载次数: 253)

[href="forum.php?mod=attachment&aid=MjUwNTAwNXwyOGQ4M2E2NXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAwNXwyOGQ4M2E2NXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:33 上传

### 3> 0x02

因为网页版会对合成的声音进行播放，所以前端肯定有对这些数据进行处理，我们只需要拿到前端处理完成的二进制文件就可以了。那问题就是处理后的二进制文件存哪里了？

可以进行简单的断点调试，没有思路就直接这个进去下一个断点，一步步往上找找看。

![](https://attach.52pojie.cn/forum/202203/19/103336drk01k5xeje51xew.png)

image-20220318172825273.png (331.14 KB, 下载次数: 254)

[href="forum.php?mod=attachment&aid=MjUwNTAwNnxjYTM1NTQxMXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAwNnxjYTM1NTQxMXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:33 上传

在这里可找到`websocket`的`onmessage`

![](https://attach.52pojie.cn/forum/202203/19/103356oi4fbijf0jnrr0nf.png)

image-20220318204848429.png (380.56 KB, 下载次数: 269)

[href="forum.php?mod=attachment&aid=MjUwNTAwN3xiNjY5NmVlYnwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAwN3xiNjY5NmVlYnwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:33 上传

这里`l.provWebsocketClientWebsocket`的一个实例

![](https://attach.52pojie.cn/forum/202203/19/103415hot9mfsouloil6ow.png)

image-20220318205340359.png (143.16 KB, 下载次数: 261)

[href="forum.php?mod=attachment&aid=MjUwNTAwOHwyZmJjYWQwYXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAwOHwyZmJjYWQwYXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:34 上传

继续往上追一层找到了每一次处理Binary消息的地方。

![](https://attach.52pojie.cn/forum/202203/19/103449tz6a0khk1uo1ih6i.png)

image-20220318204950954.png (338.59 KB, 下载次数: 257)

[href="forum.php?mod=attachment&aid=MjUwNTAxMnw4OTc1MzU4OHwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxMnw4OTc1MzU4OHwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:34 上传

### 3> 0x03

但是这么找很难找到最后`binary`传输完成后的完整音频数据。

这里再换一个方法，可以看到`websocket`通讯的最后有一个含有`Path:turn.end`的包，尝试在上面同一个`js`文件中搜一下 `turn.end`然后下断点。

![](https://attach.52pojie.cn/forum/202203/19/103504n1s6p168zsbpm87z.png)

image-20220318215256773.png (332.62 KB, 下载次数: 261)

[href="forum.php?mod=attachment&aid=MjUwNTAxNHwyOWZlZjAwMnwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxNHwyOWZlZjAwMnwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:35 上传

可以发现定到了这里

![](https://attach.52pojie.cn/forum/202203/19/103519te64v5xzpo5x5phz.png)

image-20220318220501114.png (299.18 KB, 下载次数: 270)

[href="forum.php?mod=attachment&aid=MjUwNTAxNXw5YmVjYTM1MnwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxNXw5YmVjYTM1MnwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:35 上传

这里好多`case` 一开始以为是控制流平坦化，但并不是，而且变量名是有意义的，稍微仔细一点就可以看到这样一个`synthesisCompleted`。

![](https://attach.52pojie.cn/forum/202203/19/103532xiz73zuy440y0j33.png)

image-20220318220913937.png (275.62 KB, 下载次数: 256)

[href="forum.php?mod=attachment&aid=MjUwNTAxNnwzZWQ4OGI0YXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxNnwzZWQ4OGI0YXwxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:35 上传

而且有一个`if`的判断，`synthesisCompleted`翻译过来是合成完成，打一个断点，确实是在`websockets`传输完成后会断住，里面的`g.privAudioData`猜测就是合成的`mp3`。

![](https://attach.52pojie.cn/forum/202203/19/103546xfjuinjixjg48qh9.png)

image-20220318221113258.png (282.44 KB, 下载次数: 259)

[href="forum.php?mod=attachment&aid=MjUwNTAxN3wxNjlmZDUyM3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwNTAxN3wxNjlmZDUyM3wxNzkzNTc1OTMzfDE2ODA4NnwxNjA2NDMy&nothumb=yes)

2022-3-19 10:35 上传

做一下实验，把这个二进制数据下载下载听一下，发现确实就是我们要的`mp3`
