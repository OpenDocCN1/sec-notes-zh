
[DESHelper界面](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzExMHxlMzJjZGQwNXwxNzkzNTY5MTM3fDE2ODA4NnwxODQwOTk1&nothumb=yes)

2023-10-6 12:27 上传

很显然，秘钥是处理之后的字符串，翻译成 python 就是 `[-1:0:-1]`。

## 五、实现

从网上找一个 DES 加解密的类：

```python
from Crypto.Cipher import DES
from binascii import b2a_base64, a2b_base64

class PrpCrypt(object):
    def __init__(self, key):
        self.key = key.encode('utf-8')
        self.mode = DES.MODE_ECB
        
    def changeKey(self, key):
        self.key = key.encode('utf-8')
        
    def encrypt(self, text):
        text = text.encode('utf-8')
        cryptor = DES.new(self.key, self.mode)
        length = 16
        count = len(text)
        if count  length:
            add = (length - (count % length))
            text = text + ('\07' * add).encode('utf-8')
            print(text)
        self.ciphertext = cryptor.encrypt(text)
        return b2a_base64(self.ciphertext)

    def decrypt(self, text):
        cryptor = DES.new(self.key, self.mode)
        plain_text = cryptor.decrypt(a2b_base64(text))
        return plain_text.decode("utf-8").rstrip("\01").rstrip("\03")
```

### 1. 搜索功能

发个请求测试一下：

```python
import requests

headers = {
    'Cache-Control': 'public,max-age=60',
    'Content-Type': 'application/json; charset=UTF-8',
    'Host': '43.248.116.78:20256',
    'Connection': 'Keep-Alive',
    'User-Agent': 'okhttp/4.10.0',
}

data = {
    'key': '擅长捉弄的高木同学',
    'appChannel': 'normal',
    'appKey': 'com.aster.zhbj',
    'appVersion': 'v1.10.5',
    'clientTime': 1696328622216,
    'deviceBrand': 'Xiaomi',
    'deviceType': '22041211AC',
    'ipAddr': '192.168.0.221',
    'netType': 'WIFI',
    'platform': 0,
    'sign': '',
    'systemVersion': '12',
    'userId': '1526xxxxxxxc8f70abce2',
    'uuid': 'df82xxxxxxx65d407e64ddf',
    'versionCode': 48
}

response = requests.post(
    'http://43.248.116.78:20256/api/novel/search/associate', headers=headers, json=data).json()

# 秘钥处理方法14239,14339
token = response['safety']["token"][0][-1:0:-1]

cry = PrpCrypt(token)  # 初始化密钥

decode = cry.decrypt(response["safetyData"])
decode = json.loads(decode)

print(decode)
```

看看输出结果（部分）：

```
[
    {
        "key": "擅长捉弄的高木同学",
        "relationType": 1,
        "relationId": "af0580069e7031e3f03f27fb4ff26cdb",
        "extra": "{"authorName":"山本崇一朗"}"
    },
    {
        "key": "擅长捉弄的（原）高木同学",
        "relationType": 1,
        "relationId": "9d8b4dad7954734a471cb21452e917e0",
        "extra": "{"authorName":"山本崇一朗"}"
    }
]
```

很好，解密成功了，这是一个好的开始。

### 2. 获取漫画详情

搜索实现了，接下来就是获取漫画详情了：

刚才结果里有个很重要的数据字段 `relationId`，记一下它的值：`af0580069e7031e3f03f27fb4ff26cdb`。

继续抓包：

