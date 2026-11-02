
```csharp
public static bool validate()
{
    if (PDT.status != SoftReg.VALIDATE_STATUS.FUL && PDT.status != SoftReg.VALIDATE_STATUS.TEM)
    {
        if (PDT.status != SoftReg.VALIDATE_STATUS.FUL_PRIOD)
        {
            if (PDT.status == SoftReg.VALIDATE_STATUS.OUTOFDATE)
            {
                ToolBox.SHOW_AUTHENICATE_INFO(1);
                return true;
            }
            return PDT.status == SoftReg.VALIDATE_STATUS.ERR && false;
        }
    }
    return true;
}
```

通过代码分析我们得知如果有使用权限这个方法就返回`true`。

我们不做过多分析，后面会有详细说明。

## 修改关键代码绕过注册码验证

因为是读取`PDT.status`的状态，所以我们需要让这个值直接返回对应符合条件的授权类型。

1. 修改`method_5()`的赋值代码，直接返回授权
2. 修改`SoftReg`类中的`SoftReg.VALIDATE_STATUS Validate()`返回值
3. 找一个合适的地方对`PDT.status`进行赋值（这是废话，忽略忽略...）

我推荐使用第二种方法，也就是对`SoftReg.VALIDATE_STATUS Validate()`方法内代码进行修改，为什么呢？

第一种方法你99.999%的可能性无法通过修改代码方式编译，除非你使用IL代码，至于为什么后面会详细讲解。

第三种方法虽然是废话，但是如果你对这个软件的架构和执行逻辑已经非常熟悉的话是可以找得到相应的位置的，甚至可以通过HOOK的方式来实现（非本帖讨论范畴）。

比如我们在上一篇帖子提到的那个窗口的激活按钮事件里，我们直接把调用注册码窗体的代码删掉，改成赋值`PDT.status`。当然需要添加命名空间和相关代码，难度稍稍高一些，不过对于会C# Winform开发的程序员来说这都不是事，只不过每次运行软件都需要点击一下激活按钮。

我们开工。

dnspy右键修改方法，代码就一行：

