
# [原创] 破解 [CrackMe] 自己平时用py写了个一体化打包，来检验一下好破解吗 思路

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=2061816`
- 抓取时间: `2026-11-01T22:02:39`

原帖地址：自己平时用py写了个一体化打包，来检验一下好破解吗 - 52pojie.cn

## 1. 首先观察楼主给出的图片，根据打码的长度和下面汉字的长度（一个汉字两个字符长度），判断出密码是10位

![](https://attach.52pojie.cn/forum/202509/22/102712dggrhggmdfwnwadr.png)

## 2. 下载`exe`拖入`die`判断，其实从图标也能看出是`pyinstaller`封装的

![](https://attach.52pojie.cn/forum/202509/22/102905ikouahqtpuvg881v.png)

## 3. 使用在线工具解包

`PyInstaller Extractor WEB`

这个可以把`exe`解包成压缩包，直接选择文件上传`exe`即可，解包后自动下载压缩包

![](https://attach.52pojie.cn/forum/202509/22/103101rdvluttzlnk7qnlx.png)

可以观察到主要的入口`pyc`文件

![](https://attach.52pojie.cn/forum/202509/22/103318xkus60ehd5nwwkud.png)

同时也观察到作者使用了`pyarmor`进行混淆加密，参考资料：`Pyarmor 9.0 用户文档 — Pyarmor 9.0.8 文档`

![](https://attach.52pojie.cn/forum/202509/22/103427zuxlqmuqrlddrss3.png)

## 4. 使用在线工具反编译`PYC`文件 `PyLingual`

![](https://attach.52pojie.cn/forum/202509/22/103729iljxjrrorfvjx2jd.png)

这里截图时显示排队了，使用原帖下面的截图充当一下

![](https://attach.52pojie.cn/forum/202509/22/104028ad8i070jwtjdd8il.png)

## 5. 使用github上的工具直接破解 `Lil-House/Pyarmor-Static-Unpack-1shot: ✅ No execution ✅ Pyarmor 8.0 - latest 9.1.x ✅ Universal ✅ Statically convert obfuscated scripts to disassembly and (experimentally) source code.`

该工具会自动识别`pyc`中的`pyarmor`混淆加密自动解密

![](https://attach.52pojie.cn/forum/202509/22/104304dc122e1plxxqzk2w.png)

## 6. 得到第一层解密后的python代码

```python
# Source Generated with Decompyle++
# File: obfuscated_jiami.pyc.1shot.seq (Python 3.13)

'__pyarmor_enter_860__(...)'
__assert_armored__ = '__pyarmor_assert_859__'
import sys
import os
import threading
import time

def _anti_debug_check():
    '''反调试检查'''
    '__pyarmor_enter_863__(...)'
    __assert_armored__ = '__pyarmor_assert_862__'
# WARNING: Decompyle incomplete

None(threading.Thread(target = _anti_debug_check, daemon = True).start)
import base64

def xor_decrypt(data, key):
    '''XOR 解密函数'''
    '__pyarmor_enter_866__(...)'
    __assert_armored__ = '__pyarmor_assert_865__'
    _var_var_3 = None(len, key)
    _var_var_4 = None(bytearray)
# WARNING: Decompyle incomplete

SECRET_KEY = b'my_secret_key_123'
ENCRYPTED_B64 = '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'
encrypted_data = base64.b64decode(ENCRYPTED_B64)
decrypted_script = xor_decrypt(encrypted_data, SECRET_KEY)
exec(decrypted_script, {
    '__name__': '__main__' })
'__pyarmor_exit_861__(...)'
```

很显然这个`base64`解码并异或解密后就是原始保护文件

解密后的python文件

```python
#!/usr/bin/env python3

# -*- coding: utf-8 -*-

import base64

import hashlib

import sys
```