![](https://attach.52pojie.cn/forum/202310/06/122134m25ms2assm1jpavj.jpg)

`漫画详情界面.jpg` (200.34 KB, 下载次数: 0)

[漫画详情界面](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzA4OXw2Y2YyYWY3N3wxNzkzNTY5MTM3fDE2ODA4NnwxODQwOTk1&nothumb=yes)

2023-10-6 12:21 上传

![](https://attach.52pojie.cn/forum/202310/06/122150r0p19o2ano329qt7.jpg)

`漫画详情_抓包记录界面.jpg` (234.2 KB, 下载次数: 0)

[漫画详情_抓包记录界面](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzA5MHw5NzNmZmI0ZnwxNzkzNTY5MTM3fDE2ODA4NnwxODQwOTk1&nothumb=yes)

2023-10-6 12:21 上传

我们看到第一条数据包（最下面）的 `url` 内就有刚才的 `relationId`。

点进数据包，仔细看一下响应体：

![](https://attach.52pojie.cn/forum/202310/06/122202tnwwergg577hcirn.jpg)

`漫画详情_数据包.jpg` (281.26 KB, 下载次数: 0)

[漫画详情_数据包](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzA5MXxjMzRhOWFmOXwxNzkzNTY5MTM3fDE2ODA4NnwxODQwOTk1&nothumb=yes)

2023-10-6 12:22 上传

这次居然出现了两个 `token`。不慌，我们再看一下 `safety` 里面的 `type` 字段：14439

按照刚才的方法，最终定位到：

![](https://attach.52pojie.cn/forum/202310/06/122215lg997qpu33ygeqqp.png)

`DES2界面.png` (61.76 KB, 下载次数: 0)

[DES2界面](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzA5MnxiZDBiYzgwYXwxNzkzNTY5MTM3fDE2ODA4NnwxODQwOTk1&nothumb=yes)

2023-10-6 12:22 上传

这里又处理了一下 `token`，用 python 实现就是：

```python
response = requests.get("http://43.248.116.76:20131/api/novel/book/info/9d8b4dad7954734a471cb21452e917e0.json", headers=headers, data=data)
token = response.json()['safety']["token"]
token = token[0][::2] + token[1][1::2]
cry = PrpCrypt(token)
decode = cry.decrypt(response.json()["safetyData"]).encode(
    'utf-8').decode('unicode_escape')
print(decode)
```

运行看一下结果（部分）：

```
{
    "bookId": "af0580069e7031e3f03f27fb4ff26cdb",
    "bookName": "擅长捉弄的高木同学",
    "authorName": "山本崇一朗",
    "categoryName": "恋爱",
    "intro": "【此漫画的翻译由版权方提供】因为被对方捉弄所以要想尽办法捉弄回来，这不是理所当然的嘛！气定神闲地捉弄人的高木同学和总是计划失败被捉弄到满面通红的西片，在班上邻座的两人似乎有更多机会互相搞小动作，可是真的仅仅只是想要捉弄对方而已吗？这是擅长捉弄人的女孩子和傻乎乎被捉弄了之后一本正经想要“报仇”的男孩子，他们之间轻松愉快的故事。不过，好像也不仅仅是这样哦……\n每一次开场读者仿佛就能“看到结局”，但还是会让人忍不住看下去。大家一起为西片加油吧！",
    "tags": [
        "搞笑",
        "校园",
        "恋爱",
        "都市"
    ]
}
```

### 3. 获取章节详情

```python
res = requests.get(
f"http://43.248.116.76:20131/api/novel/book/chapters/af0580069e7031e3f03f27fb4ff26cdb.json", headers=headers).json()
token = res['safety']["token"][0][-1:0:-1]
cry = PrpCrypt(token)
result = cry.decrypt(res['safetyData']).replace("\x08", "")
    #.encode().decode('unicode_escape')
print(json.loads(result))
```

在运行查看结果（部分）：

```
{
    "chapters": [
        {
            "chapterId": "YWYwNTgwMDY5ZTcwMzFlM2YwM2YyN2ZiNGZmMjZjZGJfcGx1dG9fNGU3ODViMDFiMWNkNjE5ZWNiZDQwNmQwODc4OTU1YTc=",
            "chapterName": "001 橡皮擦",
            "chapterSort": 1,
            "nextChapterId": "YWYwNTgwMDY5ZTcwMzFlM2YwM2YyN2ZiNGZmMjZjZGJfcGx1dG9fYWYyNGEwMTgwOGU0ZjI0M2VlM2NhZDhmNGEwNzk1MTg=",
        },
        {
            "chapterId": "YWYwNTgwMDY5ZTcwMzFlM2YwM2YyN2ZiNGZmMjZjZGJfcGx1dG9fYWYyNGEwMTgwOGU0ZjI0M2VlM2NhZDhmNGEwNzk1MTg=",
            "chapterName": "002 泳池",
            "chapterSort": 2,
            "nextChapterId": "YWYwNTgwMDY5ZTcwMzFlM2YwM2YyN2ZiNGZmMjZjZGJfcGx1dG9fMmE3NWZiNjhhZjg0NTUxYzRhZTlmNDNmMjg1YzE2ZjU=",
        }
    ]
}
```

### 4. 获取章节图片列表

继续抓包，发现这里的 `token` 只有一个元素，那么密钥的处理方式就和搜索功能是一样的了。

```python
res = requests.get(  f"http://43.248.116.76:20131/api/novel/book/chapters/images/af0580069e7031e3f03f27fb4ff26cdb/YWYwNTgwMDY5ZTcwMzFlM2YwM2YyN2ZiNGZmMjZjZGJfcGx1dG9fNGU3ODViMDFiMWNkNjE5ZWNiZDQwNmQwODc4OTU1YTc=.json").json()

token = res['safety']["token"][0][-1:0:-1]

cry = PrpCrypt(token)

data = json.loads(cry.decrypt(res['safetyData']))["items"]

print(data)
```

再看一下结果（部分）
