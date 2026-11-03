
不存在我们就手动新建一个，为了方便起见我就直接复制了一份 `1.dex`，将其改名为 `decode.dex`，使用 RE 文件管理器放置于与 `1.dex` 同目录下（即 `data/user/0/com.zj.wuaipojie2024_2/app_data`），这样这部分代码就能正常跑起来了。

根据"静态分析"部分代码分析，由于 `2.dex` 文件在写入、加载后即会进行自我删除，于是需要在删除代码位置下断点以保存该中间文件进行分析，断点位置如下：

```
.method private static varargs getStaticMethod(Context, [I, String, String, [Class)Method
          .registers 11
          .annotation system Signature
              value = {
                  "(",
                  "Landroid/content/Context;",
                  "[I",
                  "Ljava/lang/String;",
                  "Ljava/lang/String;",
                  "[",
                  "Ljava/lang/Class;)",
                  "Ljava/lang/reflect/Method;"
              }
          .end annotation
          .annotation system Throws
              value = {
                  Exception
              }
          .end annotation
00000000  const/4             v0, 0
:try_2
00000002  invoke-static       C->read(Context)ByteBuffer, p0
00000008  move-result-object  v1
0000000A  const/4             v2, 0
0000000C  aget                v3, p1, v2
00000010  const/4             v4, 1
00000012  aget                v4, p1, v4
00000016  const/4             v5, 2
00000018  aget                p1, p1, v5
0000001C  invoke-static       C->fix(ByteBuffer, I, I, I, Context)File, v1, v3, v4, p1, p0
00000022  move-result-object  p1
00000024  invoke-virtual      Object->getClass()Class, p0
0000002A  move-result-object  v1
0000002C  invoke-virtual      Class->getClassLoader()ClassLoader, v1
00000032  move-result-object  v1
00000034  const-string        v3, "fixed"
00000038  invoke-virtual      Context->getDir(String, I)File, p0, v3, v2
0000003E  move-result-object  p0
00000040  new-instance        v2, DexClassLoader
00000044  invoke-virtual      File->getAbsolutePath()String, p1
0000004A  move-result-object  v3
0000004C  invoke-virtual      File->getAbsolutePath()String, p0
00000052  move-result-object  v4
00000054  invoke-direct       DexClassLoader->(String, String, String, ClassLoader)V, v2, v3, v4, v0, v1
0000005A  invoke-virtual      DexClassLoader->loadClass(String)Class, v2, p2
00000060  move-result-object  p2
00000062  invoke-virtual      Class->getDeclaredMethod(String, [Class)Method, p2, p3, p4
00000068  move-result-object  p2
0000006A  invoke-virtual      File->delete()Z, p1  #在此处下断点，防止文件删除
00000070  new-instance        p3, File
00000074  invoke-virtual      File->getName()String, p1
0000007A  move-result-object  p1
0000007C  invoke-direct       File->(File, String)V, p3, p0, p1
00000082  invoke-virtual      File->delete()Z, p3
          .catch Exception {:try_2 .. :tryend_88} :catch_8A
:tryend_88
00000088  return-object       p2
:catch_8A  # used for: Ljava/lang/Exception;
0000008A  move-exception      p0
0000008C  invoke-virtual      Exception->printStackTrace()V, p0
00000092  return-object       v0
.end method
```

然后在 `data/user/0/com.zj.wuaipojie2024_2/app_data` 下即可找到修复后的 `2.dex` 文件。

修复后的 `2.dex` 中的 `A` 类如下：

```
package com.zj.wuaipojie2024_2;

import android.content.Context;

public class A {
    private static final String SUCCESS_TAG = "唉!";

    public static boolean b() {
        return false;
    }

    public static String c(String s) {
        return "?" + s + "?";
    }

    public static String d(Context context0, String s) {
        MainActivity.sSS(s);
        String s1 = Utils.getSignInfo(context0);
        if(s1 != null && (s1.equals("fe4f4cec5de8e8cf2fca60a4e61f67bcd3036117"))) {
            StringBuffer stringBuffer0 = new StringBuffer();
            for(int v = 0; stringBuffer0.length() 
```

此处的 `A.d` 函数给出了密码 `s` 的真值以及一个提示。

写个小程序获取解锁密码 `s`：

```
import java.io.*;
public class GetPass {
    public static void main(String[] args) {
        StringBuffer stringBuffer0 = new StringBuffer();
        for(int v = 0; stringBuffer0.length() 
```

输出为：

> `048531267`（九宫格密码的真值，后续会用到）

