
```
plaintext = b'{"sign":"12345678901234567890123456789012345678901234567890","email":"marlkiller","deviceID":"ee4f1d1890b4eb49a5a4d7f195ca8b67","licenseKey":"licenseKey","purchasedAt":"2026-12-30","nextChargeAt":521,"updatesAvailableUntil":"2026-12-30"}'
encryptor = Encryptor()
encrypted_data = encryptor.encrypt(plaintext, password)
write_encrypted_data_to_file(encrypted_data, licenseFile)
print("Mac Encrypt: " + encrypted_data.hex())
```

## 4. 后记

项目已经打包 github，可以直接用 `xcode` 打开：[github.com/marlkiller/dylib_dobby_hook](https://github.com/marlkiller/dylib_dobby_hook)

```
git clone https://github.com/marlkiller/dylib_dobby_hook.git
cd script
sudo bash auto_hack.sh
```

- 软件有接口请求验证，自行断网测试，`tableplus.com` 这里不做讨论
- mac/win `password` 不一致，但是算法通杀。
- `deviceId` 的生成规则：[`EncryptionUtils.m` 第55行](https://github.com/marlkiller/dylib_dobby_hook/blob/7d1a68001452d18f6ed52fd4fbac78bd61013090/dylib_dobby_hook/utils/EncryptionUtils.m#L55)
