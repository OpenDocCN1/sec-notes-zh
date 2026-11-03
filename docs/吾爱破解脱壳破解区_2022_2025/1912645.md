
# [.NET逆向] 某财务软件dnspy分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1912645
- 抓取时间：2026-11-01T22:33:17

某财务软件的旧版本（T+16），新版本不适应。

关键函数被加密处理，在内存中解密指令串并产生`DynamicMethod`调用，这部分代码做了函数名变量名混淆。

![](https://attach.52pojie.cn/forum/202404/12/084220rbnnqribdnrpborr.png)

附件1：67496425fbac3b74d097e407356691e.png (27.22 KB, 下载次数: 3) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODk4NHxkNTVhNDE4OXwxNzkzNTcyMzk1fDE2ODA4NnwxOTEyNjQ1&nothumb=yes)

上传于 2024-4-12 08:42

C# 中的`System.Reflection.Emit.DynamicMethod`类允许开发者在运行时动态地生成和执行托管代码。这意味着你可以在程序运行过程中创建一个新的方法，并且能够控制这个方法的IL（中间语言 Intermediate Language）代码，从而实现非常灵活的动态编程能力。

具体来说，`DynamicMethod`主要用途包括但不限于：

- 动态代{过}{滤}理：创建运行时代{过}{滤}理类以拦截方法调用，实现AOP（面向切面编程）的功能。
- 性能优化：在特定场景下，根据运行时条件生成优化过的代码，避免使用反射带来的性能损失。
- 动态编译表达式：类似于 LINQ Expression Trees，可以用来构建并执行复杂的表达式或逻辑，尤其是在需要高效执行的情况下。
- 兼容性层：在不同版本的.NET框架之间提供一种编写通用代码的方式，特别是在处理一些运行时特性不一致的情况时。
- 轻量级脚本引擎：作为简易的脚本环境的一部分，可以动态地执行用户定义的方法。

使用`DynamicMethod`创建的方法不需要与任何已存在的类型关联，它可以拥有指定的返回类型和参数列表，并可以选择绑定到特定的类型闭包（即该方法可以访问其所在模块或类型内的私有成员）。生成的方法可以直接通过`.Invoke()`方法来执行，无需事先编译为一个完整的类型或程序集。

例如，你可以用`DynamicMethod`实现两个对象间属性的快速拷贝，或者动态生成一个符合某种运算规则的方法。然而，这种灵活性也意味着使用时需谨慎，因为不当使用可能会导致运行时错误，同时由于绕过了编译器的静态类型检查，可能引入难以调试的问题。

解密产生的代码会放入字典中存储，下次直接调用。

![](https://attach.52pojie.cn/forum/202404/12/084318k40e64dbhzba4s13.png)

附件2：e5799f88bb25f363b3d01c43acc1717.png (50.75 KB, 下载次数: 3) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODk4NnwzMDJlZGRhMHwxNzkzNTcyMzk1fDE2ODA4NnwxOTEyNjQ1&nothumb=yes)

上传于 2024-4-12 08:43

这里没用`ILGenerator`的方式添加`ILCode`指令，而是直接恢复`ILCode`指令块字节。

先获得`DynamicMethod`的`DynamicILInfo`：

`DynamicMethod.GetDynamicILInfo()`

Returns a `DynamicILInfo` object that can be used to generate a method body from metadata tokens, scopes, and Microsoft intermediate language (MSIL) streams.

再调用`DynamicILInfo.SetCode()`设置代码字节数组：

Sets the code body of the associated dynamic method.

```csharp
public void SetCode (byte[]? code, int maxStackSize);
```

某处直接调用`DynamicILInfo.SetCode`恢复代码，省略代码解密部分。

![](https://attach.52pojie.cn/forum/202404/12/084345nb2sbbddbtssusbj.png)

附件3：06bdcacc764a98f9be78035e21dc7b4.png (39.88 KB, 下载次数: 2) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODk4N3xiOTcwYTAxNnwxNzkzNTcyMzk1fDE2ODA4NnwxOTEyNjQ1&nothumb=yes)

上传于 2024-4-12 08:43

这是存放动态生成函数的`DynamicMethod`

![](https://attach.52pojie.cn/forum/202404/12/084400ic4hkiukkkg5i72o.png)

附件4：35c71bec43560570039a2b5eb8036e7.png (36.47 KB, 下载次数: 3) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODk4OHwxNjcxY2UyNHwxNzkzNTcyMzk1fDE2ODA4NnwxOTEyNjQ1&nothumb=yes)

上传于 2024-4-12 08:44

这是从`DynamicMethod`中恢复出来的被加密的代码，可以对里面用到的函数作Hook，比如`GetUsers()`，使之返回我们想要的值。

![](https://attach.52pojie.cn/forum/202404/12/084419myyo77vno0hxhvu5.png)

附件5：32e750adec61384e4e374b6c148666d.png (62.66 KB, 下载次数: 4) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4ODk4OXwzNmIwMDA5MHwxNzkzNTcyMzk1fDE2ODA4NnwxOTEyNjQ1&nothumb=yes)

上传于 2024-4-12 08:44
