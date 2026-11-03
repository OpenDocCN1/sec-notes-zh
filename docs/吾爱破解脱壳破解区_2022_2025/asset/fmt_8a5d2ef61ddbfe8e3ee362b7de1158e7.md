
15.png (50.44 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwODQ0N3w0OGRkYjhiMXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ0N3w0OGRkYjhiMXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

继续还原

```
function (e, t, n) {
    var u = c['createCipheriv']("aes-128-cbc", t, n)
    , l = u["update"](e, "utf8", "base64");
    return l += u["final"]("base64"),
    l["replace"](/\//g, "_")
}
```

很明显了

标准的`aes-128-cbc`加密

接着我们要找到 `key` 和 `iv`

分析可知 `key` 是固定的 `iv` 每次都不一样

重点找下`iv`的生成逻辑

回到 `f` 位置

鼠标放在 `S` 上可以看到 `iv` 早已生成了

![](https://attach.52pojie.cn/forum/202510/16/134820y8tq6ui7vue68e17.png)

16.png (18.48 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ0OHxiZWYyYzIxNnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ0OHxiZWYyYzIxNnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

往上在刚进入`sessionData`处打上断点

![](https://attach.52pojie.cn/forum/202510/16/134823amvkmk6n6koztmhe.png)

17.png (35.06 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ0OXwyNjhjYmEyZXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ0OXwyNjhjYmEyZXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

`getKey`方法执行完后 `S` 中就有`key`值了

![](https://attach.52pojie.cn/forum/202510/16/134825rxmwf1vv7xxx78jx.png)

18.png (12.01 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1MHw4NDJkYjUxOXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1MHw4NDJkYjUxOXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

继续往下走

![](https://attach.52pojie.cn/forum/202510/16/134827nkkk6adakka6z0o6.png)

19.png (14.85 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwODQ1MXwyMDY0YjhlN3wxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1MXwyMDY0YjhlN3wxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

执行完`genrandomString`方法 `S` 就有 `iv`了

![](https://attach.52pojie.cn/forum/202510/16/134830wi5ibpifb1po59p9.png)

20.png (15.68 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1MnxhMTA3MmZlZnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1MnxhMTA3MmZlZnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

进方法看看

![](https://attach.52pojie.cn/forum/202510/16/134833stjdumz31oxesbnd.png)

21.png (43.05 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwODQ1M3xkMTNkYzljMXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1M3xkMTNkYzljMXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

还原

```
functio() {

    var i = Math["random"]()["toString"](36)["slice"](-8) + "-" + Math["random"]()["toString"](36)["slice"](-8) + (new Date)["getTime"]();

    var s = this["_dynamicEncrypt"](i, C, w)["replace"]("_", "");

    return this.iv = s["substring"](1, 17),

    this.iv

}
```

创建一个随机字符串还是用上面的aes加密

这次用的`key` 和 `iv` 是固定的

![](https://attach.52pojie.cn/forum/202510/16/134835lq9fm4omtz21u7m2.png)

22.png (7.36 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1NHwwZjBiM2FiMXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1NHwwZjBiM2FiMXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

生成的就是我们需要的动态 `iv` 值

还原代码如下

```
def random_iv(self):
    part1 = self.random_id(8)
    part2 = self.random_id(8)
    timestamp = str(int(time.time() * 1000))
    crypto = AesCbcCrypto(b"E08247708934F42E", b"0A234C4C639E015D")
    encrypt_info = crypto.aes_encrypt(f"{part1}-{part2}{timestamp}")
    return encrypt_info[1:17].encode()
```

请求看看结果

注意请求头`headers`中需要有`Fetch-Mode`和`etag`（动态 `iv` 值）

![](https://attach.52pojie.cn/forum/202510/16/134837ys3i8yqcuiycr99o.png)

23.png (40.82 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1NXw2ZDk5ZDljMnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1NXw2ZDk5ZDljMnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

3> `websocket`分析

`ws`链接

```
ws://www.****.cn/1ywuKELSO2ahQuWZ/pr/8XIdWc356kGBPGt7xGkFZKdv19oQMigyzm9H3A3Wupo%3D/b/ws/af10no7t8e/095f5c9f-0945-414d-aace-d4f60167807f

8XIdWc356kGBPGt7xGkFZKdv19oQMigyzm9H3A3Wupo%3D  sessions接口返回的token值

af10no7t8e  tab_id

095f5c9f-0945-414d-aace-d4f60167807f  uuid
```

跟着`ws`请求的堆栈进去

![](https://attach.52pojie.cn/forum/202510/16/134840cfqk5pnjj3pcfq3q.png)

24.png (22.35 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1NnwzY2FiZTgwZHwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1NnwzY2FiZTgwZHwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

![](https://attach.52pojie.cn/forum/202510/16/134842d262pjw9w8hp9pr9.png)

25.png (30.67 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1N3xjZTU3ZTYwYnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1N3xjZTU3ZTYwYnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

在这里可以看到创建了`websocket`的实例以及四个关键事件注册回调函数

`onmessage` 是收到服务器消息时触发的回调函数

这里绑定到了`this._onMessage`方法里

![](https://attach.52pojie.cn/forum/202510/16/134844k43u0xm9mdc2n2z0.png)

26.png (82.06 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1OHxhNjc5YTc5MnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1OHxhNjc5YTc5MnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

一步步来看看

![](https://attach.52pojie.cn/forum/202510/16/134847xiznzjzejejn9m7s.png)

27.png (19.56 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ1OXw4NzJlMTBkNXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ1OXw4NzJlMTBkNXwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

用 `t = this` 保存当前实例引用（避免在回调中 `this` 指向错误）

`lastSyncTime`记录最后一次收到消息的时间（用于心跳、超时判断等）

`n` 获取原始消息数据

如果：

`window.__wm_wsNoBinary__` 为真

消息是 JSON 字符串，用 `JSON.parse(n)` 解析

否则：

消息是二进制数据（`ArrayBuffer`）先转为`Uint8Array`，再用`r.decode`解码

接着遍历解码后的消息进行处理

![](https://attach.52pojie.cn/forum/202510/16/134849dwyyhr727fyptkef.png)

28.png (19.64 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjgwODQ2MHw2OTA1NjU3ZnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwODQ2MHw2OTA1NjU3ZnwxNzkzNTcwNDk1fDE2ODA4NnwyMDY2MTkx&nothumb=yes)

2025-10-16 13:48 上传

`e[1][1]`是取当前消息的指令

`a.CommandCodes` 是指令集如下
