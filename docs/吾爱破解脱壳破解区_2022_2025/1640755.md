
# [原创] 修改关键 `call`，小白也能让软件真实被破解！

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1640755
- 抓取时间：2026-11-01T23:24:44

今天要分享的主题是修改关键 `call`。在讲之前还是要说一下，这个东西的应用场景。

很多软件，我们在爆破了它的关键跳，点击注册以后也显示成功了，但是在使用它某些功能时候，仍然会提示我们，当前使用的是未注册版本，需要我们注册。这种伪注册，不知道大家有遇到过吗？

## 为什么修改关键 `call` 能实现软件的真实注册

首先说明一下，`call` 就相当于一个子程序，它们共同组成了一个软件。就像吾爱破解论坛下面有很多版块，每个版块都可以单独发帖，这些版块构成了吾爱破解论坛。

回过来说修改关键 `call`：软件在计算自己是否已经被注册时候，它调用的是同样的 `call`。也就是说，即使它有一万个功能，并且这些功能分布在十万个不同的 `call`（子程序）里面，但在它们判断是否注册时候，只会调用同一个 `call`。我们把它叫做关键 `call`，所以修改了这个地方，就等于改了全世界！

## 环境与初步运行

用到的 cm，我会放在附件里面，Borland Delphi 无壳。

![image.png](https://attach.52pojie.cn/forum/202205/25/084852zfz19nmqkuwusmsn.png)

运行起来。有个 `NAG` 窗口。

![image.png](https://attach.52pojie.cn/forum/202205/25/085125j3s6h666i7ts5jdz.png)

`F12` 暂停，`Alt+F9` 然后点击 `NAG` 窗口的确定按钮，此时就会断下来，向下跟踪，会执行到这里。

![image.png](https://attach.52pojie.cn/forum/202205/25/090104s1oiiy6fc6an414f.png)

接下来去掉就可以了。接下来载入去掉 `NAG` 窗口的程序，运行起来。

![image.png](https://attach.52pojie.cn/forum/202205/25/090323z19clsncwc3kps42.png)

随便输入注册名和密码，点击确定。

![image.png](https://attach.52pojie.cn/forum/202205/25/090432i6dilvngd30dgl53.png)

没有任何反应！

这个时候可以用 `OD` 带的插件，万能断点！

![image.png](https://attach.52pojie.cn/forum/202205/25/090953mi4o64994yjl1jo4.png)

![image.png](https://attach.52pojie.cn/forum/202205/25/091029x5cav2u5rkckaasl.png)

还有另外一种方法，就是 Borland Delphi 破解专用工具 `dededark`。

![image.png](https://attach.52pojie.cn/forum/202205/25/091253uggammg2ca2p6gww.png)

我们可以看到，按钮时间的起始位置是 `00450518`，`Ctrl+G` 输入 `00450518` 到达这个位置，`F2` 下断点，运行程序，然后重新点击一下确定按钮。

![image.png](https://attach.52pojie.cn/forum/202205/25/091656nczufwccrru8wfwy.png)

## 寻找关键 `call`

被断下来了，接下来向下跟踪，寻找关键 `call`，一般关键 `call` 就在关键跳的上面两行。

比如：

```
call 004XXXX          // 这里就是关键call
cmp xxx，xxx          // 对比真假
je xxx                // 关键跳
```

这里就是关键 `call`，错误了就跳，正确了就不跳。

注意这个地址 `00453BD4`。

![image.png](https://attach.52pojie.cn/forum/202205/25/101422pafimmkm2byzaqim.png)

这个地址里面存放的数据和 `0` 做了比较，如果是 `0` 就跳向失败，不是 `0` 就注册成功了。

我们 `F7` 进入关键 `call` 子程序。

![image.png](https://attach.52pojie.cn/forum/202205/25/093950se9fdr98ft4ef7k9.png)

比如软件有 `N` 个地方都会调用这个 `call` 来判断自己是否已经被注册。那怎么才能一劳永逸地修改呢？

只要将 `call` 的返回地址（`00453BD4`）里面存放的数据，在返回时候设定成为相同或者不同的数值即可（具体是否相同，要根据不同软件设定）。

这个 cm 我们为了实现成功，只要将 `00453BD4` 里面的数据设定成为非零数据即可。

## 修改关键 `call`

我们可以这么改。

![image.png](https://attach.52pojie.cn/forum/202205/25/101633iss3snoleofvfas8.png)

`mov byte ptr ds:[0x453BD4],0x1` 将数值 `1` 存在 `00453BD4` 这个地方，然后 `retn` 直接返回。

看效果。

![image.png](https://attach.52pojie.cn/forum/202205/25/103127fhsh1fj98ssrl8bj.png)

## 附件

[下载附件（158.82 KB，下载次数：123）](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyNDU4N3w1NDQ3M2VlZnwxNzkzNTc1NDgxfDE2ODA4NnwxNjQwNzU1)

下载积分：吾爱币 `-1 CB`
