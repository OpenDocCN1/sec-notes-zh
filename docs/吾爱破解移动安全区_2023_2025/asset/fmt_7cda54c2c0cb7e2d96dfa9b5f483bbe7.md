
## Objection 常用命令

### ③ 列出内存中模块

我们只关心自己的 app（containners）中的模块，其他是系统或组件模块，而且这里也可以直接看到我们应用的基址。

```
memory list modules
```

![](https://attach.52pojie.cn/forum/202305/09/143149xtfw8bbstfqukw7o.png)

43.png (325.03 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYxMzI3MHw2OTk2NTQ4NXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3MHw2OTk2NTQ4NXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:31 上传

### ④ 列出导出表

这里可以找到 `do_it` 地址，拿这个地址去减去基址（包模块地址）得到函数相对地址。`ida64 UnCrackable1.ipa`，用 `ida` 看一下，发现 `do_it` 函数是基址+相对地址。

```
memory list exports [包名]
```

![](https://attach.52pojie.cn/forum/202305/09/143153xoqz20aav4c2qe1x.png)

44.png (240.11 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI3MXw0OWU3YTQxZXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3MXw0OWU3YTQxZXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:31 上传

### ⑤ 获取当前应用存储的 cookies

```
ios cookies get --json
```

### ⑥ 在堆上找一个类的实例化内存对象把内存地址打印出来

```
iOS heap search instances UILabel
```

![](https://attach.52pojie.cn/forum/202305/09/143155h788jk45v58gkg0j.png)

45.png (50.95 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYxMzI3M3xmNjkxYTkxY3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3M3xmNjkxYTkxY3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:31 上传

### ⑦ 打印域中变量值

这里可以得到结果。

```
ios heap execute 地址 text --return-string
```

![](https://attach.52pojie.cn/forum/202305/09/143157p3rtoo8ioiz3twfs.png)

46.png (32.63 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI3NHxjMGU1NzI0MnwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3NHxjMGU1NzI0MnwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:31 上传

```
ios heap evaluate --inline
```

自己写 OC 代码。

### ⑧ 查看二进制文件信息，查看是否加密判断是否加壳

```
ios info binary
```

![](https://attach.52pojie.cn/forum/202305/09/143159k3yea92x5pxayp98.png)

47.png (27.87 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI3NXxiNzYzNTBkMnwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3NXxiNzYzNTBkMnwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:31 上传

### ⑨ `dump` 存储的密码

```
ios keychain
```

### ⑩ `hook` 一些加解密算法的库

```
ios monitor crypto
```

### ⑪ 查看是否有凭据存储

```
ios nsurlcredentialstorage
```

### ⑫ 查看用户的一些信息

```
ios nsuserdefaults
```

用户的一些信息。

![](https://attach.52pojie.cn/forum/202305/09/143202bhnf7i1il786efgz.png)

48.png (301.72 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI3N3xkMTBkNGIwNHwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3N3xkMTBkNGIwNHwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:32 上传

`hook` 证书绑定相关方法。

```
ios sslpinning
```

![](https://attach.52pojie.cn/forum/202305/09/143205x0x0ddaum513lq3x.png)

49.png (73.85 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI3OHwyOTBjYjgxY3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3OHwyOTBjYjgxY3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:32 上传

```
ios ui screenshot 123.png
```

做个截图。

![](https://attach.52pojie.cn/forum/202305/09/143207u1otemowotmft2s5.png)

50.png (75.49 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYxMzI3OXwwNTE5NzU2ZXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI3OXwwNTE5NzU2ZXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:32 上传

### ⑬ `hook` 相关

列出所有类。

```
ios hook list classes
```

![](https://attach.52pojie.cn/forum/202305/09/143329jkkiky9k8v1yz73k.png)

51.png (292.68 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYxMzI4MHxjN2EyMGZjY3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4MHxjN2EyMGZjY3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

列出相关类的所有方法。

```
ios hooking list class_methods ViewController
```

![](https://attach.52pojie.cn/forum/202305/09/143333l3tvvdvvtwvl35vd.png)

52.png (107.41 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYxMzI4MXw1YTY4MTk2OXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4MXw1YTY4MTk2OXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

`hook` 这个类的所有方法，`hook` 后去点击查看调用。

```
ios hooking watch class ViewController
```

![](https://attach.52pojie.cn/forum/202305/09/143337yjbdlz3ryl4lrdxl.png)

53.png (113.42 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYxMzI4Mnw0M2VjM2M4MXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4Mnw0M2VjM2M4MXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

`hook` 这个 `buttonClick` 方法。这里要注意 OC 中的方法签名要记得加 `:`，完整方法签名是 `-[ViewController buttonClick:]`。

```
ios hooking watch method "-[ViewController buttonClick:]" --dump-args --dump-backtrace --dump-return
```

![](https://attach.52pojie.cn/forum/202305/09/143344w4bzvjjjcj4pmjwi.png)

54.png (197.85 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjYxMzI4M3w0MTUzY2U3YXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4M3w0MTUzY2U3YXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

## 2> 5. Objection破解简单CrackMe

### (1) 首先对CrackMe进行分析

![](https://attach.52pojie.cn/forum/202305/09/143348xzflff717y1y7klp.png)

58.png (62.71 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI4NHxlOGVhN2VhM3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4NHxlOGVhN2VhM3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

① 找到点击事件响应方法 `buttonClick`，发现其对输入字符串及本地字符串比对。

② 发现本地字符串存放在类属性中，值由 `do_it()` 函数生成。

③ 查看属性类型是 `UILabel`，也就是说是隐藏起来的文本显示控件类对象。

### (2) 进行破解

首先注入该 CrackMe 进程。

```
frida-ps -U
objection -g PID explore
```

① 直接 `dump` 当前 ui 界面所有对象，可以看到设置为隐藏的 `UILabel` 对象及其 `text` 值。

```
ios ui dump dump
```

![](https://attach.52pojie.cn/forum/202305/09/143351fgnxqxoq557blqg7.png)

59.png (96.82 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMzI4NXwyZDhmYzhkM3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4NXwyZDhmYzhkM3wxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

② 直接找到所有 `UILabel` 类对象，对其进行打印，打印其域内属性及对象。

```
ios heap search instances UILabel
```

![](https://attach.52pojie.cn/forum/202305/09/143358csfy99ajs1sf1z9k.png)

61.png (51.12 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxMzI4N3wxZWMwMGE5NXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4N3wxZWMwMGE5NXwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

```
ios heap print ivars 0x101311270 --to-utf8
```

![](https://attach.52pojie.cn/forum/202305/09/143354lcnftdyaqnpdnnpo.png)

60.png (44.59 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjYxMzI4NnxhZWQ0ZDA2MnwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMzI4NnxhZWQ0ZDA2MnwxNzkzNTY5MzkzfDE2ODA4NnwxNzgzMjU2&nothumb=yes)

2023-5-9 14:33 上传

《挑战不用macOS逆向iOS APP》系列的第二课基础知识讲解内容到这就结束了，这里的内容可能并不全面，后续会根据需要进行补充，同时也会继续更新iOS App逆向学习内容，共同学习进步。
