
# [Web逆向] 某鱼sign参数逆向[适合小白新手]

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2057979
- 抓取时间: 2026-11-01T22:03:32
- 有大神的帖子其实已经逆向出来,只不过没有跟栈的过程,对我这样的小白来说,跟栈分析的过程很重要,于是有了下面的跟栈分析
- 大神帖子地址: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2021569&highlight=%CF%D0%D3%E3
- 目标网址:aHR0cHM6Ly93d3cuZ29vZmlzaC5jb20v
- 先抓个包看看数据,`sign`参数就是此次的逆向目标

![](https://attach.52pojie.cn/forum/202509/04/171941c7w2fnmx600gm0o2.png)

1.png (548.29 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwMDc4NHwxNjM3ZTk4NXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc4NHwxNjM3ZTk4NXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

## 分析过程

- 通过搜索`sign`的方式,点进去看看

![](https://attach.52pojie.cn/forum/202509/04/171944a0rh4pb1qqii1lia.png)

2.png (86.17 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDc4NXw2NTdhOTA3YXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc4NXw2NTdhOTA3YXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

- 可以看出,`sign`值为`k`,而`k`由`i(一堆参数)`函数生成

![](https://attach.52pojie.cn/forum/202509/04/171946rommsmauip9uu9p8.png)

3.png (50.75 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDc4NnxmODRiZDgyM3wxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc4NnxmODRiZDgyM3wxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

- 附近打个断点进去看看`i(参数)`函数如何传参

![](https://attach.52pojie.cn/forum/202509/04/171948kwbyubzpuramwvvi.png)

4.png (187.98 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwMDc4N3w0NDQwYjEyN3wxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc4N3w0NDQwYjEyN3wxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

- 可以看出,`i(d.token,j,h,c.data)`,参数有4个
  - `d.token`需要继续追踪
  - `j`显示是一个时间戳,生成的逻辑就在附近
  - `h`对应`appKey`,可以从请求中找到,是个定值
  - `c.data`,是一堆字符串,也是定值
- 继续追踪`d.token`,尝试搜索`d.token`,发现由`a.token`值生成

![](https://attach.52pojie.cn/forum/202509/04/171950y3110k3hiyu5r0ry.png)

5.png (73.51 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjgwMDc4OHxlNGM4ZjliZnwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc4OHxlNGM4ZjliZnwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

- 继续尝试搜索`a.token`,发现和`l(B)`函数有关,附近打个断点,刷新页面跟进`l(B)`函数

![](https://attach.52pojie.cn/forum/202509/04/171952yy1gj1ajp5llllgy.png)

6.png (75.26 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDc4OXxkNDg3ZmExOXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc4OXxkNDg3ZmExOXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

![](https://attach.52pojie.cn/forum/202509/04/171955cpibki2cgcbgcpzh.png)

7.png (29.03 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDc5MHw2MzY5YWRkYnwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc5MHw2MzY5YWRkYnwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

- `l`函数的逻辑如下,丢给AI分析

```
function l(a) {
    var b = new RegExp("(?:^|;\\s*)" + a + "\\=([^;]+)(?:;\\s*|$)").exec(document.cookie);
    return b ? b[1] : void 0;
}
```

![](https://attach.52pojie.cn/forum/202509/04/171957zq3k3j4h33tqtq85.png)

8.png (74.4 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjgwMDc5MXwyMDlhNTJkMXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc5MXwyMDlhNTJkMXwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

- 所以结论就是从Cookie中获取的,查看一下Cookie,核对一下,果然如此

![](https://attach.52pojie.cn/forum/202509/04/171959pllokr8cyg7g8nia.png)

9.png (100.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjgwMDc5Mnw3OGVjMWUyNHwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDc5Mnw3OGVjMWUyNHwxNzkzNTcwNjA5fDE2ODA4NnwyMDU3OTc5&nothumb=yes)

2025-9-4 17:19 上传

## 小结

4> 小结:如果是经验丰富的大神,看到类似`token`的参数,可能直接就从Cookie入手了,简单粗暴解决问题.而对于我这样的新手来说,更加注重跟栈分析的学习过程,以此帖记录一下!
