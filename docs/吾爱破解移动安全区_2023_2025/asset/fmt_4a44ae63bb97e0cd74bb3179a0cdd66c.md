
# `Unidbg` 补环境记录

## `Throwable` 对象创建

```
java.lang.UnsupportedOperationException: java/lang/Throwable->()V
        at com.github.unidbg.linux.android.dvm.AbstractJni.newObject(AbstractJni.java:755)
        at com.github.unidbg.linux.android.dvm.AbstractJni.newObject(AbstractJni.java:735)
```

关于 `Throwable` 相关的都不是特别好补，点进第一行日志，去看看 `unidbg` 遇到类似的情况怎么做的；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251204104617716.png)

这里稍微提一下，`AbstractJni` 里面是 `unidbg` 作者为我们实现的一些环境，绝大多数的环境都可以参考着来；

```
@Override
public DvmObject newObject(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
    switch (signature) {
        case "java/lang/Throwable->()V":{
            return vm.resolveClass("java/lang/Throwable").newObject(new Throwable());
        }
    }
    return super.newObject(vm, dvmClass, signature, varArg);
}
```

这里参考着来的，总体就是 `new` 一个 `Throwable` 对象，和它的方式类似，但是 `unidbg` 是没有内置 `Throwable` 的对象所以手动 `new` 一个出来，继续执行；

## 获取调用栈

```
java.lang.UnsupportedOperationException: java/lang/Throwable->getStackTrace()[Ljava/lang/StackTraceElement;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:186)
```

这个是获取调用栈信息的，他返回的是一个数组，我们尝试这么返回；

```
case "java/lang/Throwable->getStackTrace()[Ljava/lang/StackTraceElement;":{
    StackTraceElement[] elements = Thread.currentThread().getStackTrace();
    DvmObject[] objs = new DvmObject[elements.length];
    for (int i = 0; i >" + elements[i]);
    }
    return new ArrayObject(objs);
}
```

这么做是有隐患的，看看输出就知道了；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251204110635568.png)

我找到了一篇文章，这位大佬的方式是这样的：

```
return new ArrayObject(
    vm.resolveClass("java/lang/StackTraceElement").newObject(null),
    vm.resolveClass("java/lang/StackTraceElement").newObject(null),
    vm.resolveClass("java/lang/StackTraceElement").newObject(null)
);
```

感觉比上面的好一些，但是最保险的还是自己模拟一下调用栈情况，这里一切从简了；继续执行吧；

## `StackTraceElement` 获取类名

```
java.lang.UnsupportedOperationException: java/lang/StackTraceElement->getClassName()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:194)
```

这个好说，按照这种方式返回；

```
case "java/lang/StackTraceElement->getClassName()Ljava/lang/String;":{
    StackTraceElement obj = (StackTraceElement) dvmObject.getValue();
    return new StringObject(vm, obj.getClassName());
}
```

以后遇到这种环境都是这样来，某个对象的某个方法，没有好的方式就取出对象然后调方法，或者你直接返回空字符串也行，只要能够跑通，毕竟我们的目的是算法还原，继续执行；

在看报错之前我们先看看取到了什么；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251204111844521.png)

可以看出来不是一个正常的 `name`，这里我们可以直接返回一个正确的字符串，但还是那句话，算法还原才是目的；

## `String.replaceAll`

```
java.lang.UnsupportedOperationException: java/lang/String->replaceAll(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethodV(AbstractJni.java:419)
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethodV(AbstractJni.java:262)
```

这个方法我们手动实现一下就好；

```
@Override
public DvmObject callObjectMethodV(BaseVM vm, DvmObject dvmObject, String signature, VaList vaList) {
    switch (signature) {
        case "java/lang/String->replaceAll(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;":{
            // 获取原始字符串
            String original = (String) dvmObject.getValue();
            // 获取参数
            DvmObject regexObj = vaList.getObjectArg(0);
            DvmObject replacementObj = vaList.getObjectArg(1);
            if (regexObj == null || replacementObj == null) {
                return new StringObject(vm, original);
            }
            String regex = (String) regexObj.getValue();
            String replacement = (String) replacementObj.getValue();
            // 执行替换
            String result = original.replaceAll(regex, replacement);
            // 返回结果
            return new StringObject(vm, result);
        }
    }
    return super.callObjectMethodV(vm, dvmObject, signature, vaList);
}
```

继续执行；

## `ByteArrayOutputStream` 与 `GZIPOutputStream`

### `ByteArrayOutputStream` 构造

```
java.lang.UnsupportedOperationException: java/io/ByteArrayOutputStream->()V
        at com.github.unidbg.linux.android.dvm.AbstractJni.newObject(AbstractJni.java:755)
        at com.Samples.pdd.anti_token_64.newObject(anti_token_64.java:234)
```

