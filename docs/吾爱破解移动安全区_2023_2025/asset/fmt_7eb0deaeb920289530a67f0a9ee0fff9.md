
@7U$aPOE@$Version1_appidapp.iphone_timestamp1686473693autohomeuaiPhone  12.5.5  autohome    11.33.5 iPhoneisCheckModeratorsRemote1isapp1logincode%3123456reffersessionid744ec98679b59e66a1fdc2deb77e7ee610812039showmob1userpwdE10ADC3949BA59ABBE56E057F20F883Evalidcodeiaeh@7U$aPOE@$

A40B768AA55F204329C7769C98BB1C69

```python
usn = "123456"
pwd = "123456"
en_md5 = lambda x: hashlib.md5(x.encode()).hexdigest()

def gen_sign(params):
    s = f"@7U$aPOE@$Version{params['Version']}_appid{params['_appid']}_timestamp{params['_timestamp']}autohomeua{params['autohomeua']}isCheckModeratorsRemote{params['isCheckModeratorsRemote']}isapp{params['isapp']}logincode{params['logincode']}reffer{params['reffer']}sessionid{params['sessionid']}showmob{params['showmob']}userpwd{params['userpwd']}validcode{params['validcode']}@7U$aPOE@$"
    return en_md5(s).upper()

if __name__ == "__main__":
    params = {
        "Version": 1,
        "_appid": "app.iphone",
        # "_timestamp": int(time.time()),
        "_timestamp": "1686473693",
        "autohomeua": "iPhone   12.5.5  autohome    11.33.5 iPhone",
        "isCheckModeratorsRemote": 1,
        "isapp": 1,
        "logincode": f"%3{usn}",
        "reffer": "",
        "sessionid": "744ec98679b59e66a1fdc2deb77e7ee610812039",
        "showmob": 1,
        "userpwd": en_md5(pwd).upper(),
        "validcode": "iaeh"
    }
    _sign = gen_sign(params)
    print(f"{params=}\n{_sign=}")
```

## 小结
登录接口的 `userpwd` 和 `_sign` 已经分析完了。比较简单，就一个大写`md5`，还没有任何魔改，熟悉算法的佬应该很快就反应过来的。可惜俺不是🤣，俺开始还用`Reveal`查找登录视图，再`hook`视图分析堆栈，徒耗时间，后面才反应过来，哈哈哈哈（对算法敏感度不够，后知后觉）。
