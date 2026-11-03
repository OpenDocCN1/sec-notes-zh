
逗号表达式我借用的是蔡老板的插件，尊重蔡老板的知识产权，知识给出我自己写的ast部分，大致思路先将ob三特征放入内存方便调用，要注意的是需要格式化处理，然后是十六进制和编码的还原，接下来就是混淆的核心，多次赋值解密函数调用，我采用的是收集所有的赋值，替换函数名为解密函数，直接还原，比较菜，大佬们仅供参考。这样就实现了这个混淆代码的还原，接下来用工具替换，我采用的是fiddler本地替换。

![](https://attach.52pojie.cn/forum/202507/07/221237qqqphhpqji00ta0f.png)

image.png (90.17 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwM3xmNTc4MmQxZnwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

fiddler替换

2025-7-7 22:12 上传

看看替换后的效果

![](https://attach.52pojie.cn/forum/202507/07/221507mr6th7iiyr5s5i61.png)

image.png (114.35 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwNHwyMDY5NjMwY3wxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

替换后前置接口

2025-7-7 22:15 上传

一下子神清气爽了，很舒畅。然后继续我们的参数分析。

![](https://attach.52pojie.cn/forum/202507/07/221642ta46d8nnn3duu6fl.png)

image.png (15.81 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwNXxjOTY2N2Q1YnwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

参数位置

2025-7-7 22:16 上传

这些参数位置还是很清晰的，直接往上找找是咋生成的。

![](https://attach.52pojie.cn/forum/202507/07/221804xzsbxbygzvh3gfmd.png)

image.png (118.88 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwNnw0MWJmY2VkZnwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

参数生成定位

2025-7-7 22:18 上传

最上面是一个`n`对象，后面对`n`进行了处理，还有一个`a`对象，同样的方法对进行了处理，然后拼接进行了一个md5的操作。很清晰的逻辑。

![](https://attach.52pojie.cn/forum/202507/07/221937gf6g468gf45010oe.png)

image.png (19.24 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwN3w1NGRjNTM4Y3wxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

y函数

2025-7-7 22:19 上传

我们在测试`y`的时候发现是一个动态变化的值，看看他的生成逻辑，转换成python代码

```javascript
function Y(x, t, i) {
    var n = e;
    var r = ix(i);
    if (r)
        return r;
    r = "",
    x = x || 32,
    t = t || 0;
    for (var a = ["ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789", "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz", "0123456789", "0123456789abcdef"][t], c = a["length"], o = 0; o 
```

```python
import math
import random

def range_y(x=32, t=0):
    a = [
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789",
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz",
        "0123456789",
        "0123456789abcdef"
    ][t]
    c = len(a)
    r = ""
    for o in range(x):
        r += a[math.floor(random.random() * c)]
    return r
```

接下来就是`_0x342058`函数的实现逻辑

```javascript
e["prototype"]["encrypt"] = function(e, t) {
    var i = x;
    return _0x110baa["AES"].encrypt(t, this.key, {
        iv: _0x110baa["enc"]["Hex"]["parse"](e)
    })["ciphertext"]["toString"](_0x110baa["enc"]["Base64"])
}

function _0x342058(x, e) {
    var t = _0x15b61a;
    var i = "69783956775867344e5853626b645431";
    var n = new _0x254c65(128,1e3);
    return 0 === e ? n["encrypt"](i, x) : n["decrypt"](i, x)
}
```

应该还是很明显，这是一个aes加密，但是不同于之前的滑块，这里采用了一个hex的方式去处理数组，这个在python中对应的是`bytes.fromhex`函数，这里可以看到有一个`iv`，可以猜测是CBC加密，

![](https://attach.52pojie.cn/forum/202507/07/222454i0vjynyhuvhbtztc.png)

image.png (19.09 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwOHw1ZmU3NTQwN3wxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

key值

2025-7-7 22:24 上传

这里发现`key`值已经被转化了，我们需要先去还原他的字符串型形态的值，再用python去转化

```python
import struct

def word_array_to_hex_string(words, sig_bytes):
    byte_list = []
    for word in words:
        if word I', word)
        byte_list.extend(bytes_)
    byte_list = byte_list[:sig_bytes]

    return bytes(byte_list).hex()
```

这个函数的效果我们可义用转化后的`iv`进行测试，发现没有问题，最后得到我们的`key`值为`69783956775867344e5853626b645431`，我们就可以实现这一部分的加密逻辑

```python
import base64
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.backends import default_backend

def aesEncrypt(plaintext):
    iv = bytes.fromhex("69783956775867344e5853626b645431")
    key = bytes.fromhex('8f235bc1cac17a46530c616ff234be78')
    if isinstance(plaintext, str):
        plaintext = plaintext.encode('utf-8')
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    encryptor = cipher.encryptor()
    padder = padding.PKCS7(128).padder()
    padded_data = padder.update(plaintext) + padder.finalize()
    ciphertext = encryptor.update(padded_data) + encryptor.finalize()
    return base64.b64encode(ciphertext).decode('utf-8')
```

稍微检测一下

![](https://attach.52pojie.cn/forum/202507/07/222931uzkequxqtba4ezc3.png)

image.png (22.3 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcwOXw1YjYwYmQwM3wxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

aes加密检测

2025-7-7 22:29 上传

蒙的很准{:1_889:}

![](https://attach.52pojie.cn/forum/202507/07/223037p49408j8d59db8ba.png)

image.png (29.63 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcxMHxjZTEwNmQ3OHwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

a值加密

2025-7-7 22:30 上传

接下来的`a`值同样的函数加密就不过多赘叙了，后面就是一个md5拼接，先检测是不是标准的。

![](https://attach.52pojie.cn/forum/202507/07/223212n26rd11o9ygstj1r.png)

image.png (10.75 KB, 下载次数: 0) [附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcxMXw4MDRhNGQxNHwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

md5加密

2025-7-7 22:32 上传

非常标准，这样我们就完成了第一部分加密，我们看到后面还有一个`encodeURIComponent`函数，是url的一个编码处理，对应的的是`from urllib.parse import quote`，简单实现一下