又是另外的报错，和之前的类似；

```
case "java/io/ByteArrayOutputStream->()V":{
    ByteArrayOutputStream OutputMemoryBuffer = new ByteArrayOutputStream();
    return vm.resolveClass("java/io/ByteArrayOutputStream").newObject(OutputMemoryBuffer);
}
```

继续执行能看到一个 `gzip` 压缩相关的内容；

### `GZIPOutputStream` 构造

```
java.lang.UnsupportedOperationException: java/util/zip/GZIPOutputStream->(Ljava/io/OutputStream;)V
        at com.github.unidbg.linux.android.dvm.AbstractJni.newObject(AbstractJni.java:755)
        at com.Samples.pdd.anti_token_64.newObject(anti_token_64.java:235)
```

这个稍微换一种方式，但是原理一样；

```
case "java/util/zip/GZIPOutputStream->(Ljava/io/OutputStream;)V":{
    DvmObject obj = varArg.getObjectArg(0);
    OutputStream outputStream = (OutputStream) obj.getValue();
    try {
        return dvmClass.newObject(new GZIPOutputStream(outputStream));
    } catch (IOException e) {
        throw new IllegalStateException(e);
    }
}
```

继续执行；

### `GZIPOutputStream.write`

```
java.lang.UnsupportedOperationException: java/util/zip/GZIPOutputStream->write([B)V
        at com.github.unidbg.linux.android.dvm.AbstractJni.callVoidMethod(AbstractJni.java:987)
        at com.github.unidbg.linux.android.dvm.AbstractJni.callVoidMethod(AbstractJni.java:982)
```

这个和前面一个补法；

```
@Override
public void callVoidMethod(BaseVM vm, DvmObject dvmObject, String signature, VarArg varArg) {
    switch (signature) {
        case "java/util/zip/GZIPOutputStream->write([B)V":{
            OutputStream outputStream = (OutputStream) dvmObject.getValue();
            ByteArray array = varArg.getObjectArg(0);
            try {
                outputStream.write(array.getValue());
            } catch (IOException e) {
                throw new IllegalStateException(e);
            }
            return;
        }
    }
    super.callVoidMethod(vm, dvmObject, signature, varArg);
}
```

调用当前对象的某个方法嘛，前面提到过；

### `GZIPOutputStream.finish`

```
java.lang.UnsupportedOperationException: java/util/zip/GZIPOutputStream->finish()V
        at com.github.unidbg.linux.android.dvm.AbstractJni.callVoidMethod(AbstractJni.java:987)
        at com.Samples.pdd.anti_token_64.callVoidMethod(anti_token_64.java:278)
```

继续补上；

```
case "java/util/zip/GZIPOutputStream->finish()V":{
    GZIPOutputStream gzipOutputStream = (GZIPOutputStream) dvmObject.getValue();
    try {
        gzipOutputStream.finish();
    } catch (IOException e) {
        throw new IllegalStateException(e);
    }
    return;
}
```

后续报错基本上都是这个类型；

### `ByteArrayOutputStream.toByteArray`

```
java.lang.UnsupportedOperationException: java/io/ByteArrayOutputStream->toByteArray()[B
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:196)
```

补上就行；

```
case "java/io/ByteArrayOutputStream->toByteArray()[B": {
    ByteArrayOutputStream ByteArrayArg = (ByteArrayOutputStream) dvmObject.getValue();
    byte[] data = ByteArrayArg.toByteArray();
    // Inspector.inspect(data, "java/io/ByteArrayOutputStream->toByteArray()");
    return new ByteArray(vm, data);
}
```

补的时候注意点，这里位置换了；补完这个方法过后成功的出值了；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251204175713580.png)

## 文件访问

接下来看一看文件访问，借用龙哥的话：补文件访问在 `Unidbg` 补环境这项工作里的重要性仅次于补 `JNI` 环境，它有两个主要特征：

- 一是出现频率高；
- 二是容易被补漏；

我的习惯是不管什么样本我都会写上文件访问的代码，在我们补 `JNI` 环境的过程中其实就有一些输出，大致如下：

```
sana file open-->>stdin
sana file open-->>stdout
sana file open-->>stderr
sana file open-->>/dev/__properties__
sana file open-->>/proc/stat
sana file open-->>/system/build.prop
sana file open-->>/proc/version
sana file open-->>/proc/13352/status
```

这些文件比较普遍的是后面的三个，`build.prop` 是一个属性文件，`version` 可能就是什么版本，`status` 主要就是看调试状态，这些其实我们好像都有进行手动返回，一般来说这里不补其实也没什么；但是这个算法入参其实就是指纹数据，所以需要注意一下，我们先不补文件访问，还记得 `gzip` 有一个 `write` 方法嘛？我们打印一下写入了什么；
