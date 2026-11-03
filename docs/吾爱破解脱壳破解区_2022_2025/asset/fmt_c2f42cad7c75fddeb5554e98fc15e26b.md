
```python
headers = {
"accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
"accept-language": "zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6",
"sec-ch-ua": "\"Chromium\";v=\"122\", \"Not(A:Brand\";v=\"24\", \"Microsoft Edge\";v=\"122\"",
"sec-ch-ua-mobile": "?0",
"sec-ch-ua-platform": "\"Windows\"",
"sec-fetch-dest": "document",
"sec-fetch-mode": "navigate",
"sec-fetch-site": "none",
"sec-fetch-user": "?1",
"upgrade-insecure-requests": "1",
"cookie": "down_ip=1"
}
response = requests.get(full_url, headers=headers, allow_redirects=False)
print(response.headers['Location'])
```

运行后终于找到了最终的直链。

![](https://attach.52pojie.cn/forum/202403/17/090901og417az10sba33a1.png)

`{41986CD7-C865-48e8-83F7-FCE7641F62D5}.png` (173.14 KB, 下载次数: 4)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mjg1M3w1YWQxMDA3YXwxNzkzNTcyNTY4fDE2ODA4NnwxOTAxODg0&nothumb=yes)

2024-3-17 09:09 上传

## 完整程序

```python
import requests
import re
import json

def re_domain(url):
    pattern_domain = r"https?://([^/]+)"
    match = re.search(pattern_domain, url)
    if match:
        domain = match.group(1)
        return domain
    else:
        return None

url = "https://wwt.lanzouu.com/icuiF1o31f8d"
domain = re_domain(url)
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36 Edg/121.0.0.0"
}
response = requests.get(url, headers=headers)
iframe_pattern = re.compile(r'')
matches = iframe_pattern.findall(response.text)
response2 = requests.get(f"https://{domain}{matches[1]}", headers=headers)
pattern = r"'sign'\s*:\s*'([^']+)'"
sign = re.search(pattern, response2.text).group(1)
pattern2 = r"url\s*:\s*'([^']+)'"
url2 = re.search(pattern2, response2.text).group(1)
data = {
    'action': 'downprocess',
    'signs': '?ctdf',
    'sign': sign,
    'websign': '',
    'websignkey': 'bL27',
    'ves': 1
}
headers = {
    "Referer": matches[1],
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36 Edg/121.0.0.0"
}
response3 = requests.post(f"https://{domain}{url2}", headers=headers, data=data)
data = json.loads(response3.text)
full_url = data['dom'] + "/file/" + data['url']
headers = {
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "accept-language": "zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6",
    "sec-ch-ua": "\"Chromium\";v=\"122\", \"Not(A:Brand\";v=\"24\", \"Microsoft Edge\";v=\"122\"",
    "sec-ch-ua-mobile": "?0",
    "sec-ch-ua-platform": "\"Windows\"",
    "sec-fetch-dest": "document",
    "sec-fetch-mode": "navigate",
    "sec-fetch-site": "none",
    "sec-fetch-user": "?1",
    "upgrade-insecure-requests": "1",
    "cookie": "down_ip=1"
}
response4 = requests.get(full_url, headers=headers, allow_redirects=False)
print(response4.headers['Location'])
```

## 传送门

[https://www.52pojie.cn/thread-1904710-1-1.html](https://www.52pojie.cn/thread-1904710-1-1.html)
