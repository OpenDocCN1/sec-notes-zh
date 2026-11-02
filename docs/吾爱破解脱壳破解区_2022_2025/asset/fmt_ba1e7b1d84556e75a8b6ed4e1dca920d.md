
# 加解密分析

## 解密代码

```python
import base64
def atob(string):
    return base64.b64decode(string.encode()).decode()

def decrypt(string, res=''):
    temp = atob(string)
    for i in range(len(temp)):
        res += chr(ord(temp[i]) ^ ord("nmmeccpan"[i % 9]))
    return atob(res)

print(decrypt('DyU/VQArPVciF1QaPDRXBTgDKB03LTYWKVNXGiFeKBUuKAVfDTUJFy1QABc9KTc6BFMbGQAmLDU4DlNSIyYNQQ4lAQgzJAZaISkKWA=='))
#https://pan.baidu.com/s/1wuqwk7zoHfVkLbhpWRM5Hg?pwd=8888
```

## 另一个加密链接

还有另一个加密链接，本来看不看无所谓，但是传入解密代码它报错耶，这不能忍，这个密文链接是点击跳转另一个页面，直接 hook `window.open = (e) => {debugger}` 运行，查看`e`值。

![](https://attach.52pojie.cn/forum/202304/27/161012igrjj38rxoxeen3o.png)
`cbaf21b0b8408470da9170506f8b6a9.png` (24.39 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=MjYxMDg3OHw2ZDRlMTRiNXwxNzkzNTczNjAyfDE2ODA4NnwxNzc5Mzgw&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDg3OHw2ZDRlMTRiNXwxNzkzNTczNjAyfDE2ODA4NnwxNzc5Mzgw&nothumb=yes)
2023-4-27 16:10 上传

## 结果

结果：`python`直接请求`url = location.origin + '/open/other/' + 那串不是密文的密文`直接跳转。

![](https://attach.52pojie.cn/forum/202304/27/162724e1d8nc11dbd0zifc.png)
`fbdc1b372e793cd6a890213b91b8e7b.png` (26.7 KB, 下载次数: 0)

[forum.php?mod=attachment&aid=MjYxMDg4MnxmMDE2MDUxN3wxNzkzNTczNjAyfDE2ODA4NnwxNzc5Mzgw&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxMDg4MnxmMDE2MDUxN3wxNzkzNTczNjAyfDE2ODA4NnwxNzc5Mzgw&nothumb=yes)
2023-4-27 16:27 上传

## 完结

分析时间 < 写文章时间
