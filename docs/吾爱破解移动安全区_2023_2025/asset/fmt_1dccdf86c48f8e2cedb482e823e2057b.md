
```
import base64
import hashlib
import json
import os
import random
import string
import time
import uuid
from datetime import datetime

import requests
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import padding

# Base64格式的公钥
base64_public_key = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCOHifICBXyxzDSj9yOg9HzBMs/D0C0YS1ZrF95j6HQrrQu4zOzDyJc5hLgwcEmHE/A6k39phkSpeRqb9a+5AONCz6q0mP7z7xdzOwVXu7KZX+Ch0QU4NZutgi0IWwzCBAcOJ5+O2FxAj+O4z3Q45JtIlGWKNn+YPIixjxVsypN4QIDAQAB"

token = '202408051001001370193195139083'

def rsa_encrypt_long_data(plaintext):
    # 解码公钥
    public_key = serialization.load_der_public_key(base64.b64decode(base64_public_key))
    # 计算chunk_size
    key_size_in_bytes = public_key.key_size // 8  # 公钥大小，以字节为单位
    chunk_size = key_size_in_bytes - 11

    encrypted_chunks = []
    for i in range(0, len(plaintext), chunk_size):
        chunk = plaintext[i:i + chunk_size]
        encrypted_chunk = public_key.encrypt(
            chunk.encode(),
            padding.PKCS1v15()
        )
        encrypted_chunks.append(encrypted_chunk)
    encrypted_data = b''.join(encrypted_chunks)
    return base64.b64encode(encrypted_data).decode()

# 获取以毫秒为单位的Unix时间戳
def get_milliseconds_timestamp():
    timestamp = int(time.time() * 1000)
    return timestamp

def calculate_md5_string(input_string):
    # 创建一个MD5 hash对象
    md5_hash = hashlib.md5()

    # 更新hash对象
    md5_hash.update(input_string.encode('utf-8'))

    # 获取16进制表示的MD5值
    md5_digest = md5_hash.hexdigest()
    return md5_digest

def generate_random_string(length=6):
    # 大写字母和数字的字符集
    characters = string.ascii_uppercase + string.digits

    # 生成指定长度的随机字符串
    random_string = ''.join(random.choices(characters, k=length))

    return random_string

def generate_time_based_session_id():
    return str(uuid.uuid1())

def generate_session_id():
    return str(uuid.uuid4())

def get_time_now():
    # 获取当前时间
    now = datetime.now()

    # 将时间格式化为 'YYYY-MM-DD HH:MM:SS'
    formatted_time = now.strftime('%Y-%m-%d %H:%M:%S')
    return formatted_time

def requestAllCar(cipherTxt: str, nonceStr: str, sign: str, timestamp: int):
    # 设置请求的URL
    url = 'http://ipc.api.smallfeiyu.cn/aibg-car-compare-api/car/homeLoad.do'

    # 设置请求的参数
    params = {
        'productId': 'productId=746abe7c-796c-4ebb-a6c9-b287bf503da4',
        'vestId': '6dbfae2e-776e-47fd-872c-695dffc1935d',
        'channel': 'vivo',
        'osType': 'android',
        'version': '8',
        'cipherTxt': cipherTxt,
        'nonceStr': nonceStr,
        'sign': sign,
        'timestamp': timestamp,
        'token': token
    }

    data = {
        "app": "com.vhahbp.hfdakh",
        'sessionId': generate_session_id(),
        "remoteIp": "47.111.241.44",
        "remotePort": 80,
        "time": get_time_now(),
    }

    # 设置请求头
    headers = {
        'Host': 'ipc.api.smallfeiyu.cn',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/4.9.3',
    }

    # 发送POST请求
    response = requests.post(url, headers=headers, params=params, data=data)

    return response

def requestCar(seriesId: str, cipherTxt: str, nonceStr: str, sign: str, timestamp: int):
    # 设置请求的URL
    url = 'http://ipc.api.smallfeiyu.cn/aibg-car-compare-api/car/car.do'

    # 设置请求的参数
    params = {
        'seriesId': seriesId,
        'productId': 'productId=746abe7c-796c-4ebb-a6c9-b287bf503da4',
        'vestId': '6dbfae2e-776e-47fd-872c-695dffc1935d',
        'channel': 'vivo',
        'osType': 'android',
        'version': '8',
        'cipherTxt': cipherTxt,
        'nonceStr': nonceStr,
        'sign': sign,
        'timestamp': timestamp,
        'token': token
    }

    data = {
        "app": "com.vhahbp.hfdakh",
        'sessionId': generate_session_id(),
        "remoteIp": "47.111.241.44",
        "remotePort": 80,
        "time": get_time_now(),
    }

    # 设置请求头
    headers = {
        'Host': 'ipc.api.smallfeiyu.cn',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/4.9.3',
    }

    # 发送POST请求
    response = requests.post(url, headers=headers, params=params, data=data)

    # 打印响应
    return response

def requestDetailCar(request_body, cipherTxt: str, nonceStr: str, sign: str, timestamp: int):
    # 设置请求的URL
    url = 'http://ipc.api.smallfeiyu.cn/aibg-car-compare-api/car/carDetail.do'

    # 设置请求的参数
    params = {
        'productId': 'productId=746abe7c-796c-4ebb-a6c9-b287bf503da4',
        'vestId': '6dbfae2e-776e-47fd-872c-695dffc1935d',
        'channel': 'vivo',
        'osType': 'android',
        'version': '8',
        'cipherTxt': cipherTxt,
        'nonceStr': nonceStr,
        'sign': sign,
        'timestamp': timestamp,
        'token': token
    }

    # 设置请求头
    headers = {
        'Host': 'ipc.api.smallfeiyu.cn',
        'Accept-Encoding': 'gzip',
        'User-Agent': 'okhttp/4.9.3',
    }

    # 发送POST请求
    response = requests.post(url, headers=headers, params=params, json=request_body)

    # 打印响应
    return response

def delay_s(s: int):
    time.sleep(s)

if __name__ == '__main__':

    timestamp = get_milliseconds_timestamp()
    nonceStr = generate_random_string()
    # 首页请求
    plaintext1 = f'channel=vivo&nonceStr={nonceStr}&osType=android&productId=746abe7c-796c-4ebb-a6c9-b287bf503da4&sdkIntVersion=4056×tamp={timestamp}&token={token}&version=8&vestId=6dbfae2e-776e-47fd-872c-695dffc1935d&bizCodeAbc=79fa3d5f857cf66a&'
    cipherTxt1 = rsa_encrypt_long_data(plaintext1)
    sign = calculate_md5_string(cipherTxt1 + "BC56EAAB76C5492E")
    response1 = requestAllCar(cipherTxt1, nonceStr, sign, timestamp)
    if response1.status_code == 200:
        file = os.path.join('汽车比价大全', '总览.json')
        os.makedirs(os.path.dirname(file), exist_ok=True)
        with open(file, 'w', encoding='utf-8') as save_file:
            save_file.write(response1.text)
            print(f'{file} 已保存')
            delay_s(5)

        json_map = json.loads(response1.text)

        for data in json_map['data']:
            seriesId = data['id']
            timestamp = get_milliseconds_timestamp()
            nonceStr = generate_random_string()
            plaintext = f'channel=vivo&nonceStr={nonceStr}&osType=android&productId=746abe7c-796c-4ebb-a6c9-b287bf503da4&sdkIntVersion=4056&seriesId={seriesId}×tamp={timestamp}&token={token}&version=8&vestId=6dbfae2e-776e-47fd-872c-695dffc1935d&bizCodeAbc=79fa3d5f857cf66a&'

            cipherTxt = rsa_encrypt_long_data(plaintext)
            sign = calculate_md5_string(cipherTxt + "BC56EAAB76C5492E")
            response2 = requestCar(seriesId, cipherTxt, nonceStr, sign, timestamp)
            file = os.path.join('汽车比价大全', '车辆信息', f'{seriesId}.json')
            os.makedirs(os.path.dirname(file), exist_ok=True)
            with open(file, 'w', encoding='utf-8') as save_file:
                save_file.write(response2.text)
                print(f'{file} 已保存')
            delay_s(5)

            car_id_list = list()
            if response2.status_code == 200:
                detail_json = json.loads(response2.text)
                for detail in detail_json['data']:
                    car_id_list.append(detail['id'])
                car_id_map = {'carIds': car_id_list}

                response3 = requestDetailCar(car_id_map, cipherTxt, nonceStr, sign, timestamp)
                file = os.path.join('汽车比价大全', '详细参数', f'{seriesId}.json')
                os.makedirs(os.path.dirname(file), exist_ok=True)

                with open(file, 'w', encoding='utf-8') as save_file:
                    save_file.write(response3.text)
                    print(f'{file} 已保存')

            delay_s(5)
```
