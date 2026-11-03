
# [Web逆向] 保姆式解析某高质量听书网站音频地址全过程(适合小白练手)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2035137
- 抓取时间: 2026-11-01T22:08:56

## 免责声明：

本贴中所有内容仅供学习交流使用，不用于其他任何目的，不提供完整代码，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关。

### 一、前言

要获取网络上免费站的音视频真实地址的方法其实有很多，比如直接用`IDman`插件嗅探，或者抓包缓存等。
但是！！！这只是单个音频地址的获取方法，而且不一定有效，比如音频地址使用异步动态加载，甚至是加密包装过的，使用嗅探工具不一定能嗅探到。
而且！！！如果想要批量获取，就必须了解这个音频真实地址的产生过程，这个过程就是解析（逆向）其算法过程。
本次解析目标就是一个解析算法的过程。
我也是小白，对JS一窍不通，有错漏之处望海涵。
大佬见笑了。

### 二、目标地址

`aHR0cDovL3d3dy55dWV0aW5nYmEuY24vYm9vay9kZXRhaWwvM2ExODAxNzQtN2I4Yi05ZWVhLWUwNmYtMzk0NmQ5Y2E5NmYwLzA=`

### 三、解析思路

#### 1、通过HTML源码 -> 直接搜索关键词

通过`Ctrl + U`查看HTML源码 `Ctrl + F`搜索关键词 `.mp3` 或 `.m4a` 找到明文地址，如以下这种：

