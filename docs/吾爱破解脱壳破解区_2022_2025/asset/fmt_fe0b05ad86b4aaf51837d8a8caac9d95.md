
```
import requests
import ddddocr

headers = {
}

json_data = {
    'extend_param': quote(extend_param),
    'appid': '100032497',
    'business_site': 'crm_primelogin_h5_pic',
    'version': '1.0.6',
    'dimensions': quote(dimensions),
    'sign': sign,
}

response = requests.post(‘脱敏处理', headers=headers, json=json_data).json()
rid = response['result']['rid']
token = response['result']['token']
bg_pic = base64.b64decode(response['result']['risk_info']['process_value']['jigsaw_image'])
sc_pic = base64.b64decode(response['result']['risk_info']['process_value']['processed_image'])
with open('bg_pic.png', 'wb') as f:
    f.write(bg_pic)
with open('sc_pic.png', 'wb') as f:
    f.write(sc_pic)
det = ddddocr.DdddOcr(det=False, ocr=True, show_ad=False)
with open('sc_pic.png', 'rb') as f:
    target_bytes = f.read()
with open('bg_pic.png', 'rb') as f:
    background_bytes = f.read()
rest = det.slide_match(target_bytes, background_bytes, simple_target=True)
value = rest['target'][0]
print('识别到距离', value)
```

接下来就是处理校验接口，经过我们分析，`token`和`rid`是前置接口传过来的，所以我们处理的参数不多，还是分析他的逻辑

![](https://attach.52pojie.cn/forum/202507/07/223702i199xucs3uabur3c.png)

image.png (137.27 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NzcxMnwzMmM1NjcxZXwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcxMnwzMmM1NjcxZXwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

校验接口分析

2025-7-7 22:37 上传

看着逻辑都是一样的，我们都跟进去加密函数去看看情况

![](https://attach.52pojie.cn/forum/202507/07/223833vz7l7lsbcjbhlm5z.png)

image.png (20.72 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NzcxM3xkMjdlNDkzOHwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcxM3xkMjdlNDkzOHwxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

aes

2025-7-7 22:38 上传

好家伙，直接复用了，所以说还是前置接口那一套的逻辑，唯一的区别这里加密的是轨迹信息

![](https://attach.52pojie.cn/forum/202507/07/224003dhs66yjzfatjhtdi.png)

image.png (100 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc4NzcxNHw5NTQ1OTgyM3wxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NzcxNHw5NTQ1OTgyM3wxNzkzNTcwODQwfDE2ODA4NnwyMDQ0MDc1&nothumb=yes)

轨迹加密

2025-7-7 22:40 上传

后面一个`md5`的拼接{:1_918:}这样一个滑块就ok了

## 三、总结

所谓工欲善其事必先利其器，`ast`技术更好的帮助分析（我的`ast`还是很菜），这种虽然简单，但是看着麻烦，做一定的处理更加方便，整体来说还是一个入门级的案例。轨迹算法还是建议大佬们借鉴论坛里的大佬{:1_932:}我去准备明天考试了{:1_892:}
