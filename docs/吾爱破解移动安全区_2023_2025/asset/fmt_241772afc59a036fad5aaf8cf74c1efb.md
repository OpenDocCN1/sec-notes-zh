
```java
public void isError(String s) {
    Log.e("zj595", s);
    MainActivity.this.checkPassword(s);
}

@Override  // com.example.gesturelock.IGestureListener
public void isSetUp(String s) {
}

@Override  // com.example.gesturelock.IGestureListener
public void isSuccessful(String s) {
    Log.e("zj595", s);
}
});
}
}
```

## 程序逻辑

程序逻辑还是比较明确的，复制一份`classes.dex`，命名为`1.dex`，动态加载 dex 文件中`com.zj.wuaipojie2024_2.C`类中的`isValidate`函数，传入图形密码字符串，读取返回字符串，如果返回字符串以“诶！”开头，则隐藏密码锁，展示该字符串。

## 关键函数分析

### `isValidate`

接下来分析这个`isValidate`函数，同样转为 java 方便观看：

```java
public static String isValidate(Context context0, String s, int[] arr_v) throws Exception {
    try {
        return (String)C.getStaticMethod(context0, arr_v, "com.zj.wuaipojie2024_2.A", "d", new Class[]{Context.class, String.class}).invoke(null, context0, s);
    }
    catch(Exception exception0) {
        Log.e("ZJ595", "咦，似乎是坏掉的dex呢！");
        exception0.printStackTrace();
        return "";
    }
}
```

`isValidate`这个函数调用了同文件下的`getStaticMethod`函数。

### `getStaticMethod`

```java
private static Method getStaticMethod(Context context0, int[] arr_v, String s, String s1, Class[] arr_class) throws Exception {
    try {
        File file0 = C.fix(C.read(context0), arr_v[0], arr_v[1], arr_v[2], context0);
        ClassLoader classLoader0 = context0.getClass().getClassLoader();
        File file1 = context0.getDir("fixed", 0);
        Method method0 = new DexClassLoader(file0.getAbsolutePath(), file1.getAbsolutePath(), null, classLoader0).loadClass(s).getDeclaredMethod(s1, arr_class);
        file0.delete();
        new File(file1, file0.getName()).delete();
        return method0;
    }
    catch(Exception exception0) {
        exception0.printStackTrace();
        return null;
    }
}
```

`getStaticMethod`函数又调用了`fix`函数对 dex 文件进行修复，生成`2.dex`后加载修复后的`com.zj.wuaipojie2024_2.A`类中的`d`函数至`method0`，然后删除生成的`2.dex`文件后返回（此处为重点）。

### `fix` 与 `read`

注：修复前的`A`类是这样的：

```java
public class A {
    private static final String SUCCESS_TAG = "唉!";

    public static boolean b() {
        return false;
    }

    public static String c(String s) {
        return "?" + s + "?";
    }

    public static String d(Context context0, String s) {
        return "?" + s + "?";
    }
}
```

`fix`函数以及与之对应的`read`函数是这样的：

```java
private static File fix(ByteBuffer byteBuffer0, int v, int v1, int v2, Context context0) throws Exception {
    try {
        File file0 = context0.getDir("data", 0);
        int v3 = (int)(((Integer)D.getClassDefData(byteBuffer0, v).get("class_data_off")));
        HashMap hashMap0 = D.getClassData(byteBuffer0, v3);
        ((int[][])hashMap0.get("direct_methods"))[v1][2] = v2;
        byte[] arr_b = D.encodeClassData(hashMap0);
        byteBuffer0.position(v3);
        byteBuffer0.put(arr_b);
        byteBuffer0.position(0x20);
        byte[] arr_b1 = new byte[byteBuffer0.capacity() - 0x20];
        byteBuffer0.get(arr_b1);
        byte[] arr_b2 = Utils.getSha1(arr_b1);
        byteBuffer0.position(12);
        byteBuffer0.put(arr_b2);
        int v4 = Utils.checksum(byteBuffer0);
        byteBuffer0.position(8);
        byteBuffer0.putInt(Integer.reverseBytes(v4));
        byte[] arr_b3 = byteBuffer0.array();
        File file1 = new File(file0, "2.dex");
        FileOutputStream fileOutputStream0 = new FileOutputStream(file1);
        fileOutputStream0.write(arr_b3);
        fileOutputStream0.close();
        return file1;
    }
    catch(Exception exception0) {
        exception0.printStackTrace();
        return null;
    }
}

private static ByteBuffer read(Context context0) {
    try {
        File file0 = new File(context0.getDir("data", 0), "decode.dex");
        if(!file0.exists()) {
            return null;
        }
        FileInputStream fileInputStream0 = new FileInputStream(file0);
        byte[] arr_b = new byte[fileInputStream0.available()];
        fileInputStream0.read(arr_b);
        ByteBuffer byteBuffer0 = ByteBuffer.wrap(arr_b);
        fileInputStream0.close();
        return byteBuffer0;
    }
    catch(Exception unused_ex) {
        return null;
    }
}
```

## chatgpt 对以上代码的分析

这段代码是一个 Android 应用中的一些 Java 代码，主要涉及到动态加载 Dex 文件的操作。下面我来逐个分析这几个函数的作用：

### `fix(ByteBuffer byteBuffer0, int v, int v1, int v2, Context context0)`

