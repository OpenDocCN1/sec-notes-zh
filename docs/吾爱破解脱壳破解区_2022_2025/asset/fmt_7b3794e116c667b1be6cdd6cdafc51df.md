
## 激活返回值与 XML 处理

`8888`

其中只有一个有用的值，就是 `8888`，表示返回的是整数，值为 `8888`，而这个 `8888` 表示的是剩余可激活的次数（同一台机器多次激活也只算 1 次，也可以説是剩余可激活的设备数）。

`F8` 往下走，来到这里，对收到的 XML 内容进行处理：

![small_116.jpg](https://attach.52pojie.cn/forum/202208/15/081845ptr087s4rrlrslnr.jpg)

`small_116.jpg`（224.92 KB，下载次数：246）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyMXxiMjliZjYzMHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:18 上传）

按 `F7` 进入该函数。

还是老办法，生成一个 `XMLDocument` 对象来处理 XML 文档，如下图所示：

![small_117.jpg](https://attach.52pojie.cn/forum/202208/15/081849l8du4guitugiud3d.jpg)

`small_117.jpg`（228.91 KB，下载次数：235）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyMnw1ODI3ZTc0NHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:18 上传）

继续往下走一段代码，来到如下位置，就是解析并取得那个最终的响应值了：

![small_118.jpg](https://attach.52pojie.cn/forum/202208/15/081851x6g8iglof8969v0d.jpg)

`small_118.jpg`（223.55 KB，下载次数：241）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyM3xiZTVmOTg1YXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:18 上传）

这个函数返回的是一个 `Variant` 类型的值，其实就是 `8888`。

`Variant` 也可看作一个对象，结构如下：

```c++
variant_struct {
     pointer methods[];      /// methods = NULL
     pointer properties[];   /// 
     pointer events[];       /// events = NULL
     qword   ref_count;     /// reference count
     qword   unknown1;
     qword   unknown2;
     qword   value_or_address;   //// 当时基本类型值时，保存的是实际值;当是字符串或对象时，保存的是地址指针
}
```

取得这个值后，接下传递给下一个事件，是以 `Delegate` 调用的方式来处理的，如下图所示：

![small_119.jpg](https://attach.52pojie.cn/forum/202208/15/081855xrwzyzwdhokrdnje.jpg)

`small_119.jpg`（222.73 KB，下载次数：237）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyNHw2N2ZiZWRkYnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:18 上传）

## 激活结果处理

这样，我们进入另一个对象（`ServerComm`）的事件处理，这是前面讲过的 `StudioLicenseKeyPasswordDialog._wrapper_ServerComm` 的一个实例。

![small_120.jpg](https://attach.52pojie.cn/forum/202208/15/081858mxzf88pp69w9wxwm.jpg)

`small_120.jpg`（210.76 KB，下载次数：236）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyNXxkMTJiYTY3NXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:18 上传）

这个事件处理函数的参数就是前面的 `Variant` 类型的变量。

![small_121.jpg](https://attach.52pojie.cn/forum/202208/15/081901v0sesf9zsf149bg3.jpg)

`small_121.jpg`（219.7 KB，下载次数：237）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyNnw0NTYxYWU5MXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

在处理这个数据前，先恢复成整数值，如上图所示。

然后就是对这个数据进行对比，根据返回值进行对应的处理和提示：

![small_122.jpg](https://attach.52pojie.cn/forum/202208/15/081903bvvdud07nckddzcc.jpg)

`small_122.jpg`（221.01 KB，下载次数：240）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyN3w0Nzc1Y2E3OHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

如果 `int >= 1`，表示激活正确，还有剩余激活次数。

如果 `int = -1`，表示激活异常，激活失败了。

如果 `int = -2`，表示激活次过多，不能再激活了。

上面会根据不同的返回值，取得不同的提示信息字符串。

接着往下走，如果激活成功，就会保存当前授权文件的内容到数据库了，下面就是调用保数据的位置：

![small_123.jpg](https://attach.52pojie.cn/forum/202208/15/081906ftv9tfgf83ox7j5o.jpg)

`small_123.jpg`（217.44 KB，下载次数：236）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyOHw4Y2FmNzhkM3wxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

## 授权数据保存与激活提示

这个函数内有保存到数据库的代码，相当长，生成 `SQL` 的位置如下：

![small_124.jpg](https://attach.52pojie.cn/forum/202208/15/081909gpkoepid5ffpefb5.jpg)

`small_124.jpg`（212.58 KB，下载次数：255）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAyOXwxZDEyZDIxZHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

接下来就提示激活成功了：

![small_125.jpg](https://attach.52pojie.cn/forum/202208/15/081911w2k6b0x8e8j6n8yn.jpg)

`small_125.jpg`（117.52 KB，下载次数：236）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzMHxiOGY5NjhiNXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

上面还剩余多次的情况，如果只剩一次了，提示如下（其实就是处理了一下英文名词单数和复数的显示问题）

![small_125_1.jpg](https://attach.52pojie.cn/forum/202208/15/081913yk5xq055k5nk0k1x.jpg)

`small_125_1.jpg`（104.05 KB，下载次数：233）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzMXw3MjFjMjEyMHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

如果失败，就不会保存授权数据到数据库，同时会提示出错：

![small_126.jpg](https://attach.52pojie.cn/forum/202208/15/081918u6x6xmee36bht1hj.jpg)

`small_126.jpg`（129.11 KB，下载次数：237）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzM3xmZmFiNTA0MnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

这个就是前面判断 `-1` 的情况。

![small_127.jpg](https://attach.52pojie.cn/forum/202208/15/081920l4zypxu0mxeaqyxa.jpg)

`small_127.jpg`（154.12 KB，下载次数：239）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzNHwyOTVmZjk2YnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

这里是前面判断 `-2` 的情况，表示激活次数（设备）过多，超过次数，只能迁移以前的授权过来了。

还説明一下，前面那个判断是有 BUG 的，当值为 `0` 或 `-3` 及以及下的负数时，也是激活成功的：

![small_127_1.jpg](https://attach.52pojie.cn/forum/202208/15/081922i89iizi1ujjvl8fo.jpg)

`small_127_1.jpg`（103.46 KB，下载次数：238）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzNXw4NGJlM2RmM3wxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

![small_127_2.jpg](https://attach.52pojie.cn/forum/202208/15/081924d7o7vn827o6oo3mn.jpg)

`small_127_2.jpg`（103.15 KB，下载次数：239）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzNnw2Y2JlM2I3YnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

## serial_index 校验

前面我们説过 `serial_index` 的问题，就是新加的授权文件中的 `serial_index` 一定要大于数据库中的授权记录中的 `serial_index`，就是：

`xml_serial_index > db_serial_index` 要满足，否则就会报以下错误：

![small_128.jpg](https://attach.52pojie.cn/forum/202208/15/081927ps90l95cg5vovonh.jpg)

`small_128.jpg`（135.47 KB，下载次数：234）；[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzN3xlMDk4ZjkyNHwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes)（2022-8-15 08:19 上传）

只要将同一个授权文件添加两次，第 2 次就会报这个错误了。

这样，整个授权检查和激活过程就结束了。

## 查看对象类结构

下面，我説一下如何快速查看一个对象的类名吧，一个对象类的基本结构如下：

```c++
object_struct {
    pointer events[];
    pointer properties[];
    pointer metheds[];
    qword ref_count;
    qword inherited_count;  //// 不确定
    dword unknown1;
    pointer ref_object;   //// 最后引用的对象，没有则是指向对象本身
    dword unknown3;
    dword unknown4;
    dword unknown5;
}
```

下面是上面结束 `properties[]` 的前面几个属性：

```asm
```
