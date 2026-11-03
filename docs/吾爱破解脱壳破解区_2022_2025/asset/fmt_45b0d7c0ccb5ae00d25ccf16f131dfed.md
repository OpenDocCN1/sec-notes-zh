
# [原创] Fiddler Everywhere 5.7.1永久试用版

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1911989
- 抓取时间: 2026-11-01T22:33:25

根据4.6.2的思路（fiddler everywhere 4.6.2 去除授权校验 - 『[//www.52pojie.cn/forum-5-1.html](https://www.52pojie.cn/forum-5-1.html) [//www.52pojie.cn](https://www.52pojie.cn)区』 - [//www.52pojie.cn](https://www.52pojie.cn) - LCG - LSG |安卓破解|[//www.52pojie.cn/forum-32-1.html](https://www.52pojie.cn/forum-32-1.html)|www.52pojie.cn），将5.7.1的试用天数改为999999

![](https://attach.52pojie.cn/forum/202404/10/135643wmdcmmj3tm1jjmdf.png)  
`1.png` (192.34 KB, 下载次数: 2)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODUzM3w3MjcxNzVhMHwxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 13:56 上传

## 一、控制台启动

这部分和4.6.2的差别不大

主要做两件事，一是去除`Fiddler.WebUi.dll`中的对文件篡改的校验，二是去除`Fiddler.WebUi.dll`文件自身被篡改的校验

虽然修改的地方变了，但是调试方式是一致的，所以过程就不赘述了，大家可以直接参考4.6.2的，这里直接贴出修改点

### 1. 文件篡改校验修改点

![](https://attach.52pojie.cn/forum/202404/10/140748rwcpoygmmokmg6zf.jpg)  
`2.jpg` (70.04 KB, 下载次数: 1)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODU0MHxlMzMyMjVkOXwxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 14:07 上传

从下面的参数可以看到，`A_0`和`A_1`就是要进行对比的两个`Hash`值，通过修改二进制，直接将它变成`A_0`与`A_0`的对比

![](https://attach.52pojie.cn/forum/202404/10/140950uipiv1u86uih0yyz.png)  
`3.png` (24.95 KB, 下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODU0MXw2ZGJjYmM3Y3wxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 14:09 上传

### 2. `Fiddler.WebUi.dll`自篡改校验修改点

![](https://attach.52pojie.cn/forum/202404/10/141135vzk8pfc97o8kz9fm.png)  
`4.png` (34.56 KB, 下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODU0MnwxYzM3YjNhOXwxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 14:11 上传

这里粗暴点，直接将它变成`null == null`，即永远返回`true`

![](https://attach.52pojie.cn/forum/202404/10/141331vqsuy0hfeundm330.png)  
`5.png` (31.49 KB, 下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODU0NHw2YjdkZmRjM3wxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 14:13 上传

![](https://attach.52pojie.cn/forum/202404/10/141357a0z5roos3gnanqqk.png)  
`6.png` (15.82 KB, 下载次数: 2)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODU0Nnw1NDYxYzk5N3wxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 14:13 上传

改完之后成功开启控制台

![](https://attach.52pojie.cn/forum/202404/10/141957j950ld9x0jja9dxo.png)  
`7.png` (271.26 KB, 下载次数: 1)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODU0N3w3ZDU3N2MwM3wxNzkzNTcyNDAyfDE2ODA4NnwxOTExOTg5&nothumb=yes)  
2024-4-10 14:19 上传

## 二、代{过}{滤}理服务器

4.6.2中，接口数据没有详细说是怎么抓的，在处理5.7.1过程中，这部分自己摸索了一遍

首先，在Network中可以看到，是有两个接口的，分别是`/versions`和`/users`，这两个接口也可以看到一些关于试用时间的相关数据，只不过伪造了这两个接口后并不能进入到首页页面

而后在调试过程中，发现有些接口是直接从后端，也就是`dll`文件中直接发起的，这部分接口控制台无法抓取，好在后端发起的接口的域名部分，依旧是前端`main.*.js`中控制的，所以就有了个思路：起个代{过}{滤}理服务，将域名改成代{过}{滤}理服务地址，也就是`127.0.0.1:port`，代{过}{滤}理中将其转到真实地址，期间输出相关返回信息，用以构造接口数据

代{过}{滤}理转发可以继续使用`node.js`，不过要注意，如果需要引入非node自身的依赖包，需要手动到`npm`资源站下载js版依赖（ts的es6语法会引发报错），下载后`install`下，通过绝对路径进行引入，方便起见，本人使用了`flask`，通过设置请求拦截器，转发并输出响应信息

[Python] 纯文本查看 复制代码
