
# 轨迹加密分析

## 代码示例

```
def get_track(self, distance):
    def __ease_out_expo(sep):
        if sep == 1:
            return1
        else:
            return1 - math.pow(2, -10 * sep)

    def random_randint(min, max):
        range_val = max - min
        rand = random.random()
        num = min + round(rand * range_val)
        return int(num)

    slide_track = [[755, 325, 3526]]
    count = 30 + distance // 2
    t = random_randint(50, 100)
    _x = 0

    for i in range(1, count + 1):
        sep = i / count
        x = round(__ease_out_expo(sep) * distance)
        t += random_randint(30, 50)

        if x == _x:
            continue

        slide_track.append([755 + x, random_randint(320, 330), 3526 + t])
        _x = x

    return slide_track
```

## 加密点概述

轨迹加密共有三处
其中1和2不是强校验可以不处理不追加进`ua`
1、点击事件轨迹，将点击验证出现滑块图前的点击轨迹加密进去
2、移动事件轨迹，将点击验证出现滑块图前的移动轨迹加密进去
3、移动事件轨迹，将滑动滑块时的轨迹加密进去
这里只分析移动滑块轨迹的加密逻辑

## `recordSA` 方法分析

进入`recordSA`方法
分析可知这里获取了`pageX`、`pageY`、轨迹点时间，加密后存入`_sa`数组
![](https://attach.52pojie.cn/forum/202508/13/162647jvhebxpj5e0xgbxa.png)
图片26.png (16.96 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5NjU2OXxhZjA1MTE3NXwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU2OXxhZjA1MTE3NXwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:26 上传
需要注意，滑动距离是有偏移计算的
![](https://attach.52pojie.cn/forum/202508/13/162650xrwy88ifb68iddi8.png)
图片27.png (98.48 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5NjU3MHxkMjUyOGNmY3wxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU3MHxkMjUyOGNmY3wxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:26 上传

## `sendSA` 方法分析

`recordSA`方法之后需要进入`sendSA`方法处理`_sa`数组
![](https://attach.52pojie.cn/forum/202508/13/162652yldz5hdhjl2dz1l9.png)
图片28.png (40.26 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=Mjc5NjU3MXxiMWNkMDBhMnwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU3MXxiMWNkMDBhMnwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:26 上传
`sendSA`方法就是遍历`_sa`数组调用`app`更新`ua`
![](https://attach.52pojie.cn/forum/202508/13/162654ehmwgwm8pg35lis9.png)
图片29.png (9.58 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5NjU3MnxlNmZmMTY3MHwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU3MnxlNmZmMTY3MHwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:26 上传

## `sendTemp` 方法分析

`sendSA`完事后进入最后的`sendTemp`方法
![](https://attach.52pojie.cn/forum/202508/13/162657y94nvvvdqew3v38i.png)
图片30.png (41.31 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5NjU3M3xiMTk4YTIxYnwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU3M3xiMTk4YTIxYnwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:26 上传
传入一个`xpath`、`x`距离、`y`是获取图片地址的`a`接口响应中的参数
获取`html`标签中的一些信息再拼接上传入的参数进行加密更新`ua`
![](https://attach.52pojie.cn/forum/202508/13/162659ebo6u4ppel31ncp3.png)
图片31.png (28.74 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5NjU3NHwzOTk2OTViNXwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU3NHwzOTk2OTViNXwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:26 上传
现在的`ua`就是我们需要的最终结果了
以上就是全部`ac`的逻辑了

## 结果验证

![](https://attach.52pojie.cn/forum/202508/13/162702iym5yy85hzzwshnr.png)
图片32.png (73.5 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=Mjc5NjU3NXxkYTg5OTIyZHwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5NjU3NXxkYTg5OTIyZHwxNzkzNTcwNzM4fDE2ODA4NnwyMDUzMDY4&nothumb=yes)
2025-8-13 16:27 上传
算法不难但建议找个第三方固定`js`的网站去练习
官网每天早上10点和晚上7点更新`js`（实际会延迟1个小时）
建议先用补环境方法理解全部过程后再抠算法
