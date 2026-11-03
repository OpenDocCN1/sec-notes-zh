
```
(lldb) mem read 0x00600001da5c60 -c 32

0x600001da5c60: da 04 9e 62 d7 dc d9 64 fe 62 0f 33 88 57 3c c8  ...b...d.b.3.W
```

### 2.1.3 生成 `ciphertext`

接着看代码，`license` 解密成功后，会调用 JSON 相关函数转换成 `TBLicenseModel` 对象。

由此得知，我们的 `license` `Ciphertext` 解密后是 json 字符串。

在 `ida` 分析 `TBLicenseModel` 可以得到以下 `ivar` 属性。

![](https://attach.52pojie.cn/forum/202503/14/141901u6blaahhmdhnnddb.png)

`pic5.png`（258.35 KB，下载次数：1）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MTMzN3w3ZDA2YjIyNHwxNzkzNTcxMjI0fDE2ODA4NnwyMDE0NDY5&nothumb=yes)

2025-3-14 14:19 上传

```asm
__const:00000001006E8868 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_sign dq 10h
__const:00000001006E8868                                         ; DATA XREF: __objc_const:0000000100827C68↓o
__const:00000001006E8870 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_email dq 20h
__const:00000001006E8870                                         ; DATA XREF: __objc_const:0000000100827C88↓o
__const:00000001006E8878 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_deviceID dq 30h
__const:00000001006E8878                                         ; DATA XREF: __objc_const:0000000100827CA8↓o
__const:00000001006E8880 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_licenseKey dq 40h
__const:00000001006E8880                                         ; DATA XREF: __objc_const:0000000100827CC8↓o
__const:00000001006E8888 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_purchasedAt dq 50h
__const:00000001006E8888                                         ; DATA XREF: __objc_const:0000000100827CE8↓o
__const:00000001006E8890 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_updatesAvailableUntil dq 60h
__const:00000001006E8890                                         ; DATA XREF: __objc_const:0000000100827D08↓o
__const:00000001006E8898 _OBJC_IVAR_$__TtC9TablePlus14TBLicenseModel_nextChargeAt dq 70h
```

根据 `TBLicenseModel` 的 `ivar` 属性来拼接一个字符串。

我们可以利用 `key + iv` 来做一个 `aes cbc` 加密字符串，生成 `ciphertext`。

![](https://attach.52pojie.cn/forum/202503/14/144041zbcivy1mhzf7vvbi.png)

`6.png`（727.64 KB，下载次数：1）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MTM1Nnw3YzVhY2VjOXwxNzkzNTcxMjI0fDE2ODA4NnwyMDE0NDY5&nothumb=yes)

2025-3-14 14:40 上传

生成之后，替换我们 `license` 文件中的 `ciphertext`。

然后 `lldb r` 重启。

### 2.1.4 生成 `HMAC`

`HMAC` 是用来验证文件的完整性的，用于数据完整性校验，如果 `HMAC` 不匹配，则数据可能被篡改。

算法如下：

```text
header = version + options + encryptionSalt + HMACSalt + IV

hmac_message = header + ciphertext

HMAC = HMAC-SHA256(hmac_key, hmac_message)
```

![](https://attach.52pojie.cn/forum/202503/14/141854ox2h025x3hzz26q5.png)

`pic7.png`（203.22 KB，下载次数：1）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MTMzM3wwZmIzYmRjY3wxNzkzNTcxMjI0fDE2ODA4NnwyMDE0NDY5&nothumb=yes)

2025-3-14 14:18 上传

取到之后，替换我们 `license` 文件中的 `hmac`。

`9d79987595a230de47d90f8d7842b7821696a7517a8a2d8ce98be470ce3e2a52`

至此，`license` 伪造如下：

![](https://attach.52pojie.cn/forum/202503/14/141854e5o3tvgfsmcsc5tf.png)

`pic10.png`（185.73 KB，下载次数：1）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MTMzNHwzZjBkMmRhOXwxNzkzNTcxMjI0fDE2ODA4NnwyMDE0NDY5&nothumb=yes)

2025-3-14 14:18 上传

然后 `lldb` 重启，继续调试。

## 2.2 验证

解密后的 json 字符串会调用 `JSONDecoder.decode` 转成 `LicenseModel` 对象，我们在 `0000000100144F68` 下断。

```lldb
br s -a 0x100144F68

* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 4.1
    frame #0: 0x0000000100144f68 TablePlus`___lldb_unnamed_symbol8433 + 1368