- 这个函数的作用是修复一个`ByteBuffer`中的 Dex 文件，其中包括了修改 Dex 文件中的一些数据。
- 参数：
  - `ByteBuffer byteBuffer0`：包含 Dex 文件内容的缓冲区。
  - `int v, int v1, int v2`：用于指定要修改的数据的位置和值。
  - `Context context0`：上下文对象，用于获取应用程序的目录等信息。
- 主要步骤：
  - 解析 Dex 文件中的类数据和方法数据。
  - 根据传入的参数修改方法数据中的某个值。
  - 将修改后的数据写回`ByteBuffer`中。
  - 计算并更新 Dex 文件的`SHA1`校验和和校验码。
  - 将修改后的`ByteBuffer`写入到应用程序的私有目录中，并返回该文件。

### `getStaticMethod(Context context0, int[] arr_v, String s, String s1, Class[] arr_class)`

- 这个函数的作用是获取一个静态方法。
- 参数：
  - `Context context0`：上下文对象。
  - `int[] arr_v`：一个整型数组，用于指定修复 Dex 文件时所需的参数。
  - `String s, String s1`：要调用的类名和方法名。
  - `Class[] arr_class`：方法参数的类型数组。
- 主要步骤：
  - 调用`fix()`函数修复 Dex 文件，并获取修复后的文件。
  - 使用修复后的 Dex 文件创建一个`DexClassLoader`。
  - 加载指定类名的类，并获取其中声明的静态方法。
  - 删除修复后的文件和其在私有目录中的引用。
  - 返回获取到的静态方法。

### `isValidate(Context context0, String s, int[] arr_v)`

- 这个函数的作用是验证某个字符串。
- 参数：
  - `Context context0`：上下文对象。
  - `String s`：要验证的字符串。
  - `int[] arr_v`：一个整型数组，用于指定修复 Dex 文件时所需的参数。
- 返回值：返回验证结果的字符串。
- 主要步骤：
  - 调用`getStaticMethod()`函数获取指定类的静态方法。
  - 调用获取到的静态方法来进行验证，并返回结果。

### `read(Context context0)`

- 这个函数的作用是读取 Dex 文件。
- 参数：
  - `Context context0`：上下文对象。
- 返回值：返回读取到的 Dex 文件的`ByteBuffer`。
- 主要步骤：
  - 构造 Dex 文件的路径。
  - 读取 Dex 文件内容到一个字节数组中。
  - 将字节数组包装成`ByteBuffer`，并返回。

这些函数主要用于动态加载并修复 Dex 文件，然后执行其中的方法来实现某些功能。

## 思路

目前`flag`的关键在于两点，一个是图形锁代表的字符串`s`，一个是修复 dex 以后两个“?”的内容。

## 动态分析 & dex 修复 \*3

### 修复1

静态分析结束，下面用 010Editor 打开`classes.dex`，图中可以明显看到 dex 的`Adler32`校验值和`sha1`校验值是错误的。

也就是说附件里的`classes.dex`文件是损坏的。

dex 校验值损坏后，在反射加载类过程中会被系统拒绝，因此必须进行修复操作。

读取`logcat`可见如下错误：

> E System : Unable to Load dex file: `/data/user/0/com.zj.wuaipojie2024_2/app_data/1.dex`
>
> E System : java.io.IOException: Failed to open dex files : from `/data/user/0/com.zj.wuaipojie2024_2/app_data/1.dex` because: Failure to verify dex file '`/data/user/0/com.zj.wuaipojie2024_2/app_data/1.dex`': Bad Checksum (c607ea12, expected 22dcea4c)

可见系统确实因校验值异常，拒绝加载了`assets`下复制的`classes.dex`。

![0bd162d9f2d3572c6594c4e5cc13632762d0c391.png（107.36 KB，下载次数: 0）](https://attach.52pojie.cn/forum/202410/30/172842g7spsmqz76399zb2.png)

`0bd162d9f2d3572c6594c4e5cc13632762d0c391.png`（107.36 KB，下载次数: 0）

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjE5OXw2YjM1ZjY3YXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:28 上传

由于 dex 文件在 010Editor 中除了头部外并无明显异常情况，因此我们使用[DexRepair](https://github.com/luoyesiqiu/DexRepair)对 dex 头部进行修复：`java -jar DexRepair.jar /path/to/dex`

修复后头部变为全绿：

![30adcbef76094b366d7e40d4e5cc7cd98d109d1f.jpg（151.14 KB，下载次数: 0）](https://attach.52pojie.cn/forum/202410/30/172844de1e8yefec913epj.jpg)

`30adcbef76094b366d7e40d4e5cc7cd98d109d1f.jpg`（151.14 KB，下载次数: 0）

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIwMHxkOGZiN2VjMXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:28 上传

将修复完的 dex 命名为`classes.dex`，置入`assets`文件夹，重新打包、签名。

打开 jeb 调试，发现依然无法正常运行。

### 修复2

单步调试，定位到错误发生点，发现是程序在：

```java
try {
    File file0 = new File(context0.getDir("data", 0), "decode.dex");
    if(!file0.exists()) {
        return null;
    }
}
```

处返回了`null`，即这个`file0`，也就是`decode.dex`不存在。

然后程序跳转到了：

```java
catch(Exception exception0) {
    Log.e("ZJ595", "咦，似乎是坏掉的dex呢！");
    exception0.printStackTrace();
    return "";
}
```

打开`adb logcat`工具可见返回的错误信息：

> E ZJ595 咦，似乎是坏掉的dex呢！

可见确实是文件不存在的原因导致程序运行异常。
