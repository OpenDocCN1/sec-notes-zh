
# [iOS 原创] 记一次某相机逆向

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2039817
- 抓取时间: 2026-11-01T15:43:41.813Z

哈喽，大家好

很久没有发帖了，今天水一贴吧，其实这个应用很多人[破解](https://www.52pojie.cn/)了，所以这次就小计一下破解逻辑思路跟教程，大佬勿喷。

- 调试应用：Dazz相机
- 调试工具：hopper + AppSearch

进入主题：

首先我们下载Dazz相机并且砸壳

得到ipa文件后提取mach-o文件`DazzCamera`

接下来直接拖到hopper进行加载

加载的时间段打开我们AppSearch(自写的)，搜索dazz相机并且查看，如图：

然后点击查看，就会看到我们需要的东西，如图：

![](https://attach.52pojie.cn/forum/202506/18/184546h2tu7yigypwziuql.jpg)

我们看到我们需要的`dazz_products_pro`

![](https://attach.52pojie.cn/forum/202506/18/184700leccl8s1sooiuozl.jpg)

接下来打开我们的hopper搜索，如图：

![](https://attach.52pojie.cn/forum/202506/18/184847onv4iwvjevvvl0lj.jpg)

双击跳转，然后看伪代码：

![](https://attach.52pojie.cn/forum/202506/18/185026uxd1mxd2s4gy1rdm.jpg)

解释含义：

```
// 调用一个内部函数 sub_100323f34，检查 "dazz_products_pro" 是否已经购买。
// 这个函数很可能返回 1 (代表已购买) 或 0 (代表未购买)。
if ( (sub_100323f34(..., "dazz_products_pro", ...) 返回的结果) == 0 ) {
    
    // 如果结果等于 0 (即【没有】购买 Pro 版)
    // 就跳转到 loc_10005002c，那里是为【普通用户】准备的代码。
    goto loc_10005002c;
}

// 如果上面的 if 条件不成立 (即【已经】购买了 Pro 版)
// 程序就不会跳转，而是继续往下执行，这里是为【VIP用户】准备的代码。
```

然后我们看汇编代码，进行修改：

![](https://attach.52pojie.cn/forum/202506/18/185930ojjuxdvosjnxvjnu.jpg)

结束，大功告成

喜欢记得给个Start留言，谢谢
