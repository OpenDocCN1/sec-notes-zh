
# [Android 原创] 【CoCollider】让系统和应用适配如此简单

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1978212>
- 抓取时间：2026-11-01T16:21:35.397Z

在各平台应用开发过程中，随着业务的功能增加，不免会涉及到非公开的API依赖，针对某些应用或厂商系统的适配，每个版本都需要投入精力去排查，CoCollider 可以让我们的适配效率从几个星期提升到几小时即可完成。

项目已开源：

- [GitHub：fireyer](https://www.github.com/iofomo/fireyer)
- [GitHub：wing](https://www.github.com/iofomo/wing)

如果您也喜欢，别忘了给我们点个星。

![](https://attach.52pojie.cn/forum/202411/04/103356pgmrvofe0g4eguvg.jpg)

## 01. 写在前面

在Android平台（后续会支持iOS平台）日常开发过程中，不免会接触到一些 Hidden（或私有函数和类）的类，JVM层通过反射，Native则通过查找函数 symbol 来完成。特别是涉及系统Framework层和底层库较多的实现（特别是安全产品和系统工具类应用），成为每次系统和厂商ROM更新适配的重灾区，严重影响适配效率。

以往，开发同学要么先一个个自行排查代码后再提交测试，这个过程重复，耗时且容易遗漏。要么先让测试同学先进行用例覆盖测试，然后开发同学通过BUG来进行分析，这会导致整个团队效率更加低下。

由于我们虚拟化产品的特性，工程中涉及系统和应用适配的接口众多，工作量很大，主要有两部分：

- 如何快速寻找到系统新增的模块，感兴趣的可以查看我们的另一个开源项目【ASeeker】[GitHub：aseeker](https://www.github.com/iofomo/aseeker)
- 如何快速定位现有使用的API是否已发生变化，这就是【CoCollider】要解决的问题。

CoCollider 已经在我们内部使用，极大提升我们系统适配效率，原本需要几个星期的工作量，现在只需几小时便可适配完成，原理简单，堪称适配效率神器。Android 15发布在即，现在分享给大家。

CoCollider同样适用于其他平台（如：iOS）和某些深度定制的应用版本适配。

## 02. 我们需要

对于系统的API，我们的需求是：

- 类的属性是否有变化。
- 类的成员属性是否有变化。
- 类的方法属性是否有变化。
- Native库和函数是否存在。

对于开发工程师的工作量，首次需要在源码中添加注释标签，这个工作不可省略，却一劳永逸。后续需要在每次涉及适配的代码处加上`@CoCollider`标签就行。

Java代码标签格式：

```
格式：
// @CoCollider {class name},{-/+}{field name/method name},...

范例：
# 仅查看类是否变更
// @CoCollider android.utils.Abc
# 仅查看类所有成员是否变更
// @CoCollider android.utils.Abc,-*
# 查看类和某成员或方法是否变更（支持多个）
// @CoCollider android.utils.Abc,-mFile
// @CoCollider android.utils.Abc,-mFile,mName
// @CoCollider android/utils/Abc,-mFile
// @CoCollider android.utils.Abc,+getFile,-mName
// @CoCollider android/utils/Abc,+getFile,+getName,-mName

# 支持缺省自动填充：（按照顺序依次填充）
Class.forName("android.utils.Abc");// @CoCollider
# 等同于：// @CoCollider android.utils.Abc

ReflectUtils.getStaticField("android.utils.Abc", "mName");// @CoCollider ,-
# 等同于：// @CoCollider android.utils.Abc，-mName
```

Native 代码标签格式：

```
格式：

// @CoCollider ~{lib name},{-/+}{field name/method name},...

范例：

# 查看方法是否变更

// @CoCollider ~libc.so,+open

// @CoCollider ~/system/lib/libc.so,+open

// @CoCollider ~/system/lib/libc.so,+open,+close

# 支持缺省自动填充：（按照顺序依次填充）

utils_dlsym("libc.so", "open");// @CoCollider ~,+

# 等同于：// @CoCollider ~libc.so,+open

utils_dlsym("open");// @CoCollider ~libc.so,+

# 等同于：// @CoCollider ~libc.so,+open
```

## 03. 配置运行

- 电脑下载配置 `wing`。
- 手机安装 Fireyer，或集成 CoCollider 模块的应用，链接 `adb`。
- 扫描代码：

    ```
    # 命令格式：扫描当前目录下的代码（默认支持 h/c/cpp/java/kt/aidl，也可以追加文件类型）
    $ wing -cocollider scan
    $ wing -cocollider scan /home/space
    # 或
    $ python ./cocollider.py scan
    $ python ./cocollider.py scan /home/space
    # 在当前目录输出结果
    >>> cocollider-scan-20241023-112044.txt
    ```

- 解析运行：

    ```
    # 命令格式：调用 Fireyer 应用
    $ wing -cocollider run /home/cocollider-scan-20241023-112044.txt
    # 或
    $ python ./cocollider.py run /home/cocollider-scan-20241023-112044.txt
    # 在当前目录输出结果
    >>> cocollider-run-20241023-112044.txt
    ```

- 使用对比工具查看结果即可快速定位变更内容。

## 04. 扫描结果

`cocollider-scan-20241023-112044.txt` 中的文件内容为：

```
############################################################
= android.utils.Abc

- mFile
> application/fireyer/test/test1.java,14

- mName3
> application/fireyer/test/test1.java,14

- mPath
> application/fireyer/test/test1.java,14

############################################################
= android.utils.Abc1

- mFile
> application/fireyer/test/test1.java,5

...
```

## 05. 运行结果

`cocollider-run-20241023-112044.txt` 中的文件内容（已对class，lib库，function，field和method排序，便于使用对比工具对比）

```
############################################################

~ libwilhelm.so

[OK]

+ _ZN7androidxx6BufferE

[OK], public final class libcore.io.Linux

> cmpt/xxx/jni/xxx/jni/src/xxx.cpp,362

+ _ZN7android1xx6BufferE

[OK], public final class libcore.io.Linux

> cmpt/xxx/jni/xxx/jni/src/xxx.cpp,350

############################################################

= ohos.abilityshell.HarmonyApplication

[Fail]

- applicationHandler

[Fail]

> scene/xxx/xxx.java,36

############################################################

= ohos.system.Parameters

[Fail]

+ nativeGet

[Fail]

> scene/xxx.cpp,250

...
```

## 06. 对比结果

分别在不同版本系统运行 `python cocollider.py run` 之后，使用对比工具（如：Beyond Compare）对内容进行比较查看，可以快速找到新增，修改和删除项，从而可以快速进行排查和修复。

![](https://attach.52pojie.cn/forum/202411/04/103419qdyzt8y0jymjb1om.jpg)
