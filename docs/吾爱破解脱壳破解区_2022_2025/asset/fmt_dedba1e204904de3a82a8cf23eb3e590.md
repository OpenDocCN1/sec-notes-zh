
![](https://attach.52pojie.cn/forum/202503/21/213811w01yahyhhyrcz1rj.png)

17.png (21.15 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5MXxhMDM1MTViZXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5MXxhMDM1MTViZXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

发现这个方法是加在`k`的原型上的，向上找就能找到`var H = function(){}`

![](https://attach.52pojie.cn/forum/202503/21/213813nhceok7hree4lec7.png)

18.png (20.65 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5MnwxOTk1YThhOHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5MnwxOTk1YThhOHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

把整个`H`拿下来，再把4个方法加上，稍微补一补环境，就能搞定。

![](https://attach.52pojie.cn/forum/202503/21/213815kxm6bccvwmvmf85m.png)

19.png (39.27 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5M3xlMTljYzE0M3wxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5M3xlMTljYzE0M3wxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

![](https://attach.52pojie.cn/forum/202503/21/213817rmnhnq8usjs8omdp.png)

20.png (31 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5NHw2NTE4OWQ4MnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5NHw2NTE4OWQ4MnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

![](https://attach.52pojie.cn/forum/202503/21/213819td91uwazzpsd6nyd.png)

21.png (14.93 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5NXwxZDMxOGY5MHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5NXwxZDMxOGY5MHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

跟网站生成的值一对比，发现不一样，其实也正常，这个算法是rsa，非对称加密，也可以用标准库去解决试试，反正这里我已经扣下来js了就这样不管了。

到这里，`u`已经搞定。

## 5. `p`逆向

重头戏来了，这里`p`的组成比较复杂，需要逆向的值有好几个。

![](https://attach.52pojie.cn/forum/202503/21/213821n6s635sminccz6gg.png)

22.png (18.5 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5NnwxMTA4ODRjZnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5NnwxMTA4ODRjZnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

`X["encrypt"](ae["stringify"](o), n["$_CDHy"]())`

`p`来自于`h`，那先解决`h`。观察`n["$_CDHy"]()`的值跟上一步一样，其实也就是一个加密key值，流程如下：

> - 先`e() + e() + e() + e()`生成一个字符串，作为明文来rsa加密
> - 接下来的所有需要用到这个明文值的地方，都保持为第一次生成的

所以`h`的重点在于`X["encrypt"]`和`ae["stringify"](o)`以及`o`是什么，那就一个个来，先看`o`

### 1. `o`

`o`的组成很复杂，有这么多东西

> - `a`：其实是点击的坐标，但经过处理
> - `ep`：看起来像轨迹和环境
> - `h9s9`：可能要逆向
> - `lang`、`pic`：固定或者之前能取到
> - `rp`、`tt`：不知道含义，要逆向

![](https://attach.52pojie.cn/forum/202503/21/213823bhau5dt4dlfofqal.png)

23.png (25.89 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5N3w4NDNhYWVjYXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5N3w4NDNhYWVjYXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

### 2. `ae["stringify"]`

其实就是`JSON.stringify`，直接改写就好

### 3. `X["encrypt"]`

跟进去看看，跟之前`u`的`var H = function(){}`一样，直接把整个`var X = function(){}`拿出来。这里先不贴图，先往下解决再一起看

![](https://attach.52pojie.cn/forum/202503/21/213825mz1311w5gvhgy515.png)

24.png (15.08 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDY5OHw5YzgyYWFmYnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDY5OHw5YzgyYWFmYnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:38 上传

到此，`h`就搞定了，接下来看看`p`的逻辑。

![](https://attach.52pojie.cn/forum/202503/21/214144btgnnodon5benzey.png)

25.png (31.17 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcxOXxjMWY0MWUyOHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcxOXxjMWY0MWUyOHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

本质上是调用的`this["$_ECS"](e)`

![](https://attach.52pojie.cn/forum/202503/21/214146h2lt5lget2qgl45g.png)

26.png (14.25 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyMHxlZGMyN2MzYXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyMHxlZGMyN2MzYXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

![](https://attach.52pojie.cn/forum/202503/21/214148n5gcynlykqk5lmtm.png)

27.png (28.39 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyMXw3NjNhODg2NXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyMXw3NjNhODg2NXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

`$_ECS`是属于`w = {}`，把整个`w`拿下来，再把之前`var X = function(){}`拿下来，最终js如下：

![](https://attach.52pojie.cn/forum/202503/21/214150n6o5o9c565icpiic.png)

28.png (69.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyMnxmOWU3OGU4NnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyMnxmOWU3OGU4NnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

运行后与网站生成的对比，值一致

![](https://attach.52pojie.cn/forum/202503/21/214152ad0juu6u0rdwp1ug.png)

29.png (32.9 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyM3xkMGY3YTJjMnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyM3xkMGY3YTJjMnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

到此，`p`也搞定。但`o`中还有好几个参数没处理，继续逆他，累鼠了{:301_972:} {:301_972:} 

多提一嘴其实这里是aes加密，不过我试过用标准库来解，发现解后的数组跟网站的值对不上，就索性直接扣了，应该是漏了啥，本人对密码学属实不了解

## 6. `o`逆向

`o`有以下字段，实际需要逆向的是`a`，`rp`，`tt`，其他的固定即可，不校验，不过其实也可以跟，大概说下我的理解：

> - `a`：点击坐标，要特殊处理
> - `ep`：鼠标移动的全部轨迹，还有一些浏览器事件的环境
> - `h9s9`：看其他大佬描述为每天变化的值，实际测下来这个值好几天没变了，固定即可
> - `passtime`：点击耗时，随机就行
> - `rp`、`tt`：含义未知，逆他就完事

![](https://attach.52pojie.cn/forum/202503/21/214154viychckukpq2uuk8.png)

30.png (20.64 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyNHw3ZGZlN2E1MHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyNHw3ZGZlN2E1MHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

### 1. `a`

先看看`a`吧。在`w`的位置向上看，定位到`a`的逻辑，发现是参数过来的，那就下断再往前的栈跟

![](https://attach.52pojie.cn/forum/202503/21/214156tlu21mz1509zpza2.png)

31.png (18.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyNXw5YjY0Y2YyYXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyNXw5YjY0Y2YyYXwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

向前跟一个栈就能看到`a`其实来自于`n["$_BJGs"]["$_EJC"]()`，继续向前跟。

![](https://attach.52pojie.cn/forum/202503/21/214159x0hk000z0jz2ich7.png)

32.png (49.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyNnwxNzgxODk2NHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyNnwxNzgxODk2NHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:41 上传

发现其实来自于`e["$_CEFZ"]`

![](https://attach.52pojie.cn/forum/202503/21/214201athv553386vxys3e.png)

33.png (24.68 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyN3xiZDEzZjc2MHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyN3xiZDEzZjc2MHwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:42 上传

搜一下"`$_CEFZ`"，发现又来自于`t`，`t`在上一个栈，再跟

![](https://attach.52pojie.cn/forum/202503/21/214203wkump4woww3aru03.png)

34.png (22.6 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDcyOHw3OThlMTkyNnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDcyOHw3OThlMTkyNnwxNzkzNTcxMTE0fDE2ODA4NnwyMDE3Mzk0&nothumb=yes)

2025-3-21 21:42 上传

跟到这里了，下断点刷新在跟
