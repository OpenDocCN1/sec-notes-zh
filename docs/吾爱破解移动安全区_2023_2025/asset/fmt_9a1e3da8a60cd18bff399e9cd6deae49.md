
# Frida Hook 实战示例

## Hook `com.msec.MSecClient` 类

hook `com.msec.MSecClient` 类，查看类中的方法，找到要hook的方法

```
android hooking watch class com.msec.MSecClient
```

![](https://attach.52pojie.cn/forum/202308/18/150929xamstrkk9xfk4xnn.png)

## Hook 方法

hook 以上两个方法

```
android hooking watch class_method com.msec.MSecClient._ts_getHeaderKey --dump-args --dump-backtrace --dump-return
android hooking watch class_method com.msec.MSecClient._ts_getRequestHeader --dump-args --dump-backtrace --dump-return
```

去 app 中请求接口，即可看到 hook 结果

![](https://attach.52pojie.cn/forum/202308/18/150927sgqnphtkegleq83q.png)

根据 hook 结果可知 `_ts_getRequestHeader()` 方法的返回值即是 `tzRgz52a` 签名

## 3. 使用 Frida Hook 并注入

`hookjs` 如下：

```
function get_yjj_tzRgz52a() {
    console.log('script 加载成功')
    Java.perform(function () {
        console.log('hook test')
        var MainActivity = Java.use('com.msec.MSecClient');
        MainActivity._ts_getRequestHeader.implementation = function (x, y, z) {
            // 这里开始是原函数的逻辑
            console.log(x)
            console.log(y)
            console.log(z)

            // 这里对原函数的参数重新赋值
            var vaule = this._ts_getRequestHeader(x, y, z);

            // 返回原函数的结果
            console.log(vaule);
            return vaule

        }
    })
}

setTimeout(get_yjj_tzRgz52a)
```

使用以下命令注入：

```
frida -U -l 1.js 中国药品监管
```

注意：这里注入后面写的是 `name` 而并非包名

如图所示，注入成功

![](https://attach.52pojie.cn/forum/202308/18/151156efxdmdnhpfqhduuu.png)

在 app 中重新请求接口，hook 结果如下

![](https://attach.52pojie.cn/forum/202308/18/151159wcto8tscirzlgy8z.png)

`0 0 http://mobile.nmpa.gov.cn/datasearch/QueryList?tableId=25&searchF=Quick%20SearchK&pageIndex=4&pageSize=15`

分别对应方法的参数 `x` `y` `z`，多次请求后发现 `0 0` 是固定的，即只有参数 `z` 是变量，`z` 就是请求的数据接口链接

## 4. RPC 主动调用

确定方法的参数后即可使用 RPC 将被动调用改为主动调用

稍微改动下 hook 的 js

```
function getYjjTzRgz52a(x, y, z) {
    var vaule;
    Java.perform(function () {
        Java.choose('com.msec.MSecClient', {
            onMatch: function (instance) {
                vaule = instance._ts_getRequestHeader(x, y, z);
            },
            onComplete: function () {
                console.log('_ts_getRequestHeader()调用完成')
            }
        })
    })
    return vaule
}

rpc.exports = {
    getyjjtzrgz52a: getYjjTzRgz52a
}
```

RPC 使用 `python` 实现，代码如下：

```
#!/usr/bin/env python3

# -*- coding: utf-8 -*-

# 

# Copyright (c) 2022 LouShimin, Inc. All Rights Reserved

#

# @version : 1.0

# @AuThor  : LouShimin

# @Time    : 

# @FileName: yjj.py

# @desc    :

import frida

def on_message(message, data):

    if message['type'] == 'send':

        print(message['payload'])

    elif message['type'] == 'error':

        print(message['stack'])

device = frida.get_usb_device()

process = device.attach('中国药品监管')

with open('./yjj_end_rpc.js', encoding='utf-8') as f:

    jscode = f.read()

script = process.create_script(jscode)

script.on('message', on_message)

script.load()

def get_yjj_rpc():

    rpc = script.exports

    return rpc

def get_yjj_tzRgz52a(page):

    rpc = get_yjj_rpc()

    a1 = rpc.getyjjtzrgz52a(0, 0,

                             'http://mobile.nmpa.gov.cn/datasearch/QueryList?tableId=25&searchF=Quick%20SearchK&pageIndex={}&pageSize=15'.format(

                                 page))

    print(a1)

if __name__ == '__main__':

    while True:

        page = input('请输入页码：')

        print(page)

        get_yjj_tzRgz52a(page)
```

运行程序后输入页码，返回值即为 `tzRgz52a` 签名

![](https://attach.52pojie.cn/forum/202308/18/151422gkpxvbiz34z51kzo.png)

## 5. Sanic + RPC 远程调用

接口文件如下：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# 
# Copyright (c) 2022 LouShimin, Inc. All Rights Reserved
#
# @Version : 1.0
# @Author  : LouShimin
# @Time    : 
# @FileName: sanic_app.py
# @Desc    :
from sanic import Sanic
from sanic.response import json as response_json

from commons.frida_rpc.yjj_rpc import get_yjj_rpc

app = Sanic(__name__)

yjj_rpc = get_yjj_rpc()

@app.route("/getSignFromJni", methods=['POST'])
async def req_proxy_data(request):
    res = request.json
    yjj_url = res['yjj_url']

    tzRgz52a = yjj_rpc.getyjjtzrgz52a(0, 0, yjj_url)
    print(tzRgz52a)
    res_data = {
        'msg': 'success',
        'code': 200,
        'tzRgz52a': tzRgz52a,
        'yjj_url': yjj_url
    }
    return response_json(res_data)

if __name__ == '__main__':
    app.run("0.0.0.0", port=16383, debug=True, workers=30)
```

改造并引入三、4 中的 `getyjjtzrgz52a()` 方法接口通过接口形式调用：

![](https://attach.52pojie.cn/forum/202308/18/151424rzitmpbdp2hzcpzr.png)
