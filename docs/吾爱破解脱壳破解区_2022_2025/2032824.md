
# [Web逆向] 某投资网站登录密码逆向(适合小白新手)

## 基本信息
- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=2032824`
- 抓取时间: `2026-11-01T22:09:26`
- 目标网址

```
- aHR0cHM6Ly93d3cuaGZheC5jb20vbG9naW4uaHRtbA==

- 输入测试账户和密码(验证码自己输)
```

- `username`: `admin`
- `password`: `123456`

## 调试过程
调试查看信息如下,显然密码是经过加密的,所以目标就是找出代码的加密逻辑,模拟即可

```
password: "2bc3d344845b58a87cd0b0f34f84ff79"
username: "admin"
```
![](https://attach.52pojie.cn/forum/202505/21/084241k4skr1k1isk1lava.png)
6.png (26.11 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3ODI4Mnw3M2JjOGNkY3wxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODI4Mnw3M2JjOGNkY3wxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes)
2025-5-21 08:42 上传

- 点击请求的启动器,直接定位到`toLogin`代码块
![](https://attach.52pojie.cn/forum/202505/21/084137gnf2bq22pfdf72j2.png)
2.png (34.77 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3ODI3OHxlZGNkMmE2NXwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODI3OHxlZGNkMmE2NXwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes)
2025-5-21 08:41 上传

- 点进去以后,定位到了发请求的地方,所以这边打上断点,我们点击登录按钮,查看断点调试信息
![](https://attach.52pojie.cn/forum/202505/21/084149qhatana47ah5z017.png)
3.png (54.57 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3ODI3OXwxOWVmZTcwOHwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODI3OXwxOWVmZTcwOHwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes)
2025-5-21 08:41 上传
![](https://attach.52pojie.cn/forum/202505/21/084213u8p54zs88p98gzsx.png)
4.png (82.72 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3ODI4MHw0ZTI3YzI4Y3wxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODI4MHw0ZTI3YzI4Y3wxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes)
2025-5-21 08:42 上传

- 很幸运,密码的加密逻辑,就在发请求的附近

`password`的值为`a`,而`a = encryptByDES(this.password)`;

现在进入`encryptByDES`的逻辑
![](https://attach.52pojie.cn/forum/202505/21/084227bzim9myma7m9tyoi.png)
5.png (63.57 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3ODI4MXxhMTlhNGQ2NHwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODI4MXxhMTlhNGQ2NHwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes)
2025-5-21 08:42 上传

## 加密逻辑
- 加密逻辑如下

```
encryptByDES: function(t) {
        return CryptoJS.MD5(t + "TuD00Iqz4ge7gzIe2rmjSAFFKtaIBmnr8S").toString()
},
```
- 丢给AI分析,是`MD5`加密

- 参数`t`就是`password`,后面一堆字符串就是'盐'
  - 注意事项: 这个'盐'有可能是固定值(很幸运就是),也有可能是随机算法生成的(比较绕了)

## Python 模拟
- 使用pyton模拟如下,结果一致,逆向成功!

```
import hashlib

def encrypt_by_des(t: str) -> str:
    # 固定盐值（与 JavaScript 代码一致）
    salt = "TuD00Iqz4ge7gzIe2rmjSAFFKtaIBmnr8S"

    # 拼接原始字符串和盐值
    salted_data = t + salt

    # 计算 MD5 哈希值
    md5_hash = hashlib.md5(salted_data.encode("utf-8")).hexdigest()

    return md5_hash

# ---------------
# 使用示例
# ---------------
if __name__ == "__main__":
    plaintext = "123456"
    hashed_result = encrypt_by_des(plaintext)
    print(f"输入: {plaintext}") # 输入: 123456
    print(f"输出: {hashed_result}") # 输出: 2bc3d344845b58a87cd0b0f34f84ff79
```
![](https://attach.52pojie.cn/forum/202505/21/084119gv8xxpbpxj1myz8x.png)
1.png (25.77 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc3ODI3N3xiZWM5NGFhYnwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3ODI3N3xiZWM5NGFhYnwxNzkzNTcwOTY0fDE2ODA4NnwyMDMyODI0&nothumb=yes)
2025-5-21 08:41 上传

## 小结
4> 小结: 本次逆向适合对象是新手小白(本人就是),学会基本的跟栈和打断点的操作
