
# [Web逆向] 【JS逆向】某某经营网jsl逆向分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1983279
- 抓取时间：2026-11-01T22:20:53

## 逆向目标

- 网址：aHR0cDovL3d3dy5jYi5jb20uY24v
- 目标：cookie 中的 `__jsl_clearance` 参数

## 抓包分析

清空 cookie，刷新网站，会发起三个相同的请求，其中前两次响应状态码为 512，第三次响应状态码为 200，这是 jsl 的典型特征。

![1.png](https://attach.52pojie.cn/forum/202411/18/231551vq5jb5q0121shvjh.png)

1.png (17.23 KB, 下载次数: 1) · 2024-11-18 23:15 上传

第一次响应设置了一个名为 `__jsluid_h` 的 cookie。

![2.png](https://attach.52pojie.cn/forum/202411/18/231554mj6z555pz6pl6600.png)

2.png (16.82 KB, 下载次数: 0) · 2024-11-18 23:15 上传

第二次请求携带上一个响应的 cookie 和一个名为 `__jsl_clearance` 的 cookie。

![3.png](https://attach.52pojie.cn/forum/202411/18/231557w8b5w38b57bw317w.png)

3.png (26.28 KB, 下载次数: 0) · 2024-11-18 23:15 上传

第三次请求携带和第二次响应名字相同的 cookie，但 `__jsl_clearance` 的值不一样了，比较明显的区别就是第二次 cookie 值中间的数字是 -1，第三次 cookie 值中间的数字是 0。

![4.png](https://attach.52pojie.cn/forum/202411/18/231600iw0w9g09z959d11g.png)

4.png (25.72 KB, 下载次数: 0) · 2024-11-18 23:16 上传

## 逆向分析

流程分析完了，下面开始正式逆向，一般来说，对于 cookie 加密的情况，直接 hook。

成功 hook 到第二次的 cookie，而且代码也非常熟悉，经典的 OB 混淆。

![5.png](https://attach.52pojie.cn/forum/202411/18/231604f8559e1952vyex35.png)

5.png (18.08 KB, 下载次数: 0) · 2024-11-18 23:16 上传

那第二次请求的 `__jsl_clearance` 在哪生成的呢，为什么没有 hook 到。

还是下熟悉的 script 断点看看吧。

第一加载 index 的时候，会发现是这个时候生成了第二次请求的 `__jsl_clearance`，这个并不难，我们重点需要分析第二次生成的 `__jsl_clearance`。

![6.png](https://attach.52pojie.cn/forum/202411/18/231607rz63wp3177nagwz1.png)

6.png (17.22 KB, 下载次数: 0) · 2024-11-18 23:16 上传

既然遇到 OB 混淆了，那就无脑 AST。

还原后的代码只有 240 行，我们甚至可以直接静态分析。

![7.png](https://attach.52pojie.cn/forum/202411/18/231610lyzxxc9ra2hq3aog.png)

7.png (18.11 KB, 下载次数: 0) · 2024-11-18 23:16 上传

直接搜索 `document["cookie"]`，只有一个地方，`_0x4954f5` 就是我们的目标了。

![8.png](https://attach.52pojie.cn/forum/202411/18/231613t448rsmp4pxrq8zp.png)

8.png (15.49 KB, 下载次数: 0) · 2024-11-18 23:16 上传

继续往前分析，发现 `_0x5bef39[0]` 就是 `__jsl_clearance` 最终的值。

![9.png](https://attach.52pojie.cn/forum/202411/18/231616jugpryirzdnrut11.png)

9.png (11.12 KB, 下载次数: 0) · 2024-11-18 23:16 上传

再往前分析，最终找到目标参数的生成位置。

![10.png](https://attach.52pojie.cn/forum/202411/18/231620ctjyppt4ardjsstw.png)

10.png (18.2 KB, 下载次数: 0) · 2024-11-18 23:16 上传

剩下的事情就是，你懂的，缺啥补啥。其中 `_0x23b046` 是调用 `go()` 函数传进去的对象。

![11.png](https://attach.52pojie.cn/forum/202411/18/231623nu12ap2pp2bzj3nq.png)

11.png (11.74 KB, 下载次数: 0) · 2024-11-18 23:16 上传

下面直接模拟请求：

![12.png](https://attach.52pojie.cn/forum/202411/18/231626jahsin6irnj94f4i.png)

12.png (70.66 KB, 下载次数: 0) · 2024-11-18 23:16 上传

成功！！！

> 请求的时候可以 session 保持，其次还需要注意 JS 脚本是动态的，每次使用的 hash 算法可能都不一样，需要将动态的参数提取出来进行请求。