![](https://attach.52pojie.cn/forum/202307/22/163605bg65yhfx56s8sggz.png)

`sshot-10.png` (13.81 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyODg2NXw3M2U3OTdhOHwxNzkzNTczMzY0fDE2ODA4NnwxODExNTU0&nothumb=yes)

2023-7-22 16:36 上传

编译一下，然后保存文件打开软件注册框看效果：

![](https://attach.52pojie.cn/forum/202307/22/163749fptd25d2p56gc1cq.png)

`sshot-12.png` (60.09 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyODg2NnxjMzBjNjRmZHwxNzkzNTczMzY0fDE2ODA4NnwxODExNTU0&nothumb=yes)

2023-7-22 16:37 上传

OK，没有任何问题，不用输入注册码直接就是正式版~Nice~

## 从插件功能授权验证视角分析

我们从注册码的过程中分析完了，我们再从宿主软件调用插件的视角看一下这个软件应该怎么过验证。

当然要把我们脱壳后的程序替换回来，变成最开始没有修改过的状态。

### 分析DLL主入口类

切换到`PDT.PDT`类中。

通过命名空间我们可以发现这里调用了大量的宿主软件引入库文件：

![](https://attach.52pojie.cn/forum/202307/22/164148zrzfksccjcrik3br.png)

`sshot-14.png` (28.68 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyODg2OXwyMjI4YWJhMHwxNzkzNTczMzY0fDE2ODA4NnwxODExNTU0&nothumb=yes)

2023-7-22 16:41 上传

这也是为什么我上面提到的那个99.99你不能通过代码方式编译的原因，因为这些文件在`PDT`的目录下根本不存在，所以我们要把宿主软件中的这些文件复制到插件目录下。

### 补全文件

具体复制哪些文件可以看这个插件的引用，用到什么就复制什么：

![](https://attach.52pojie.cn/forum/202307/22/163844w5ktvvcvr8f88nwh.png)

`sshot-11.png` (69.07 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyODg2N3w2M2M0NTQyM3wxNzkzNTczMzY0fDE2ODA4NnwxODExNTU0&nothumb=yes)

2023-7-22 16:38 上传

没有三角符号的就不需要复制，因为不是NET程序。

### 分析代码

![](https://attach.52pojie.cn/forum/202307/22/164209p0ju0a25zweofeob.png)

`sshot-13.png` (41.99 KB, 下载次数: 0)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYyODg3MHxjZTFiNjQ0MHwxNzkzNTczMzY0fDE2ODA4NnwxODExNTU0&nothumb=yes)

2023-7-22 16:42 上传

通过当前类中的代码我们可以发现，大部分方法中都进行了验证操作。

`if (PDT.validate())` 必须返回 `true` 才能执行对应的功能，否则就会调用 `show_validate()` 方法。

```csharp
public static void show_validate()
{
    PDT.fv = new FormSoftBuy();
    PDT.fv.StartPosition = FormStartPosition.CenterScreen;
    PDT.fv.ShowInTaskbar = false;
    Application.ShowModelessDialog(Application.MainWindow.Handle, PDT.fv, false);
}
```

通过代码可以看出只要不返回`true`就会调用注册窗口（前面分析过的，不再解释）。

所以我们要把`PDT.validate()`强制返回`True`。

把前面的代码全部删掉，直接一行代码搞定：

```csharp
public static bool validate()
{
    return true;
}
```

编译之后保存，看效果。

虽然依然是未注册状态，但是插件正常使用已经不受影响了。

当然我们也可以给这个方法的条件改为满足条件，那就是上面讲到的直接赋值`PDT.status`。

好了，正向和逆向已经完全对接上了，打通了任督二脉，具体用什么招式的武功完全看个人喜好了。

:lol

## 实用小技巧

我们在使用修改代码的方式进行操作时，代码中标红的代码一般是因为无法读取解析上下文代码中的代码关系，所以我们就要去还原或者修补代码，上下打通了，才能顺利编译代码。

记得我提到的IL方式修改代码吗？这个不受上下文代码影响，但是要保证栈平衡，具体怎么操作呢？

我们在方法代码中右键选择编辑IL指令，然后把上面的代码全部NOP掉，只保留：

```il
ldc.i4.1
ret
```

关于具体的操作方法和说明请参考我以前的帖子：

[https://www.52pojie.cn/thread-1614580-1-1.html](https://www.52pojie.cn/thread-1614580-1-1.html)

[https://blog.csdn.net/zuoyuanchun/article/details/123822381](https://blog.csdn.net/zuoyuanchun/article/details/123822381)

对于C#逆向来说，学习一些必要的IL指令知识和用法是非常必要的。对dnspy的编译器不可能像Visual Studio的编译器那样完善和强大，某些情况下只能用IL指令修改的方式来修改程序。

## 个人点评

对于这个软件，爆点是相当的多，所有关键验证参数都是用返回值，这样就给了逆向的人更多的操作空间。如果编写注册机，甚至都不需要拷贝代码，直接调用插件本身的方法返回函数就可以直接获取所有和注册相关数据，甚至是插件功能的返回数据。

对于初级程序员来说，通过引用的方式来实现获取数据。

对于高级程序员来说，甚至可以在不脱壳的情况下利用C#的反射功能来获取相关数据，甚至是修改内存数据来实现。

## 总结

虽然作为一个官方已经近乎放弃更新的插件，里面还是有很多值得我们学习借鉴的代码，比如操作数据库、宿主软件的插件API参数，和宿主软件的一些操作函数。

同时也为广大程序员提了个醒，软件验证爆点越多，软件越容易被逆向，毕竟基于NET开发的软件代码保护程度相对来说都比较低。

对于软件的逆向，有时候思路比技术更重要，有时候顺着来不行就倒着来，找因果关系，因果关系理清了，软件开发者挖的坑我们才能顺利避开。

逆向的目的不是为了白嫖资源，而是为了规避那些已经暴露的潜在风险。

创作不易，感谢大家的支持。
