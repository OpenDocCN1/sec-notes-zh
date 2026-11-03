
https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NDU2OXwxM2Q5MWNjMnwxNzkzNTY4NjEyfDE2ODA4NnwxODg4MDE4&nothumb=yes

2024-2-4 20:39 上传

到此为止，我们已经通过使用 Frida Hook 了请求和响应，现在我们就可以很方便地进行分析了。

通过同样的方法，我们也可以获得请求直播源地址时所有参数的含义了。

也许这时候有人会说，为什么不直接 Hook App 中常用的 `okhttp` 模块来进行抓包呢？其实我也尝试过……

只要 Hook `okhttp` 的模块，就和上面提到的 Android 抓包一样，提示无网络 :'(weeqw ，不知道是否有大佬能解答下{:1_889:}

## 4. 通过 Frida 创建 `getCKey` API

上面的所有案例中，Frida 都是通过被动调用的方式来执行的，那么它可以主动调用对应的方法吗？

这样就可以直接使用 Frida 创建一个 `api` 来获取 `cKey`，然后使用 Python 来进行更多操作。

Frida 官方也提供了这样一个案例：

https://github.com/frida/frida-python/blob/ebd797e4bc248b8d895d68ebf244a34744cb3ea9/examples/rpc.py

简单说，就是通过 frida 提供的 `rpc.exports`，来导出方法，以方便 python 可以随时调用。

`getCkey.js`

```javascript
// getCkey.js 只需要输入时间戳和cnlid即可
function ckey(timestamp, cnlid){
    var result = '';
    Java.perform(function(){
        var CKeyFacade = Java.use("com.tencent.qqlive.tvkplayer.vinfo.ckey.CKeyFacade");
        var guid = '9d3644141c7047acb072f6343cfaa433';
        var appVer = 'V8.12.1034.4382';
        var platform = '4330303';
        var sdtfrom = 'v5028';
        var _iArr = [0,0,0];
        var _i = 3;
        var _str6 = '';

        result = CKeyFacade.getCKey(guid, timestamp, cnlid, appVer, platform, sdtfrom, _iArr, _i, _str6);

    });
    return result
}

rpc.exports = {
    ckey: ckey
}
```

Python 脚本：

```python
import frida, sys, time

# 读取js脚本，也可以直接以字符串的形式写在python脚本中
with open('getCkey.js', 'r+') as f:
    js_code = f.read()

# 通过USB获取设备
device = frida.get_usb_device()

# attach进程
session = device.attach('某视频')

# 加载脚本
script = session.create_script(js_code)
script.load()

# 获取frida导出的函数
api = script.exports_sync

# 可以直接调用ckey函数，来获取CKey
result = api.ckey(int(time.time()), '2000210103')
print(result)
```

### 远程调用 Frida

在实际应用的环境中，手机一直用 USB 连着电脑似乎也不太靠谱，而 Frida 也提供了远程调用的方式。

让 `frida-server` 监听 `0.0.0.0`：

```shell
./frida-server64 -l 0.0.0.0
```

那么之前注入脚本的命令也变成了：

```shell
frida -H 10.10.10.170 -n 某视频 -l test.js
```

在 python 中调用的方式，也变成了：

```python
device = frida.get_device_manager().add_remote_device('10.10.10.170')
session = device.attach('某视频')
script = session.create_script(js_code)
script.load()
```

甚至也可以在[相关帖子](https://www.52pojie.cn/thread-661779-1-1.html)或者树莓派等设备中安装安卓系统作为专门的 `api` 服务器。

### 非实体 Android 系统

若在树莓派中的 Android 系统，主要有两种选择：

- EmteriaOS（注册账户后即可下载安装程序，通过安装程序可刷写系统到 TF 卡中，非商业版 8 小时重启一次）
- LineageOS（由 konstakang 大佬魔改而来，下载镜像刷写到 TF 卡中即可）

其他方案也可以参考大佬分享的安卓容器化部署方案。

不管使用哪种方式，都需要自动启动应用以及 frida，可以使用 Script Manager 来实现。

![](https://attach.52pojie.cn/forum/202402/04/204339liyjhqz92big998d.png)

`微信截图_20240204193413.png` (74.33 KB, 下载次数: 2)

https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NDU3MXxjMmQ4YWE0ZXwxNzkzNTY4NjEyfDE2ODA4NnwxODg4MDE4&nothumb=yes

2024-2-4 20:43 上传

## 5. 整理脚本

最后可以将上述所有的整理成脚本即可根据自己的需求去获取各个直播流的地址了。

`WaterRequests.py`

```python
import requests, time, logging
from functools import wraps
from requests.exceptions import ConnectTimeout, ConnectionError

class WaterRequest():
    def __init__(
        self, 
        headers={}, 
        config={}, 
        logger=logging.getLogger(__name__),
        retry_times=5,
        **kwargs
    ):
        self.session = requests.Session()
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.61 Safari/537.36',
            **headers
        }
        self.config = {
            'timeout': 300,
            'verify': True,
            **config
        }
        self.logger = logger
        self.session.headers.update({**self.headers})
        self.retry_times = retry_times

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

    def retry_decorator(func):
        @wraps(func)
        def wrapped_function(*args, **kwargs):
            _self = [*args][0]
            result = None
            for i in range(_self.retry_times):
                if i > 0:
                    _self.logger.warning(f'正在进行第 {i} 次重试...')
                try:
                    result = func(*args, **kwargs)
                    break
                except (ConnectionError, ConnectTimeout) as e:
                    _url = {**kwargs}.get('url', None)
                    if _url is None:
                        if len([*args])>1:
                            _url = [*args][1]
                    if _url is not None and len(_url)>20:
                        _url = f'{_url[:10]}...{_url[-10:]}'
                    if type(e) == ConnectionError:
                        _self.logger.warning(f'[ConnectionError][URL:{_url}]')
                    elif type(e) == ConnectTimeout:
                        _self.logger.warning(f'[ConnectTimeout][URL:{_url}]')
                    else:
                        _self.logger.warning(f'[未知错误][URL:{_url}]')
                    if i == _self.retry_times-1:
                        _self.logger.error(f'已重试 {i+1} 次，达到最大重试次数，访问失败')
                        break
                    else:
                        _self.logger.info(f'3秒后进行第 {i+1} 次重试')
                        time.sleep(3)
                        continue
            return result
        return wrapped_function

    def updateHeaders(self, new_header:dict):
        self.headers.update({**new_header})
        _s = self.session
        _s.headers.update({**new_header})

    @retry_decorator
    def get(self, url, query:dict=None, data:dict=None):
        _s = self.session
        _r = _s.get(url, params=query, data=data, **self.config)
        return _r.json()

    @retry_decorator
    def post(self, url, json:dict=None, query:dict=None, data:dict=None):
        _post_data = {}
        if data is not None:
            _post_data = {
                'data': data
            }
        if json is not None:
            _post_data = {
                'json': json
            }
        _s = self.session
        _r = _s.post(url, params=query, **_post_data, **self.config)
        return _r.json()

    @retry_decorator
    def delete(self, url, json:dict=None, query:dict=None, data:dict=None):
        _s = self.session
        _r = _s.delete(url, json=json, params=query, data=data, **self.config)
        return _r.json()

    def close(self):
        return self.session.close()
```

`config.py`
