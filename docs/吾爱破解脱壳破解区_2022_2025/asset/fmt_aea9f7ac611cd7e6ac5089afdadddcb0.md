
# [Web逆向] 顶象滑块验证码纯算逆向分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2053068
- 抓取时间：2026-11-01T22:05:40

## 声明

本文章中所有内容仅供学习交流使用，不用于其他任何目的，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关！若有侵权，请联系作者删除。

## 逆向目标

目标网站：

`aHR0cHM6Ly93d3cuZGluZ3hpYW5nLWluYy5jb20vYnVzaW5lc3MvY2FwdGNoYQ==`

关键要点：图像撕裂还原、轨迹、参数【`lid`, `c`, `ac`】

## 抓包分析

本篇文章以官网 `demo 5.1.53` 版本进行分析，需要注意官网关键 `js` 文件每天会动态更新两次。

进入网页，按 `F12` 打开调试窗口后刷新页面，建议每次都清理一下缓存或者直接使用无痕模式。

刷新页面后定位到 `demo` 位置，点击滑动拼图查看发送请求。

![图片1](https://attach.52pojie.cn/forum/202508/13/162543ip4g7s4d1i9a7ddi.png)

### 接口 `a` 请求载荷

![图片2](https://attach.52pojie.cn/forum/202508/13/162546hwbbwbwf8wk8esf7.png)

这里只需要注意 `aid` 的生成。

```
aid = 'dx-' + str(int(time.time() * 1000)) + '-' + str(random.randint(10000000, 99999999)) + '-3'
```

### 接口 `a` 响应内容

![图片3](https://attach.52pojie.cn/forum/202508/13/162551gcwnaownq8dyrwqr.png)

- `p1`：背景图地址
- `p2`：滑块图地址
- `sid` 和 `y` 后面需要用到

当我们滑动滑块完成验证后会发送一个 `v1` 接口的请求。

![图片4](https://attach.52pojie.cn/forum/202508/13/162555vwcjuj0szu9jckaz.png)

提示：本文截图中某些值可能和之前截图中的不一致，以文字说明为准，图知其意便可。

- `ac`：包含轨迹的加密参数
- `c`：`c1` 接口获取，需要逆向
- `sid`：`a` 接口响应中获取
- `aid`：这里 `aid` 要与 `a` 接口的 `aid` 一致
- `x`：滑动距离
- `y`：`a` 接口响应中获取

`v1` 接口响应中的 `token` 值是我们本次逆向的最终目标。

![图片5](https://attach.52pojie.cn/forum/202508/13/162558lztc51yo97n5l74w.png)

## 流程梳理

1. 第一次请求 `c1` 接口获取 `lid`
2. 第二次请求 `c1` 接口获取 `data` 值
3. 请求 `a` 接口获取图片地址以及一些关键参数
4. 请求 `v1` 接口完成验证获取 `token`

## 图像还原

老样子打上 `canvas` 断点后点击验证滑块，断到如下图位置。

![图片6](https://attach.52pojie.cn/forum/202508/13/162600tu435z54yjtfrk44.png)

分析入参：

- `n`：`canvas` 对象
- `t`：`img` 对象
- `i`：图片高度
- `e`：图片高度
- `r`：32 位数组

分析代码：

1. 将一张图像完整绘制到 `canvas` 上
2. 然后根据数组 `r` 的长度将图像水平分割成多个切片
3. 再按照数组 `r` 的顺序重新排列这些切片绘制到 `canvas` 上

知道逻辑了那么就需要找到关键 32 位数组是如何生成的。

继续往上跟栈找到生成位置。

![图片7](https://attach.52pojie.cn/forum/202508/13/162603cjsa7qu6qvzu755x.png)

函数 `sn` 算法还原，传入 `a` 接口 `p1` 的值也就是背景图地址。

```
def get_arr_32(self, img_url):
    n = img_url.split("/")[-1].split(".")[0]
    c = []
    for i in range(len(n)):
        u = ord(n[i])
        while u % 32 in c:
            u += 1
        c.append(u % 32)
    return c
```

图像还原算法：

```
def draw_slices_from_array(self, image_content, output_path, arr):
    src_img = Image.open(BytesIO(image_content))
    src_width, src_height = src_img.size
    canvas_width = 32 * 12
    canvas_height = 200
    dest_img = Image.new("RGB", (canvas_width, canvas_height), "white")
    for index, x in enumerate(arr):
        r = x * 12
        if r + 12 > src_width:
            continue
        cropped = src_img.crop((r, 0, r + 12, 200))
        dest_img.paste(cropped, (index * 12, 0))
    dest_img.save(output_path)
```

还原结果：

![图片8](https://attach.52pojie.cn/forum/202508/13/162605xgwzt0c5q020rllu.png)

![图片9](https://attach.52pojie.cn/forum/202508/13/162608unutusvy9jd94cts.png)

## `C` 值分析

对 `v1` 接口的 `c` 值进行搜索后发现是在 `c1` 接口返回的。

`c1` 接口请求了两次。

第一次是 `get` 请求，`headers` 中有个短的加密参数 `Param`。返回一个 `lid`。

![图片10](https://attach.52pojie.cn/forum/202508/13/162610rk4hnzeseybbl3qb.png)

![图片11](https://attach.52pojie.cn/forum/202508/13/162612ozavtwufupzvxtqw.png)

第二次是 `post` 请求，表单中有个长的加密参数 `Param`。返回一个 `data` 值也就是我们要的 `c` 值。

![图片12](https://attach.52pojie.cn/forum/202508/13/162615oymsrrg84q5yzgj4.png)

![图片13](https://attach.52pojie.cn/forum/202508/13/162617mfjao2wloapommqj.png)

### 第一次 `c1` 请求的 `Param` 分析

打上 `c1` 的 `xhr` 断点清缓存刷新页面。

往上跟栈可定位到其生成位置在一个 `index.js` 文件中。

注意，它有两个 `index.js` 文件，咱分析的这个是带 `?_t=***` 的。

![图片14](https://attach.52pojie.cn/forum/202508/13/162620zq0ju57881se72mu.png)

参数除 `lid` 外都是固定的。

`lid` 生成代码：

```
l = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
lid = str(int(time.time() * 1000)) + ''.join(random.choice(l) for _ in range(32))
```

进入加密函数后可以看到是个控制流。

![图片15](https://attach.52pojie.cn/forum/202508/13/162622xxqpk0kjtbx6pxpf.png)
