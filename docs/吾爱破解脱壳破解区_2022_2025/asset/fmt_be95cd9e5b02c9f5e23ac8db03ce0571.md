
[href="forum.php?mod=attachment&aid=Mjc4Nzc1M3xiMDJjZGMxYXwxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4Nzc1M3xiMDJjZGMxYXwxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes)

2025-7-8 03:13 上传

![](https://attach.52pojie.cn/forum/202507/08/031313iznn9hdda9d6a2fd.png)

IMG_0136.png (164.96 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc4Nzc1NHwwNmY1NmNiN3wxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4Nzc1NHwwNmY1NmNiN3wxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes)

2025-7-8 03:13 上传

呜呜呜别骂了别骂了好哥哥，回头找到地方打工给你补票还不行嘛

重新进入`updateLicenseBehaviorAndShowNotification`开始单步调试，发现是之前比较粗，只改了`sub_10010F8D8`里面`isLicenseCompormised`的调用，忘了直接把他改了，遂patch掉这个函数

![](https://attach.52pojie.cn/forum/202507/08/031338bg9mauqhte77emz0.png)

PixPin_2025-07-08_02-28-17.png (59.47 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc4Nzc1NXw2NzIxNjRkZXwxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4Nzc1NXw2NzIxNjRkZXwxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes)

2025-7-8 03:13 上传

再次运行，应该是成功了，不想看他还有没有什么上传key为黑名单啥的操作了，目前的功能也够用，一个磁盘整理工具也不需要联网，所以就直接把它禁止联网了

![](https://attach.52pojie.cn/forum/202507/08/031356cedq0r87ax7bfn8x.png)

PixPin_2025-07-08_02-41-00.png (654.79 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc4Nzc1Nnw1YjE0MjBjY3wxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4Nzc1Nnw1YjE0MjBjY3wxNzkzNTcwODM3fDE2ODA4NnwyMDQ0MDk1&nothumb=yes)

2025-7-8 03:13 上传

## 最后

其实更优雅的解决方案可能还是通过推出来的私钥去构造任意的License，这样完全不需要侵入源文件，但是学艺不精实在是没搞出来，这里吧拿到的公钥和我推出来的私钥分享出来给有兴趣的兄弟稍微省点事儿

```
-----BEGIN PUBLIC KEY-----
MDUwDQYJKoZIhvcNAQEBBQADJAAwIQIaALeN7XjhVyfaLWpCJDR8Ke/6E2KcvAKI
HqMCAwEAAQ==
-----END PUBLIC KEY-----

-----BEGIN RSA PRIVATE KEY-----
MIGKAgEAAhoAt43teOFXJ9otakIkNHwp7/oTYpy8AogeowIDAQABAhkVPP/QL8qk
2tLMa7C/mWP+gQlqWLzGuM1xAg0MpeIf+HSxurX+9AgpAg0OgzYo5Qcyt7fph7nr
Ag0IjDc1ChG7Z4bF5iB5Ag0Ft3omMdHcELLJs0VxAg0BhvFKthFCuBMpyM1d
-----END RSA PRIVATE KEY-----
```

## 最后的最后

这是我第二次尝试破解应用，第一次是一个iOS App，感觉还是手忙脚乱，很多基础知识都不知道，全靠AI吊着，工具链不熟，整个工作流程也很粗糙。

把过程记录下来，一方面是给自己做个备忘，另一方面也想和大伙交流一下，看看有没有什么更好用的工具，思路有没有问题，或者有什么更好的方法。

## 更新

干就干完，最后还是静下心来重新看了下逻辑，再结合 @skrets 老哥发出来的码，把整个流程走完了，感谢老哥的帮助。之前其实只差临门一脚，根本没注意到这边的base32 decode 是用了自定义的字符表的

```
do
{
  *((_BYTE *)&xmmword_10028E0D0 + (unsigned __int8)aAbcdefghjklmnp[v1]) = v1;
  ++v1;
}
while ( v1 != 32 );
// aAbcdefghjklmnp DCB "ABCDEFGHJKLMNPQRSTUVWXYZ23456789",0
```

有uv的话可以直接`uv run --with pycryptodome main.py`，不用uv的话就先安装一下

目前只知道他有个黑名单会下载下来，但是没注意是不是会上传，所以还是建议断掉他的网，推荐LuLu

```
import hashlib

from Crypto.PublicKey import RSA
from Crypto.Util.number import bytes_to_long, long_to_bytes

PRIVATE_KEY = bytes.fromhex("30818a020100021a00b78ded78e15727da2d6a4224347c29effa13629cbc02881ea302030100010219153cffd02fcaa4dad2cc6bb0bf9963fe81096a58bcc6b8cd71020d0ca5e21ff874b1bab5fef40829020d0e833628e50732b7b7e987b9eb020d088c37350a11bb6786c5e62079020d05b77a2631d1dc10b2c9b34571020d0186f14ab61142b81329c8cd5d")
PUBLIC_KEY = bytes.fromhex("3035300D06092A864886F70D01010105000324003021021A00B78DED78E15727DA2D6A4224347C29EFFA13629CBC02881EA30203010001")
BASE32_ALPHABET = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789"
CHAR_MAP = {char: idx for idx, char in enumerate(BASE32_ALPHABET)}

def encode_custom_base32(data: bytes, pad: bool = True) -> str:
    bits = 0
    bit_buffer = 0
    encoded = []
    for byte in data:
        bit_buffer = (bit_buffer = 5:
            bits -= 5
            index = (bit_buffer >> bits) & 0b11111
            encoded.append(BASE32_ALPHABET[index])
    if bits > 0:
        index = (bit_buffer  bytes:
    encoded = encoded.strip("=").upper()
    for c in encoded:
        if c not in CHAR_MAP:
            raise ValueError(f"Invalid character in input: {c}")
    bits = 0
    bit_buffer = 0
    decoded_bytes = bytearray()
    for char in encoded:
        bit_buffer = (bit_buffer = 8:
            bits -= 8
            decoded_bytes.append((bit_buffer >> bits) & 0xFF)
    return bytes(decoded_bytes)

def strip_pkcs1_padding(data: bytes) -> bytes:
    if data[0] != 0x01:
        raise ValueError("Not PKCS#1 v1.5 padding")
    sep = data.find(b'\x00', 1)
    if sep  bytes:
    max_msg_len = total_length - 11
    if len(message) > max_msg_len:
        raise ValueError("Message too long")
    ps_length = total_length - len(message) - 3
    ps = b'\xFF' * ps_length
    return b'\x00\x01' + ps + b'\x00' + message

def decrypt(payload: bytes, key: RSA.RsaKey) -> bytes:
    c = bytes_to_long(payload)
    m = pow(c, key.e, key.n)
    padded = long_to_bytes(m)
    return strip_pkcs1_padding(padded)

def encrypt(payload: bytes, key: RSA.RsaKey) -> bytes:
    k = (key.n.bit_length() + 7) // 8
    padded = pkcs1_padding(payload, k)
    m = bytes_to_long(padded)
    c = pow(m, key.d, key.n)
    return long_to_bytes(c, k)

def build_license_payload(
        customer_name: str,
        issue_date: int = 0,
        num_copies: int = 0,
        license_type: int = 1
) -> bytes:
    normalized_name = " ".join(customer_name.strip().split()).lower()
    name_hash = hashlib.sha1(normalized_name.encode('utf-8')).digest()[:10]
    payload = name_hash
    payload += issue_date.to_bytes(2, 'little')
    payload += num_copies.to_bytes(1, 'little')
    payload += license_type.to_bytes(1, 'little')
    return payload

def format_license_key(license_key: str) -> str:
    return '-'.join(license_key[i:i + 8] for i in range(0, len(license_key), 8))

def verify_license_key(license_key: str, customer_name: str, key: RSA.RsaKey) -> bool:
    try:
        decoded = decode_custom_base32(license_key)
        decrypted_payload = decrypt(decoded, key)
        normalized_name = " ".join(customer_name.strip().split()).lower()
        name_hash = hashlib.sha1(normalized_name.encode('utf-8')).digest()[:10]
        return decrypted_payload[:10] == name_hash
    except Exception as e:
        print(f"Error verifying license key: {e}")
        return False

def main():
    name = "52pojie.cn"
    print(f"Generating license key for: {name}")
    private_key = RSA.import_key(PRIVATE_KEY)
    public_key = RSA.import_key(PUBLIC_KEY)
    payload = build_license_payload(name)
    license_key = encode_custom_base32(encrypt(payload, private_key))
    if not verify_license_key(license_key, name, public_key):
        print("License key verification failed!")
        return
    print("License key generated successfully!")
    print(f"License key: {format_license_key(license_key)}")

if __name__ == "__main__":
    main()
```
