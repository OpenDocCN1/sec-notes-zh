
``` 
timestamp: Math.round(            new Date()[_0x2639ff(0x3e3, '\x69\x4a\x58\x55')](),
          ),
          nonce: _0x1b59d6.sCtwF(
            Math[_0x2639ff(0x5ac, '\x64\x6d\x6e\x5d')](
              new Date()[_0x2639ff(0x183, '\x28\x67\x62\x55')](),
            ),
            Math[_0x2639ff(0x5df, '\x74\x52\x38\x79')](
              0x5f5e100 * Math[_0x2639ff(0x290, '\x79\x5d\x51\x64')](),
            ),

```

我们对混淆借助控制台进行处理后，还原代码为

[Asm] 纯文本查看 复制代码

```
var timestamp = Math.round(new Date()['getTime']());

var nonce = Math['floor'](100000000 * Math['random']())
```

这两个参数我们就处理完了，接下来就是找`sign`的位置和加密逻辑，我们直接在返回值位置下断点，看看整体的流程。

![](https://attach.52pojie.cn/forum/202507/02/064852weay44lolmreox9z.png)

image.png (40.6 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY0NnwxMDdlNGEyYnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY0NnwxMDdlNGEyYnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## sign逻辑

2025-7-2 06:48 上传

很明显发现了`sign`的赋值位置和大致的逻辑

[JavaScript] 纯文本查看 复制代码

```
_0x1b59d6[
            _0x2639ff(0x147, '\x5e\x33\x58\x44')
          ](md5, _0x3ca1de))
         
```

![](https://attach.52pojie.cn/forum/202507/02/065032dl366aulzqffy6qz.png)

image.png (13.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY0N3xkMTdlN2Q2OXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY0N3xkMTdlN2Q2OXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## sign加密逻辑

2025-7-2 06:50 上传

很明显能看出加密的逻辑，第一个参数是一个函数名，对第二参数进行加密，我们可以看到的函数名是`md5`，传入参数是`_0x3ca1de`，我们需要先简单测试一下是不是标准算法

![](https://attach.52pojie.cn/forum/202507/02/065254ojzze3zgj0jzm309.png)

image.png (31.57 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY0OHxkYjljZWQyOXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY0OHxkYjljZWQyOXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## md5标准检测

2025-7-2 06:52 上传

所以`sign`核心是一个`md5`加密，我们再看看`_0x3ca1de`的生成逻辑

![](https://attach.52pojie.cn/forum/202507/02/065418x43211ewj0z11uzo.png)

image.png (67.79 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY0OXwyM2ExOWQ2M3wxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY0OXwyM2ExOWQ2M3wxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 生成逻辑1

2025-7-2 06:54 上传

![](https://attach.52pojie.cn/forum/202507/02/065444dmm8blzey66040x4.png)

image.png (15.4 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1MHwzNWVkYWNjZXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1MHwzNWVkYWNjZXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 生成逻辑2

2025-7-2 06:54 上传

大体上就是对前面的请求头的参数转化为字符串拼接，所以这一部分我们整体用`python`还原

[Python] 纯文本查看 复制代码

```
from hashlib import md5

import time

def sign_md5(en_data):

    return md5(en_data.encode()).hexdigest()

def nonce_get():

    t1 = round(time.time() * 1000)  

    t2 = random.randrange(100000000) 

    return t1 + t2

def first_response():

    timestamp = str(int(time.time() * 1000))

    nonce = nonce_get()

    headers = {

    }

    data = {

        "appId": "dc1db94ea7b3843c",

        "dc": 0,

        "ec": 0,

        "hc": 0,

        "nonce": nonce,

        "os": 3,

        "pc": 0,

        "phone": 10000000000,

        "pn": "com.web.tianyu",

        "rc": 0,

        "sdkName":脱敏处理,

        "timestamp": timestamp,

        "type": 1,

        "ui": 'null',

        "version": "2.0.0",

        "xc": 0,

    }

    encode_sign = ''.join(f"{key}{value}" for key, value in data.items())

    sign = sign_md5(encode_sign)

    data['sign'] = sign

    response = requests.post(脱敏处理, headers=headers, data=data).json()
```

## 2、校验接口还原分析

完成前置接口的加密逻辑，我们可以提取的参数有背景图地址，滑块地址，`captchaId`，`token`，接下来就是分析校验参口的加密逻辑`report`参数

![](https://attach.52pojie.cn/forum/202507/02/070224awjo057c7o0315wv.png)

image.png (45.84 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1MXxkMDk0ZTZiYXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1MXxkMDk0ZTZiYXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 校验接口作用域1

2025-7-2 07:02 上传

![](https://attach.52pojie.cn/forum/202507/02/070324jhmu9h9uuff0fhb2.png)

image.png (29.65 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1MnxhNjUwMTI4ZHwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1MnxhNjUwMTI4ZHwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 校验接口作用域2

2025-7-2 07:03 上传

![](https://attach.52pojie.cn/forum/202507/02/070356z73d53le42e31l3c.png)

image.png (128.66 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1M3xjODg3MDQ0OXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1M3xjODg3MDQ0OXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 校验接口加密逻辑

2025-7-2 07:03 上传

还是同理跟着作用域可以很快找到加密的地址，都是明文的（说明直接搜也可以找到），然后就是来分析他的加密逻辑

[JavaScript] 纯文本查看 复制代码

```
_0x1b59d6[_0x34f404(0x169, '\x35\x69\x66\x73')](            _0x204684,
            _0x1b59d6.sQorU,
          )(
            JSON[_0x34f404(0x424, '\x7a\x5b\x6d\x49')](
              _0x1022c0[_0x34f404(0x4f5, '\x5a\x69\x26\x6a')],
            ),
          ),
```

还是一样的，我们通过控制台进行还原

![](https://attach.52pojie.cn/forum/202507/02/071702rtkunonxrgbokxoz.png)

image.png (26.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1NHw4M2MyMWQzZHwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1NHw4M2MyMWQzZHwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 校验接口加密逻辑

2025-7-2 07:17 上传

![](https://attach.52pojie.cn/forum/202507/02/071939g7qdtu84ep986uqu.png)

image.png (98.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1NXw4ZDc2M2NhMXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1NXw4ZDc2M2NhMXwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 轨迹

2025-7-2 07:19 上传

可以看到前面一部分是通过 `_0x204684`(`_0x1b59d6.sQorU`)返回一个函数，后面轨迹的值就是传入的参数

![](https://attach.52pojie.cn/forum/202507/02/072036yh2l6zc515c12aa1.png)

image.png (227.22 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1Nnw3OWRlMDU4MnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1Nnw3OWRlMDU4MnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 加密值

2025-7-2 07:20 上传

很明显他的加密逻辑就是对轨迹的处理，我们跟进去看看。

![](https://attach.52pojie.cn/forum/202507/02/072212m7nqxhhkk74tuwxu.png)

image.png (104.96 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1N3xmYTZiODExY3wxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1N3xmYTZiODExY3wxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 加密逻辑1

2025-7-2 07:22 上传

这里的嵌套比较多，我们要按照代码的执行顺序来分析。

[JavaScript] 纯文本查看 复制代码

```
_0x145c35[_0x1426e3(0x571, '\x46\x59\x43\x6f')](

              _0x1b59d6[_0x1426e3(0x348, '\x69\x4b\x4c\x39')](

                atob,

                _0x4070dc.vConfig.k,

              ),

            )
```

![](https://attach.52pojie.cn/forum/202507/02/072411tjhh1agp80v8fddd.png)

image.png (86.01 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NjY1OHxlMTA3M2ViYnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NjY1OHxlMTA3M2ViYnwxNzkzNTcwODUzfDE2ODA4NnwyMDQyODk4&nothumb=yes)

## 加密逻辑1分析

2025-7-2 07:24 上传

这样我们就有了一个比较清晰的逻辑，还有一个大胆的猜测，这是一个`rsa`加密，`setPublicKey(atob(_0x4070dc.vConfig.k))`

![](https://attach.52pojie.cn/forum/202507/02/072659lkmrrxp724mr7xct.png)
