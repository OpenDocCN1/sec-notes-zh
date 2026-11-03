
# AES CBC PKCS7 加密示例

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import binascii

# 参数还原
plaintext = '欸嘿'
key_str = 'b903bbf7a0871a11'
iv_str = '0000000000000000'

key = key_str.encode('utf-8')
iv = iv_str.encode('utf-8')

# 补足 16 字节
key = key.ljust(16, b'\x00')
iv = iv.ljust(16, b'\x00')

# AES CBC PKCS7 加密
cipher = AES.new(key, AES.MODE_CBC, iv)
ciphertext = cipher.encrypt(pad(plaintext.encode('utf-8'), AES.block_size))

# 输出 hex 串用于比对
hex_output = binascii.hexlify(ciphertext).decode()
print(hex_output)
```

## 结果

结果如下：

![](https://attach.52pojie.cn/forum/202504/16/185639tyy01e1bnlebw99l.png)

16.png (221.33 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3MTc5NnwzNTQzOTgwNHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5NnwzNTQzOTgwNHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

成功了，不用扣代码了。爽！不过这个扣的难度不高。认真一点没啥问题。

最后的结果就是--> 两个结果合并返回。

最后测试一下，嗯哼，没毛病。无感相较于滑块简单点。滑块还有轨迹收集。
