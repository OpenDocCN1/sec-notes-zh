
```
import json
import re
import uuid
import logging
import random
import requests
import base64
from Crypto.Cipher import PKCS1_v1_5 as Cipher_pksc1_v1_5, AES
from Crypto.PublicKey import RSA
from Crypto.Util.Padding import unpad
from pyDes import *

def get_Rsa(data):
    key = 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwIbK7LEP3b5y0UabUG9ZD71GUGTOf0lPPDnrDE1+6DK+t/giyQQfBHqeCHe7MQtoGFzxsWfZyWlEoWtB1cO3TzP+f8lm4PxpRAJ7uTaDepUyBltsj264D8xdaCweqNRfxu3CpKDAmRaN7HGxZLP6nVITy66SdZtS7vFZIj3uehKRW1VNqGUm+S4wtmV+IGHJW1of8mIXe7jDGzXtwh2aF65Gd2IaHlDzqTT0dr6xgJaRrSgMrquVE66pFZhvebn5bHfEZws3CJ34Mx5Kps/EQbWn6w8MpKs85Kl+ZMJ+CnJ6R66B2Zcio6U4Evn3sLp0KaGXuCItmZXqTD3p6Hsj7QIDAQAB'
    public_key = '-----BEGIN PUBLIC KEY-----\n' + key + '\n-----END PUBLIC KEY-----'
    rsakey = RSA.importKey(public_key)
    cipher = Cipher_pksc1_v1_5.new(rsakey)
    cipher_text = base64.b64encode(cipher.encrypt(data.encode()))
    return cipher_text.decode()

def get_Des(res):
    Des_Key = "1a2s3d%*"
    Des_IV = "%*1a2s3d"
    k = des(Des_Key, CBC, Des_IV, pad=None, padmode=PAD_PKCS5)
    b64str = base64.b64decode(res)
    return k.decrypt(b64str).decode()

def get_AES(data):
    Aes_Key = "1a2s3d%*1a2s3d%*"
    Aes_IV = "%*1a2s3d%*1a2s3d"
    cryptor = AES.new(key=Aes_Key.encode(), mode=AES.MODE_CBC, iv=Aes_IV.encode())
    dekey = cryptor.decrypt(base64.b64decode(data))
    return unpad(dekey, 16).decode()

def get_Token(url):
    res = requests.get(url=url).text
    token=re.findall('(.+?)',res)[0]
    #print(token)
    #print(res)
    return get_kv(token)
```

3> to be continued

就先这样了，后面的有机会在写，写完这个，摸鱼生活正式结束了，给自己放两周假
