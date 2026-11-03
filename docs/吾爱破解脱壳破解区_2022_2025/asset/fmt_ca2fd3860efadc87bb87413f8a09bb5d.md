
```python
import hashlib
import json

import requests
import re
import websockets
import asyncio
# 设置 测速的网址
http_url = "https://www.52pojie.cn"

gbl_taskId = None
gbl_taskToken = None

def get_task_id():
    url = "https://www.itdog.cn/http/"
    # 下面的data里面有一个参数"host_s"建议自己替换下，有点小问题，但是应该也不影响。
    data = f"line=&host={http_url}&host_s={http_url}&check_mode=fast&ipv4=&method=get&referer=&ua=&cookies=&redirect_num=5&dns_server_type=isp&dns_server="
    headers = {
        'upgrade-insecure-requests': '1',
        'origin': 'https://www.itdog.cn',
        'sec-fetch-dest': 'document',
        'sec-fetch-mode': 'navigate',
        'priority': 'u=0, i',
        'referer': 'https://www.itdog.cn/http/',
        'sec-ch-ua': '',
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7',
        'sec-ch-ua-mobile': '?0',
        'sec-fetch-site': 'same-origin',
        'cache-control': 'no-cache',
        'sec-ch-ua-platform': '',
        'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8',
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36',
        'sec-fetch-user': '?1',
        'content-type': 'application/x-www-form-urlencoded',
        'pragma': 'no-cache',
        # 'cookie': ''
    }
    response = requests.post(url, data=data, headers=headers)
    print(response.text,response.status_code)
    match = re.search(r"var task_id='(.*?)';", response.text)
    if match:
        task_id = match.group(1)
        return True, task_id
    else:
        return False,""

async def connect_and_listen(uri, message_handler=None, extra_headers=None):
    """
    连接到指定的WebSocket URI并监听消息。

    :param uri: WebSocket服务器的URI（例如wss://www.example.com/websockets）
    :param message_handler: 一个可选的回调函数，用于处理接收到的消息
    :param extra_headers: 额外的HTTP头信息，如包含Cookie等
    """
    try:
        async with websockets.connect(uri, extra_headers=extra_headers) as websocket:
            print(f"Connected to {uri}")

            message = {
                "task_id": gbl_taskId,
                "task_token": gbl_taskToken
            }
            # 如果需要发送初始消息给服务器，可以在这里添加
            await websocket.send(json.dumps(message))
            print(json.dumps(message))
            while True:
                message = await websocket.recv()
                # print(f"Received message: {message}")

                # message_handler回调函数，调用它来处理消息
                if message_handler:
                    message_handler(message)

    except websockets.ConnectionClosed:
        print("Connection closed unexpectedly")

def start_websocket_listener(uri, message_handler=None):
    """
    启动WebSocket监听器。
    :param uri: WebSocket服务器的URI
    :param message_handler: 处理接收到消息的回调函数
    """
    extra_headers = {
        # 'cookie': ""
    }
    # 使用asyncio运行异步函数
    asyncio.get_event_loop().run_until_complete(connect_and_listen(uri, message_handler, extra_headers))

def websocket_return_message(message):
    json_obj = json.loads(message)
    print(json_obj)

if __name__ == '__main__':
    # 获取 task_id
    task_flag,task_id = get_task_id()
    if task_flag == False:
        print("获取task_id失败，请检查函数内部调用！")
        exit()
    # 获取 task_token
    task_token_str = "token_20230313000136kwyktxb0tgspm00yo5"
    task_token = hashlib.md5((task_id + task_token_str).encode('utf-8')).hexdigest()[8:24]
    gbl_taskId =  task_id
    gbl_taskToken = task_token
    print(f'{{"task_id":"{gbl_taskId}","task_token":"{gbl_taskToken}"}}')
    start_websocket_listener("wss://www.itdog.cn/websockets",websocket_return_message)
```

![](https://attach.52pojie.cn/forum/202503/21/163950etzcgald9tcgczlo.png)

`image.png` (311 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDU3Mnw1NWI0MjgwYXwxNzkzNTcxMTE3fDE2ODA4NnwyMDE3MzA0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDU3Mnw1NWI0MjgwYXwxNzkzNTcxMTE3fDE2ODA4NnwyMDE3MzA0&nothumb=yes)

2025-3-21 16:39 上传