TablePlus`___lldb_unnamed_symbol8433:
->  0x100144f68 : callq  0x1006d5c8c    ; symbol stub for: dispatch thunk of Foundation.JSONDecoder.decode(_: τ_0_0.Type, from: Foundation.Data) throws -> τ_0_0
    0x100144f6d : movq   %r14, %rbx
    0x100144f70 : testq  %r12, %r12
    0x100144f73 : je     0x100144fb7    ; 
Target 0: (TablePlus) stopped.
(lldb) 
```

断下来后，输出前几个参数调试一下。

可以看到第一个参数是一个 `TBLicenseModel` 类的模板，第二个参数未知，第三个参数 `Foundation.__DataStorage` 看着像是我们要验证的数据。

```lldb
(lldb) po $rdi

TablePlus.TBLicenseModel

(lldb) po $rsi

1060856922112

(lldb) reg r rsi

     rsi = 0x000000f700000000

(lldb) po $rdx

4611791571565053824

(lldb) reg r rdx

     rdx = 0x4000600001468780

(lldb) po 0x600001468780

Foundation.__DataStorage
```

```swift
class __DataStorage {
    var isa: objc_class* // NULL for pure Swift class
    var refCount: UInt64
    var _bytes: UnsafeMutableRawPointer?
    var _length: UInt64
    var _capacity: UInt64
    var _offset: UInt64
    var _deallocator: ((UnsafeMutableRawPointer, Int64) -> Void)?
    var _needToZero: Bool
}
```

```lldb
(lldb) x/8xg 0x600001468780
0x600001468780: 0x00007ff852ef01e0 0x0000000000000003
0x600001468790: 0x00007feeedd21c60 0x00000000000000f7
0x6000014687a0: 0x0000000000000140 0x0000000000000000
0x6000014687b0: 0x0000000000000000 0x0000000000000000
```

`__DataStorage` 是 Swift 的类，可以看到有下面几个属性。通过 `lldb` 命令查看指针的一些偏移，可以看到第三个为 `char*` 数组，第四个为长度；读取该地址内存，可以看到我们的 `license` 成功解密出来了。

```lldb
mem read 0x00007feeedd21c60 -c 0xf7
```

![](https://attach.52pojie.cn/forum/202503/14/141849w2aazhkenzbh66a2.png)

`pic8.png`（328.13 KB，下载次数：1）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MTMzMXxmNmZjYTIxOHwxNzkzNTcxMjI0fDE2ODA4NnwyMDE0NDY5&nothumb=yes)

2025-3-14 14:18 上传

至此，`license` 解密完成。

`lldb r` 重启程序，然后直接运行，看效果。

![](https://attach.52pojie.cn/forum/202503/14/152036ddz91tkeo9o662ff.png)

`fina.png`（345.11 KB，下载次数：1）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MTM2NnwwOGY4NmYwYXwxNzkzNTcxMjI0fDE2ODA4NnwyMDE0NDY5&nothumb=yes)

2025-3-14 15:20 上传

# 3. `keygen`

利用前面获取到的各种信息，我们用 `python3` 基于 `pycryptodomex` 库手写一个 `keygen`，来方便快捷生成 `license` 文件。

```python
# pip3 install pycryptodomex
class CryptoBase:
    def __init__(self):
        self.schema_version = 3
        self.aes_mode = AES.MODE_CBC
        self.options = 1
        self.hmac_includes_header = True
        self.hmac_includes_padding = False
        self.hmac_algorithm = SHA256
        self.key_len = 32
class Encryptor(CryptoBase):

    def encrypt(self, plaintext, password):
        salt = os.urandom(8)
        hmac_salt = os.urandom(8)
        iv = os.urandom(16)

        key = self.generate_key(salt, password)
        hmac_key = self.generate_key(hmac_salt, password)

        cipher = AES.new(key, AES.MODE_CBC, iv)
        ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))

        # schema + options + salt + hmac_salt + iv
        header = self.assemble_header(salt, hmac_salt, iv)

        print_hex(key, "key")  # schema (1 byte)

        hmac_message = header + ciphertext
        hmac = self.generate_hmac(hmac_message, hmac_key)

        encrypted_data = header + ciphertext + hmac
        return encrypted_data

    def generate_key(self, salt, password):
        return hashlib.pbkdf2_hmac('SHA1', password.encode('utf-8'), salt, 10000, dklen=self.key_len)

    def assemble_header(self, salt, hmac_salt, iv):
        schema = struct.pack('B', self.schema_version)
        options = struct.pack('B', self.options)
        return schema + options + salt + hmac_salt + iv

    def generate_hmac(self, message, key):
        hmac = HMAC.new(key, digestmod=self.hmac_algorithm)
        hmac.update(message)
        return hmac.digest()

if __name__ == '__main__':

    licenseFile = '/Users/voidm/Library/Application Support/com.tinyapp.TablePlus/.licensemac'

    password = "马赛克"
```