![图片](https://attach.52pojie.cn/forum/202410/30/172847y4m0qlscmqao4wlm.png)

79f0f736afc37931342bc610adc4b74543a91143.png（4.48 KB，下载次数：0）

[href="forum.php?mod=attachment&aid=MjczMjIwMXxkMDNjMjFlNXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIwMXxkMDNjMjFlNXwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:28 上传

下面来看看提示：

> "唉!哪有什么亿载沉睡的玄天帝，不过是一位被诅咒束缚的旧日之尊，在灯枯之际挣扎的南柯一梦罢了。有缘人，这份机缘就赠予你了。坐标在`B.d`"

找到 `B` 类......

（怎么还是讨厌的 `"?" + s + "?"`）

```
public class B {
    public static String d(String s) {
        return "?" + s + "?";
    }
}
```

看来这个 `B.d` 还是坏的，还得修复一次。

## 5> 修复3

原先程序里的那个 `fix` 函数只能修复 `A.d` 部分，接下来修复 `B.d`。

回到 `MainActivity`，还是定位到这句：

```
String s1 = (String)new DexClassLoader(file0.getAbsolutePath(), file1.getAbsolutePath(), null, classLoader0).loadClass("com.zj.wuaipojie2024_2.C").getDeclaredMethod("isValidate", Context.class, String.class, int[].class).invoke(null, this, s, this.getResources().getIntArray(array.A_offset));
```

语句中出现了 `A_offset`，说明是修复 `A` 类型的，定位到 `array.A_offset` 这个变量：

```
public static final class array {
    public static int A_offset = 0x7F030000;  // array:A_offset
    public static int B_offset = 0x7F030001;  // array:B_offset
}
```

（诶，`A_offset` 下面那个 `B_offset` 是啥？这不是我们要找的“`B` 类”的偏移嘛，这下不用手算了）

直接 `jeb` 跑起来，在 `A_offeset` 处下断点，把 `A_offset` 的值换成 `B_offset`：

```
000000E4  sget                v3, R$array->A_offset:I  #此处下断点，修改Locals中v3的值为B_offset，即将其改为7F30001h，如图
000000E8  invoke-virtual      Resources->getIntArray(I)[I, p1, v3
```

![图片](https://attach.52pojie.cn/forum/202410/30/172849d533ouur13xv54gr.png)

8cb1cb1349540923360d2647d458d109b3de49a7.png（31.33 KB，下载次数：1）

[href="forum.php?mod=attachment&aid=MjczMjIwMnwxYWE1Y2ZlOHwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMjIwMnwxYWE1Y2ZlOHwxNzkzNTY4NTg4fDE2ODA4NnwxODg4ODgz&nothumb=yes)

2024-10-30 17:28 上传

运行程序，断点至本文"修复2"部分提及的删除函数，得到 `B.d` 修复后的 `2_new.dex`（此处为了避免与上部分中 `2.dex` 混淆，将其命名为 `2_new.dex`，实际由程序生成的文件名仍为 `2.dex`）。

打开 `2_new.dex`，反编译得到其中的 `B` 类，定位至 `B.d`：

```
public class B {
    public static String d(String s) {
        return "机缘是{" + Utils.md5(Utils.getSha1("password+你的uid".getBytes())) + "}";
    }
}
```

> `Flag："{" + Utils.md5(Utils.getSha1("password+你的uid".getBytes())) + "}"`

其中的 `password` 上面已经提到了，是 `"048531267"`。

到此，这个 CM 终于完事了。

此处特别感谢 正己 佬提供的题目以及指点。

## 4> 附——`Utils` 中的两个算法（由 `smali` 代码反编译得到，无法直接运行）：

```
    public static byte[] getSha1(byte[] arr_b) {
        try {
            return MessageDigest.getInstance("SHA").digest(arr_b);
        }
        catch(Exception unused_ex) {
            return null;
        }
    }
    public static String md5(byte[] arr_b) {
        int v;
        try {
            String s = new BigInteger(1, MessageDigest.getInstance("md5").digest(arr_b)).toString(16);
            v = 0;
            while(true) {
            label_2:
                if(v >= 0x20 - s.length()) {
                    return s;
                }
                s = "0" + s;
                break;
            }
        }
        catch(NoSuchAlgorithmException unused_ex) {
            throw new RuntimeException("ops!!");
        }
        ++v;
        goto label_2;
    }
```

## 4> 写个小程序计算一下 `flag`（注册机）

```
import java.io.*;
import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Scanner;

public class GetFlag {
    public static byte[] getSha1(byte[] arr_b) {
        try {
            return MessageDigest.getInstance("SHA").digest(arr_b);
        }
        catch(Exception unused_ex) {
            return null;
        }
    }

    public static String md5(byte[] arr_b) {
        try {
            String s = new BigInteger(1, MessageDigest.getInstance("md5").digest(arr_b)).toString(16);
            return s;
        }
        catch(NoSuchAlgorithmException unused_ex) {
            throw new RuntimeException("ops!!");
        }
    }

    public static void main(String args[]) {
        try (Scanner scanner = new Scanner(System.in)) {
            System.out.println("Please Input Your UID：");
            String s = "048531267" + scanner.next();
```
