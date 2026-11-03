
```
WebAssembly.Instance(module, importObject);  同步  module=new WebAssembly.Module(bytes)
WebAssembly.instantiate(bytes, importObject)  异步
WebAssembly.instantiateStreaming(bytes, importObject) 异步
```

所以我们可以搜索关键词`WebAssembly`，来找初始化的地方，调用wasm文件的关键。

这个网站有混淆，可能你不能直接找到`instantiate`啊之类的关键词，但是`WebAssembly`是可以完全搜索到的，

![](https://attach.52pojie.cn/forum/202403/14/220925dvidc99c5qc95qws.png)
12.png (83.64 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4MjA2MXxiNjc2MDAwZHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2MXxiNjc2MDAwZHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

其实看到`fetch`大概率能猜到是第二个了，经验吧。

![](https://attach.52pojie.cn/forum/202403/14/220933hwq447qawwkqal28.png)
13.png (39.03 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4MjA2MnxlMWFkMDI4Y3wxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2MnxlMWFkMDI4Y3wxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

这里采用的是`instantiateStreaming`异步方式初始化。

![](https://attach.52pojie.cn/forum/202403/14/220935fgeis24dvgegv5t8.png)
14.png (51.95 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4MjA2M3w1Y2VhMDM4NXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2M3w1Y2VhMDM4NXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

居然还把它以`.jpg`后缀伪装成图片，其实你直接访问链接下载的，真实后缀就是`.wasm`。

所以到这里传入的关键点一：`bytes`我们就拿到了。

接下来就是第二个参数`importObject`

![](https://attach.52pojie.cn/forum/202403/14/220938fy2u85gn58jvggfy.png)
15.png (55.95 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4MjA2NHxkZDg4ZmIwYXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2NHxkZDg4ZmIwYXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

展开随便进一个函数，发现里面是go编译的，没做什么处理，可以发现其实用这个算法的人也不太熟悉，不敢随便改动，直接拿生成的就来用了。

直接复制下来，然后我们找到要的`importObject`是`new .Go`下面的

![](https://attach.52pojie.cn/forum/202403/14/220940nkarz0yrc820jcyl.png)
16.png (57.28 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4MjA2NXw2ZDU3NjE3OXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2NXw2ZDU3NjE3OXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

直接运行不行啊，因为最底下是命令行调用形式，所以把底下这一块删掉，就可以直接运行这个自执行了。

![](https://attach.52pojie.cn/forum/202403/14/220942pdy090x9xdpsd083.png)
17.png (46.14 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4MjA2NnxlMjU3YTBmNXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2NnxlMjU3YTBmNXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

然后`new`就有这个对象啦！！！

![](https://attach.52pojie.cn/forum/202403/14/220944jhzhq2030fqufnrh.png)
18.png (95.59 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4MjA2N3xhMjMxNjY3ZnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2N3xhMjMxNjY3ZnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

接下来就是初始化了，文件我们已经下载下来了。我这里采用同步的形式，返回就是`instance`了，同步后期好调用一点。

![](https://attach.52pojie.cn/forum/202403/14/220946gg2cn82wkg99qsgf.png)
19.png (53.12 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4MjA2OHw3NzQxOGRiNnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2OHw3NzQxOGRiNnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

然后网页调用了`run()`方法就在全局加载了`encrypt()`，`decrypt()`函数。

![](https://attach.52pojie.cn/forum/202403/14/220949ceogol6hjyll7zt4.png)
20.png (46.52 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4MjA2OXxmZjg5OTYzY3wxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA2OXxmZjg5OTYzY3wxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:09 上传

我们也来模拟一下吧

![](https://attach.52pojie.cn/forum/202403/14/221055z8ndn87ddsyd7g3n.png)
21.png (148.01 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4MjA3MHxiMGQ2YTkyYnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3MHxiMGQ2YTkyYnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:10 上传

那么，它环境是怎么检测的呢，通过什么呢？

5> 其实就是通过第二个参数`importObject`中的js方法拿浏览器环境中的值。

我们可以代{过}{滤}理全局`global`，看看它是不是取了什么东西，也就是对应网站的`window`。

以下是简单的挂代{过}{滤}理函数：

```
function myProxy(obj,name){

    return new Proxy(obj,{

    get(target, propKey, receiver)

    {

        let temp =Reflect.get(target,propKey,receiver);

        if  (typeof temp == 'object'){

            temp=myProxy(temp,name+'=>'+propKey.toString())

        }

        console.log(`${name}->get ${propKey.toString()}  return ->${temp}`)

        return temp;

    },

    set(target, propKey, value, receiver)

    {

        let temp = Reflect.set(target,propKey,value);

        console.log(`${name}->set ${propKey.toString()}  value ->${value}`)

        return temp;

    }

})

}
```

![](https://attach.52pojie.cn/forum/202403/14/221058tzmzx27sji7ydxjj.png)
22.png (109.67 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4MjA3MXwwN2JmMmEwZXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3MXwwN2JmMmEwZXwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:10 上传

可以发现它全局取不到`location`就导致了报错，所以我们补一个`location`

![](https://attach.52pojie.cn/forum/202403/14/221101gd40dh4hd6d5b47g.png)
23.png (122.18 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4MjA3MnwwODA0Yzg1MHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3MnwwODA0Yzg1MHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:11 上传

又缺了东西，按照报错来补。补`document`，`window`，然后又报了一个核心函数错。

![](https://attach.52pojie.cn/forum/202403/14/221103ftkk9118qtdc6w94.png)
24.png (123.11 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4MjA3M3w0ZTA2ZTQyNHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3M3w0ZTA2ZTQyNHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:11 上传

一搜索关键词（`toStatus()`）就能搜索到：

![](https://attach.52pojie.cn/forum/202403/14/221105zsiufee11evfzheg.png)
25.png (24.69 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY4MjA3NHw2YzUyMDk4ZHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3NHw2YzUyMDk4ZHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:11 上传

原来返回了一个url，一个指纹，经过测试，只要`fingerprint`任意值都可以。

所以我们就直接

![](https://attach.52pojie.cn/forum/202403/14/221107xu5c4i7l2egx0hpp.png)
26.png (26.83 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=MjY4MjA3NXxjYWRmYTBlMHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3NXxjYWRmYTBlMHwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:11 上传

成功补好，运行就有加密函数了

![](https://attach.52pojie.cn/forum/202403/14/221110czo2wo2lckit21dd.png)
27.png (62.91 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY4MjA3NnxiNDUzZDM1NnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MjA3NnxiNDUzZDM1NnwxNzkzNTcyNTcyfDE2ODA4NnwxOTAwODQ1&nothumb=yes)
2024-3-14 22:11 上传

运行加密函数，成功出值，多试几次，可以发现只有后面一部分会变，我们用第二种方法解就会知道是什么原因了。至此，第一种文件调用基的方案就是这样了，基本上文件调用的都可以这样通杀吧。

![](https://attach.52pojie.cn/forum/202403/14/221112ymb3smmissjm53s7.png)
28.png (87.48 KB, 下载次数: 0)
