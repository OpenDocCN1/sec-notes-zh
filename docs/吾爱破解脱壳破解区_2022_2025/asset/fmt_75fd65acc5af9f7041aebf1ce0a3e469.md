
# [Web逆向] 某花顺登录滑块逆向

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2035508
- 抓取时间：2026-11-01T22:08:49

本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！

目标：实现账号登录、滑块验证与 `python` 纯算。

网站：aHR0cHM6Ly91cGFzcy4xMGpxa2EuY29tLmNuL2xvZ2luP3JlZGlyPUhUVFBfUkVGRVJFUg==

## 1. 分析登录过程中的所有请求

老样子先分析登录过程中需要的所有请求：账号密码登录、触发滑块，先寻找获得图片的接口，分上下两部分分析整个过程。该网站会禁用 `f12`，所以在请求之前就先打开控制台。

我们根据验证码图片的 URL 可以找到，在 `getprehandle` 请求中有我们想要的参数，后面再分析。

![](https://attach.52pojie.cn/forum/202505/31/123641sid3i00zl2lgi3e0.png)

## 2. 验证码触发前的请求

验证码触发前，主要有两次请求：`getGS` 和 `dologinreturnjson2`。在 `getGS` 的请求参数中可以看到是在验证账号，得到 `dsk`、`dsv`、`ssv`，这些后面都会用到。

![](https://attach.52pojie.cn/forum/202505/31/124229mrozcwgklwvzq11s.png)

![](https://attach.52pojie.cn/forum/202505/31/124336pllbgm8pl684ljnp.png)

### 2.2 `dologinreturnjson2` 请求

在 `dologinreturnjson2` 请求中可以看到参数有点多，结果触发验证码。需要分析的参数就是 `uname`、`passwd`、`passwdsalt`、`dsk`、`crnd`。

![](https://attach.52pojie.cn/forum/202505/31/124523lelt398y35y3k5ay.png)

![](https://attach.52pojie.cn/forum/202505/31/124603qktm59jjmo5sma8v.png)

## 3. 定位参数生成位置

先是定位到参数生成的地方，那我直接根据参数名搜索 `passwdsalt`，这个结果会少很多，可以快速定位。跳转过来之后，在其之上就是所有参数的生成。太好了，源码中还有注释，直接都打上断点逐步分析。

![](https://attach.52pojie.cn/forum/202505/31/124912jocne5fkejcobd9c.png)

## 4. 分析 `uname` 与 `passwd` 参数

登录账号先定位到 `uname`，先看 `uname` 和 `passwd` 这两个参数。加密方式还备注上了，真好。可以看到，传入账号-密码，经过 `thsencrypt` 中 `encode()` 方法加密；密码参数多做了一步处理，经过 `hex_md5()`，看名字应该是 MD5 编码。在控制台打印测试一下，找个网站测试一下是标准的 MD5，这就可以不用分析 `js` 源码了，主要进 `thsencrypt` 的 `encode()` 方法里面看看。单步跳进去。

![](https://attach.52pojie.cn/forum/202505/31/125524an7cxnx3l4lk9l7w.png)

![](https://attach.52pojie.cn/forum/202505/31/130057w17ab4p98xsy11b1.png)

## 5. 分析 `encode()` 方法

在 `encode()` 中传入三个参数，直接返回函数 `encryptencode()` 的结果，其中 `b` 是账号，`a` 是密钥，`c` 是公钥指数，返回 Base64 格式字符串。转换成 `python` 代码。

```python
def encrypt_encode(plaintext: str) -> str:
    modulus_hex = ""  # 这里填入十六进制模数字符串
    exponent_hex = "10001"  # 一般是 65537 的十六进制表示
    if not modulus_hex or not exponent_hex:
        raise ValueError("modulus and publicExponent could not be null")
    modulus = int(modulus_hex, 16)
    exponent = int(exponent_hex, 16)
    key = RSA.construct((modulus, exponent))
    cipher = PKCS1_v1_5.new(key)
    plaintext_bytes = plaintext.encode('utf-8')
    encrypted_bytes = cipher.encrypt(plaintext_bytes)
    if not encrypted_bytes:
        raise ValueError("encrypt failed")

    return b64encode(encrypted_bytes).decode('utf-8')
```

![](https://attach.52pojie.cn/forum/202505/31/130557bbgu9bb9qunree4e.png)

## 6. 验证 RSA 代码与 `crnd` 生成

先验证代码是否能用，测试是不是标准 RSA，因为加密结果每次都不一样，直接测试请求。在 `getGS` 中请求参数是 `uname`、`rsa_version` 和 `crnd`，现在看一下 `crnd` 怎么生成，继续往下走，进入 `getSaltGs()` 中。

![](https://attach.52pojie.cn/forum/202505/31/131952fyhu70eeyg9zh0fr.png)

![](https://attach.52pojie.cn/forum/202505/31/132109dmf6r919bk1e18o2.png)

## 7. 分析 `crnd` 随机数生成

可以看到随机数转 36 进制取后八位，重复两边，生成 16 位只包含数字和字母的字符串。那我直接从 0 到 9 和小写字母中随机取十六位，一样的效果。

![](https://attach.52pojie.cn/forum/202505/31/132758m22pu0z74x7u4e5t.png)

## 8. 测试 `getGS` 与 `passwdsalt` 生成

用生成的 `uname` 和 `crnd` 测试 `getGS`。若 `uname` 参数为空，返回参数错误；若加密不对，返回的 `dsk`、`ssv` 等都是空。

测试完接着往下分析。相对 `dologinreturnjson2` 里面还有 `passwdsalt` 和 `dsk` 这两个参数没有分析，`dsk` 是 `getGS` 中返回的。

继续往下走：`postData.passwdSalt = encodeDataSaltOnce(passwdVal, unameVal);`，进入 `encodeDataSaltOnce()` 中，分析生成过程。

![](https://attach.52pojie.cn/forum/202505/31/134608ll20syey28l0o251.png)

![](https://attach.52pojie.cn/forum/202505/31/141202fer9bdbpdzgqzddy.png)

## 9. 分析 `encodeDataSaltOnce()` 与 `getStrXOR()`

先到第一次生成 `n` 的地方，可以看到是拿 `crnd + dsk`，传给 `r.hex`，`r` 在上面定义的有，可以看到是 `hash 256`。往下走是先把 `ssv` 进行 Base64 解码，再传入 hash 后的 `n`，在函数 `getStrXOR()` 中转换。

可以看到是实现两个字符串的字符逐位异或操作（XOR 操作）。转化成 `python` 代码。

```python
def get_str_xor(e: str, t: str) -> str:
    s = len(e)
    r = len(t)
    o = []
    for d in range(s):
        n = d % r
```

![](https://attach.52pojie.cn/forum/202505/31/141619fnhen6no6gnnnhnl.png)
