
# [Android 分享] 关于XPlayer 万能视频播放器 破解方法改进

- 原帖: <https://www.52pojie.cn/forum.php?mod=viewthread&tid=2011803>
- 抓取时间: 2026-11-01T23:57:42

> 安卓逆向之破解XPlayer-万能视频播放器
>
> [//www.52pojie.cn/thread-1951890-1-1.html](https://www.52pojie.cn/thread-1951890-1-1.html)
>
> (出处: 吾爱破解论坛)

我按照他的方法实操以后，发现不是安装就是会员，需要手动点击购买才行，所以我照着他的思路找了上下代码的跳转关系，看看能不能直装就是会员。

软件有签名校验，都是比较常规的操作，略。如果不去掉下载模块的设置会打不开。

软件我用的版本是2.4.4.2，谷歌市场下载。首先照着他的方法过一遍，默认不是会员，需要手动点击恢复购买。

然后搜索字符串`"SB08XyFV"`，结果不多，就2个。看到形如下方的代码，字符串上面有个`const/4 v1, 0x0`，改成`1`直装就是会员了。

```
.method private synthetic z(ZLjava/util/List;)V
    .registers 5

    iget-object v0, p0, Lvw0;->a:Lt52;

    if-nez v0, :cond_5

    return-void

    :cond_5
    iget-object v0, p0, Lvw0;->c:Lvw0$b;

    const/4 v1, 0x1

    invoke-static {v0, v1}, Lvw0$b;->d(Lvw0$b;Z)Z

    invoke-virtual {p0}, Lvw0;->F()V

    if-eqz p1, :cond_45

    invoke-interface {p2}, Ljava/util/List;->iterator()Ljava/util/Iterator;

    move-result-object p1

    :cond_14
    invoke-interface {p1}, Ljava/util/Iterator;->hasNext()Z

    move-result p2

    const/4 v0, 0x0

    if-eqz p2, :cond_2c

    invoke-interface {p1}, Ljava/util/Iterator;->next()Ljava/lang/Object;

    move-result-object p2

    check-cast p2, Lu52;

    invoke-virtual {p2}, Lu52;->a()Ljava/util/List;

    move-result-object p2

    invoke-direct {p0, p2}, Lvw0;->y(Ljava/util/List;)Z

    move-result p2

    if-eqz p2, :cond_14

    goto :goto_2d

    :cond_2c
    const/4 v1, 0x0

    :goto_2d
    if-nez v1, :cond_38

    const-string p1, "SB08XyFV"

    invoke-static {p1, v0}, Lk32;->b(Ljava/lang/String;Z)Z

    move-result p1

    if-eqz p1, :cond_38

    return-void

    :cond_38
    const-string p1, "adRemoved"

    invoke-static {p1, v1}, Lk32;->g(Ljava/lang/String;Z)V

    iget-object p1, p0, Lvw0;->c:Lvw0$b;

    invoke-static {p1, v1}, Lvw0$b;->b(Lvw0$b;Z)Z

    invoke-static {v1}, Lva0;->o(Z)V

    :cond_45
    invoke-direct {p0}, Lvw0;->D()V

    return-void
.end method
```

该软件可投屏，个人感觉还不错。
