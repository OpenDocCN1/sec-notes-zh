
```csharp
// 动态加载指定路径的DLL文件
Assembly assembly = Assembly.LoadFile(Util.GetAppPath("/app/" + app.Path) + "/" + app.Path + ".dll");

// 获取assembly中指定类名的Type对象
Type type = assembly.GetType(app.ClassName);

// 使用反射创建type类型的实例
object obj = Activator.CreateInstance(type);

// 如果obj实现了IACSetter接口，将调用其SetIAC方法
if (obj is IACSetter setter)
{
    // 创建一个AppController实例并传给SetIAC方法
    setter.SetIAC(new AppController(app));
}
```

进这个类就可以看到

![](https://attach.52pojie.cn/forum/202410/12/155745crq61tmtbibz6rbi.png)

30.png (19.5 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODM5OHw5ZGFkYTM3MnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODM5OHw5ZGFkYTM3MnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

这个检测函数了，但是这个检测函数值负责返回，不判断。分析也没有发现调用，那说明是小App的DLL调用了这个方法，小APP有点多，不可能一个个去改，这个函数留给我们的操作空间也不是很多，因为没有声明临时变量，所以修改`ApiResult`的内容还挺麻烦的，那我们换个思路，让他请求一个返回status返回200的地址。

打开抓包软件，看看软件请求的哪个数据一直都是200返回值的

![](https://attach.52pojie.cn/forum/202410/12/155747udv1z878brhh1vh8.png)

31.png (34.32 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODM5OXxlZjMyNzgwMnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODM5OXxlZjMyNzgwMnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

找到了一个符合要求的，这个是每次退出都会调用的，我们吧参数改一下看看是不是还返回200

经过测试修改参数之后返回的还是200，说明没对传入的参数检测

![](https://attach.52pojie.cn/forum/202410/12/155749rzkeglutm5e35ees.png)

32.png (61.92 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODQwMHw3NTU4NzE0ZXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwMHw3NTU4NzE0ZXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

那我们修改一下访问的路径

![](https://attach.52pojie.cn/forum/202410/12/155751wy4964zznavvlr1y.png)

33.png (15.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjcyODQwMXw5NGZhNDJlM3wxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwMXw5NGZhNDJlM3wxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

直接右键编辑C#方法，把这个改成`active.php`或者修改获取的改成获取active的。

保存，测试

![](https://attach.52pojie.cn/forum/202410/12/155753kai6bu3hhxggd3dg.png)

34.png (28.45 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODQwMnwyMjdjYjM5MXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwMnwyMjdjYjM5MXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

OK，可以打开了，但是我们还是遇到了和抓包破解一样的问题，每次都要重新下载。

### 处理状态检测

我们再去找状态检测的地方，继续返回到修改应用中心的地方，在这个类里面可以看到有添加`item`到列表的操作，那我们就看看在哪里设置了按钮的状态，往上翻翻可以看到一个`private void SetActionStyle(ReApp app)`的函数设置了按钮的状态，

这里是用switch语句判断添加的

![](https://attach.52pojie.cn/forum/202410/12/155755x1mz25x2f0ps2s92.png)

35.png (20.96 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODQwM3w2ZDhmYWNjYnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwM3w2ZDhmYWNjYnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

我们点击这个成员变量，找到他定义的地方（这里直接点分析是不行的，因为他是初始化过的一个`Map`（字典）），要找往字典添加的方法调用，所以先跳转到它定义的地方，因为是个私有变量，所以肯定在这个类内部添加的数据，在代码搜索`btnActions`

![](https://attach.52pojie.cn/forum/202410/12/155757stwwvv3jvw3vnirj.png)

36.png (40.45 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODQwNHw1YzZjNTUzYnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwNHw1YzZjNTUzYnwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

很容易就能找到赋值的地方，关键点就是这个`rApp.Status == 1`，那我们就把这个判断去掉，右键，编辑IL指令

![](https://attach.52pojie.cn/forum/202410/12/155759tdtl5rw9ssw7zdf5.png)

37.png (60.52 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODQwNXw1YjM4MjgwOXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwNXw1YjM4MjgwOXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:57 上传

修改后的C#代码如下

![](https://attach.52pojie.cn/forum/202410/12/155801onq5an800z585t5u.png)

38.png (34.7 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcyODQwNnwzYmM0YWZlNXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcyODQwNnwzYmM0YWZlNXwxNzkzNTcyMDAwfDE2ODA4NnwxOTUwNDQx&nothumb=yes)

2024-10-12 15:58 上传

这样就没有状态判断了，默认都是已经购买了的。

我们保存打开看看效果（和上面图一样，我就不贴了），可以看到下载了的软件默认就是打开按钮，不需要再次下载了，至此，分析结束。

### 4、后语

> 具体的IL指令代表的什么意思就不解释了，可以百度或者问GPT都可以得到很好地回答。

这个软件分析没花多久，我个人认为这个软件比较简单，很适合新手练手，不管是Patch技巧还是抓包姿势都很适合。但是还是那句话，只做分析讨论，不提供成品，如有需求请支持正版。
