
# [Web逆向] 某迪汽车品牌小程序逆向

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1934663
- 抓取时间: 2026-11-01T22:28:48

## 零、前言

这两天看到了一位大佬发了两篇小程序的逆向帖子

https://www.52pojie.cn/thread-1934354-1-1.html

https://www.52pojie.cn/thread-1934003-1-1.html

手痒想试一试，刚好想做某迪汽车品牌的自动签到，就盯向了他！！！

## 一、工具准备

- https://reqable.com/zh-CN/ （抓包工具）
- `unveilr` （解包工具）
- `vscode`
- `Wechat` 3.9.11.15

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718348330002_3b7398b475459.png)

## 二、分析请求

### 1、配置抓包软件

抓包软件的配置见 https://reqable.com/zh-CN/docs/getting-started

### 2、启动软件抓包

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718348275447_3f3c10b03d47b.png)

### 3、启动小程序并抓包

我这边微信版本为3.9.11.15，应该是最新版的

请求可能会比较多，我们可以在控制台筛选只显示微信的请求

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718348433841_d23af70b080bd.png)

## 三、分析目标

### 1、目标一：返回值

启动小程序后就会看到很多的请求，随便点击一个即可

我们可以看到他的返回值是加密的

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718348522011_6dd6ea0e8e92e.png)

这种返回值后面有两个等号，一般是base64或者AES，我这边测试了base64不可以，那么就只能进行逆向分析了

### 2、目标二：请求头

查看请求头发现有几个请求头是非标准http协议的请求头，那么大概率也是需要逆向的，因为这几个请求头每一个请求都有！！！

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718348749076_76d010d295007.png)

`x-clienttraceid`：暂时不知道是什么，但是是一个什么id，可能是固定值
`Nonce`：暂时不知道是什么
`Curtime`：很明显的时间戳
`Checksum`：暂时不知道是什么，不过很像md5之类的摘要算法生成的

目标确定后，开始动手！！！！！！！

## 四、解包

### 1、找到wxapkg文件

在pc微信上将目标小程序打开之后，尽量把所有能点的页面都先点一遍，防止部分小程序存在分包的情况导致未能完全加载(直接复制大佬的话，我肯定没有偷懒[认真脸])

大佬发的文章中`wxapkg`文件是在微信的保存文件下，我这边不知道是版本问题还是设置问题，我这边是在电脑的文档目录下

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349042978_70a8e6fd6a08e.png)

我担心不好确定小程序的id，就直接全部删除了，重新加载后出现的肯定是我想要的

随便点了几下后的结果，其实每个接口都需要的内容，不点应该没啥问题(纯属推测，没试过)

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349145335_e4c8082271d7e.png)

### 2、使用unveilr解包

执行命令：

```
unveilr.exe "D:\Backup\Documents\WeChat Files\Applet\wxf62054ec313d6f53\58"
```

`D:\Backup\Documents\WeChat Files\Applet\wxf62054ec313d6f53\58`是我这边小程序的目录，自己替换为自己本地目录，路径一定要用双引号引起来！

执行完成：

命令执行后的结果

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349324997_d5d6827d2f664.png)

文件输出：

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349355641_e82b87e550e89.png)

## 五、开始分析

直接使用`vscode`打开项目(因为这个项目不需要跑起来，所以就不需要微信开发工具)

### 一、返回值解密

#### 1、找到关键代码

因为我们前文已经说明了，返回值可能是base64或者AES，base64的可能被我们暂时pass掉了，所以先搜索AES碰碰运气

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349531710_96d29c24e1e68.png)

看这个文件名，我感觉我运气还是很不错的！！！

点进来后代码是压缩过的，直接找个网页格式化一下(我试了`vscode`的格式化，效果不太行。。。)

格式化后的代码：

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349646384_2f337e6d5fc0e.png)

代码结构如此清晰，位置如此正确，我感觉没必要写下去了！！！(不是)

#### 2、分析解密代码

根据函数名，我们直接看解密的函数

```
exports.aesDecrypt = function(e) {
    var r = t.AES.decrypt({
        ciphertext: t.enc.Base64.parse(e)
    }, a, {
        iv: n,
        mode: t.mode.CBC,
        padding: t.pad.Pkcs7
    }).toString(t.enc.Utf8);
    return JSON.parse(r)
}
```

代码太清晰了，真不知道说啥了。。。

加密模式是`CBC`，填充方法是`pkcs7`，接下来找到偏移量和密钥就可以了

#### 3、找到密钥和偏移量

密钥是变量`a`，偏移量是`n`，可以直接在下面看到对应的代码。。。

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349885727_19883089bb1c9.png)

根据代码，我们去`constant-obfuscated.js`文件里面继续找

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349950259_e4ddd772397e8.png)

同样先把代码格式化一下，方便我们观看

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718349992655_f37e5cf6ffb8f.png)

1、找密钥

搜索`aes_key`，可以搜索到两个结果，

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718350052853_c658fa1013d1c.png)

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718350060341_d893e3b5bdb02.png)

第一个看着不太像。。。先试暂时确定为第二个

2、找偏移量

我们发现`aes_iv`是一个函数的运行结果！可以大展拳脚了！！

然后发现没啥可扣的。。。

![](https://pic2.ziyuan.wang/user/2513002960/2024/06/1718350193340_60cabd823c6a2.png)

感兴趣的可以去试一试，我这边直接说结果，偏移量为：`uDEHPGzSIHIWBlNT`

#### 4、测试一下

返回值：