>
![](https://attach.52pojie.cn/forum/202505/29/155436vqtdmdf9ftsh1qw6.jpg)
m4a明文.jpg (49.57 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTg4NHxlZTkyNWFiNXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTg4NHxlZTkyNWFiNXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
m4a明文
2025-5-29 15:54 上传

如果搜索关键词 `.mp3` 或 `.m4a` 没能找到，则试着搜索 `var now` 或 `var next`，找到以下这种，一般都是被加密或Base64转码了。
>
![](https://attach.52pojie.cn/forum/202505/29/155506lj35ij3juimjcbjj.jpg)
星号.jpg (42.82 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc3OTg4N3wyZWJhODE1OXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTg4N3wyZWJhODE1OXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
星号
2025-5-29 15:55 上传

如果是这种加“*”号的，可以使用这个方法解析获得明文地址：
```javascript
// 可自行转成Python的方法;
function FonHen_JieMa(u) {
    var a = u.split("*");
    var b = '';
    for (var i = 1, n = a.length; i 
```

#### 2、找到对应API接口分析

如果在HTML源码并没有找到疑似加密或明文的音频真实地址，大概率就是通过API接口请求了。
有些通过API接口可以直接返回明文真实地址，比如某听书网：
>
![](https://attach.52pojie.cn/forum/202505/29/155402xrjjqsjlrsryqgjh.jpg)
明文api.jpg (36.25 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTg4MHxmMTYxMjA5MnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTg4MHxmMTYxMjA5MnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
明文api
2025-5-29 15:54 上传

还有一些API接口返回的是经过加密的地址，需要二次解密，比如这一次的yue听巴网：
>
![](https://attach.52pojie.cn/forum/202505/29/160133xir5gti9ig9lcg9x.jpg)
yuetingba api.jpg (45.77 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTg5NHxmMTQ5MDRlZXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTg5NHxmMTQ5MDRlZXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
yuetingba api
2025-5-29 16:01 上传

### 四、开始对目标地址解析全过程

#### 1、发现有防调试机制 -> 可利用脚本破除控制台检测

首先，经过关键词搜索尝试之后，发现这个站点的音频真实地址并没有加载在静态的HTML源码中
然后通过`F12` 或手动打开浏览器开发者工具，发现被秒关闭调试网页，说明该站点有防御控制台打开的反调试机制。
猜测可能是检测控制台被打开然后触发反调试，这个简单，我们可以通过开启“控制台防检测”脚本或工具，以绕过这种反调试机制。
比如这个油猴脚本：`aHR0cHM6Ly9ncmVhc3lmb3JrLm9yZy96aC1DTi9zY3JpcHRzLzUyMzc5Mi3mtY/op4jlmajmjqfliLblj7DpmLLmo4DmtYs=`（需自行将目标站点添加进包含规则@match）
![](https://attach.52pojie.cn/forum/202505/29/163927abptk5kk45tylksg.jpg)
控制台.jpg (63.43 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTkwN3wzYzRhZDExMHwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTkwN3wzYzRhZDExMHwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
控制台
2025-5-29 16:39 上传

#### 2、打开开发者工具 -> 查找API接口

对目标站点开启“控制台防检测”脚本后，接下来就是分析API了。
因为`F12`被禁止，我们可以手动打开浏览器开发者工具。
切换到“网络”标签选项卡 ->重载刷新（`F5`）当前网页 -> 随意点播放一个音频章节 -> 找到可疑的api接口地址“`ting-with-efi`”：
>
![](https://attach.52pojie.cn/forum/202505/29/170014hmkia59g9sql5nig.jpg)
找到API.jpg (171.63 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTkxMXw1NTcxYjY0YnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTkxMXw1NTcxYjY0YnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
ting-with-efi
2025-5-29 17:00 上传

这个API接口地址：`http://www.yue听巴.cn/api/app/docs-listen/3a1801e1-74ec-871b-6d8b-68bce50caf01` 这个ID来确定区分章节的，直接在HTML源代码搜索验证了猜想：
![](https)
id章节来源.jpg (26.9 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTkzM3w4YzI1MjBkMnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTkzM3w4YzI1MjBkMnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
id章节来源
2025-5-29 17:09 上传

而且通过替换这个ID，API返回的是不同的`efi`字段（疑似加密的真实音频地址）。

#### 3、解密真实地址 -> 查找解密函数逻辑链

接下来也就是分析api返回的`efi`字段的值具体是什么了，看看到底是不是音频地址。
在调试工具窗口试着`Ctrl + Shift + F` 全局搜索“`efi`”这个字段，看看它生成逻辑。
BSQUOTE)
大海捞针.jpg (222.76 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTkzOXwwZDcwNzZkN3wxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTkzOXwwZDcwNzZkN3wxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
大海捞针
2025-5-29 17:37 上传

发现搜索结果太多干扰项了，换成跟踪`ting-with-efi`的启动器调用栈，发现都是源自同一个js解析，这说明这个js启动器文件才是关键。
>
![](https://attach.52pojie.cn/forum/202505/29/173246xzzg9p9yakkkcp99.jpg)
启动器.jpg (46.28 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc3OTkzN3xiZDc0YzNiY3wxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTkzN3xiZDc0YzNiY3wxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
启动器
2025-5-29 17:32 上传

很明显，`initGetData`这个调用栈名称是关于初始化数据包的，我们直接从这一个点进去（以后分析推荐直接从启动器入手）。
>
![](https://attach.52pojie.cn/forum/202505/29/173315d6c9c7c5pfubfiun.jpg)
data.jpg (51.02 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTkzOHw2ODRlMDg2MnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTkzOHw2ODRlMDg2MnwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
data
2025-5-29 17:33 上传

太明显了，一眼就看到了相关初始化方法，特别是`initAudio()`，一看就是关于音频初始化的。
>
![](https://attach.52pojie.cn/forum/202505/29/175421r3klrbr1zcick8bh.jpg)
get方法.jpg (96.6 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTk0MXwzMWU1NGY0NHwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTk0MXwzMWU1NGY0NHwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
get
2025-5-29 17:54 上传

从`initAudio()`入手，发现`e.playUrl = o,`这一行，右键加断点然后刷新查看，果然是真实的音频地址，看生成逻辑链，很明显，我们下一步就是要找到`vs`函数：
>
![](https://attach.52pojie.cn/forum/202505/29/180646e1t1o1t3u5k611u3.jpg)
需要找到vs方法.jpg (143.86 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3OTk0MnxhODc3YWMyYXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3OTk0MnxhODc3YWMyYXwxNzkzNTcwOTM0fDE2ODA4NnwyMDM1MTM3&nothumb=yes)
需要找到vs
2025-5-29 18:06 上传

#### 4、解析函数算法-> 找到关键算法逻辑方法

上一步发现`vs`函数才是解码地址的关键，完整的明文地址实际上就是服务器地址 `http://36.5.86.202:50010` 与 `vs.d(e.currentPlayInfo.ef, n, i.padEnd(20, "0"))` 返回的结果进行拼接。
js关键代码片段：
```javascript
const n = e.currentPlayInfo.tingId.replaceAll("-", "")
 , i = e.currentPlayInfo.creationTime.replaceAll("-", "").replaceAll(":", "").replaceAll("T", "").replaceAll(".", "").replaceAll(" ", "")
 , r = vs.d(e.currentPlayInfo.ef, n, i.padEnd(20, "0"))
 , o = e.tingDefaultData.audioServer.url + r;
```
api返回的响应数据：
