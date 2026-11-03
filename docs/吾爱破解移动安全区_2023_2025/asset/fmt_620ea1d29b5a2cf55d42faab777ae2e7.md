
```java
package cn.fxlcy.anative;

import android.annotation.SuppressLint;
import android.content.Context;
import android.content.pm.PackageManager;
import android.os.Build;
import android.util.Base64;
import androidx.annotation.Keep;
import androidx.core.app.Person;
import com.kuaishou.weapon.p0.t;
import f.b.a.a;
import f.b.a.b;
import f.b.a.c;
import f.b.a.d;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.security.MessageDigest;
import okhttp3.Interceptor;
import okhttp3.Response;
import org.json.JSONException;
import org.json.JSONObject;

/* loaded from: classes.dex */
public class Native {

    @Keep
    public static final Object ALOCK = new Object();

    @SuppressLint({"StaticFieldLeak"})
    public static a a = null;
    public static c b = null;
    public static String c = "";
    public static String d = null;
    public static String e = "";

    /* renamed from: f, reason: collision with root package name */
    public static boolean f146f = true;

    public static Response a(Interceptor.Chain chain) throws IOException {
        return b.f(chain);
    }

    public static String b(String str) {
        return new String(Base64.decode(str, 0));
    }

    public static byte[] c(byte[] bArr) {
        if (bArr == null) {
            return null;
        }
        return b.e(bArr);
    }

    public static Response d(Interceptor.Chain chain) throws IOException {
        try {
            synchronized (ALOCK) {
                if (f146f) {
                    l();
                    f146f = false;
                }
            }
            Response responseA = d.c.a(chain);
            if (responseA == null) {
                responseA = a(chain);
            }
            return responseA == null ? chain.proceed(chain.request()) : responseA;
        } catch (Throwable th) {
            if (th instanceof IOException) {
                throw th;
            }
            throw new IOException(th);
        }
    }

    public static String[] e(String str, boolean z) {
        return b.g(str, z);
    }

    public static String f() {
        return "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3VLHgbkFN0ebMaR4e0Dz6Z2mFexPBFKGqK0tuRhzu7XOrG92nKWfnublf2p1i22UN81whBLINjMttOuqW6fM9DCnAPTelud1zCXWYWIsv5Z19inJSG8vytJ7xg1dnfuRSRUkx11IE7bm0T/sM0sI4GgcktQJNSizyirHtuJjUUxxQabEhFkFeqQ5r+A69KjB5QkotCc4pG5lENyTARHGSsfaiJthaiH0yJ/8tUlyMgJ9H6/jbQg0wlLcEUzdfe2KuCPrTRzIzx4Cjm1JogT6JV2byvXpzAMC3O48LDiekJdVztg2Cj7E0cGrOsGs+IK6F7TWsKD/cIELTFhLz6dExQIDAQAB";
    }

    public static String g() {
        return e;
    }

    public static void h(Context context) {
        b = new c("3VLHgbkFN0ebMaR4e0Dz6Z2m".getBytes(), "FexPBFKG".getBytes(), Base64.decode("MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3VLHgbkFN0ebMaR4e0Dz6Z2mFexPBFKGqK0tuRhzu7XOrG92nKWfnublf2p1i22UN81whBLINjMttOuqW6fM9DCnAPTelud1zCXWYWIsv5Z19inJSG8vytJ7xg1dnfuRSRUkx11IE7bm0T/sM0sI4GgcktQJNSizyirHtuJjUUxxQabEhFkFeqQ5r+A69KjB5QkotCc4pG5lENyTARHGSsfaiJthaiH0yJ/8tUlyMgJ9H6/jbQg0wlLcEUzdfe2KuCPrTRzIzx4Cjm1JogT6JV2byvXpzAMC3O48LDiekJdVztg2Cj7E0cGrOsGs+IK6F7TWsKD/cIELTFhLz6dExQIDAQAB", 0));
        new b(context);
        a = new a(context, false);
        d = context.getFilesDir().getAbsolutePath();
        PackageManager packageManager = context.getPackageManager();
        String packageName = context.getPackageName();
        try {
            e = k((Build.VERSION.SDK_INT >= 28 ? packageManager.getPackageInfo(packageName, 134217728).signingInfo.getApkContentsSigners() : packageManager.getPackageInfo(packageName, 64).signatures)[0].toCharsString().getBytes());
        } catch (Exception e2) {
            throw new RuntimeException(e2);
        }
    }

    public static long i() {
        return System.currentTimeMillis() / 1000;
    }

    public static String j(byte[] bArr) {
        StringBuilder sb = new StringBuilder();
        for (byte b2 : bArr) {
            sb.append(String.format("%02x", Byte.valueOf(b2)));
        }
        return sb.toString();
    }

    public static String k(byte[] bArr) {
        try {
            return j(MessageDigest.getInstance("MD5").digest(bArr));
        } catch (Exception unused) {
            return "";
        }
    }

    public static void l() throws JSONException, IOException {
        String str;
        if (c.isEmpty()) {
            try {
                JSONObject jSONObjectC = a.c();
                String string = new JSONObject(a.b(b(jSONObjectC.getString(t.a)) + b(jSONObjectC.getString("z")) + b(jSONObjectC.getString("a")), "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCCX+ktw91mt0E6EgD+5DiWL8RZusboHw3a6PvINTjO6rRrO2Jv/AWVO9NatFn0ho5NGGg7UpHTVkgjYXWuapkEgvLNkQeqLX6ERHZomy2IiYo7xewiExf6zDVrDTDhCB1PRWPNROV7FtBMsiKrodVERC/+sra3XGgWOr1Ef8CsUwIDAQAB")).getString(Person.KEY_KEY);
                if (string.split("\n").length > 1) {
                    c = string.split("\n")[0];
                    str = string.split("\n")[1];
                } else {
                    c = string;
                    str = "android";
                }
                byte bD = a.d();
                File file = new File(d + "/vuk.kk");
                if (!file.exists()) {
                    file.createNewFile();
                }
                FileWriter fileWriter = new FileWriter(file);
                byte[] bytes = c.getBytes();
                for (int i2 = 0; i2
```
不看不知道，一看吓一跳！这个 `Native` 类里真是藏龙卧虎：

### `public static String f()`
这个方法让我大跌眼镜！它居然直接返回了一个硬编码的超长字符串！

```java
public static String f() {

    return "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3VLHgbkFN0ebMaR4e0Dz6Z2mFexPBFKGqK0tuRhzu7XOrG92nKWfnublf2p1i22UN81whBLINjMttOuqW6fM9DCnAPTelud1zCXWYWIsv5Z19inJSG8vytJ7xg1dnfuRSRUkx11IE7bm0T/sM0sI4GgcktQJNSizyirHtuJjUUxxQabEhFkFeqQ5r+A69KjB5QkotCc4pG5lENyTARHGSsfaiJthaiH0yJ/8tUlyMgJ9H6/jbQg0wlLcEUzdfe2KuCPrTRzIzx4Cjm1JogT6JV2byvXpzAMC3O48LDiekJdVztg2Cj7E0cGrOsGs+IK6F7TWsKD/cIELTFhLz6dExQIDAQAB";

}
```

这个字符串，和我最早在 `g.e.a.a` 配置文件里看到的 `AP.PKEYPRE` 的值一模一样！原来这个RSA公钥是被硬编码在这里，并通过 `Native.f()` 提供给其他代码使用的。在 `RtcClient.F()` 方法的 `preSignStr` 构造中，这个RSA公钥字符串就是作为 `&key=` 参数的值参与了签名计算。这意味着，这个RSA公钥字符串本身，也扮演了“盐”的角色，或者说是签名原始数据的一部分。

### `public static String m(String str)`
这个方法更是重量级！它的实现逻辑，简直就是为图片URL的 `key` 和 `time` 参数量身定做的！

```java
public static String m(String str) {
    String str2 = c; // c 是一个静态字段，就是我们要找的 vuk (密钥盐)
    if (str2 == null || str2.isEmpty()) {
        throw new RuntimeException("vuk is not initialized");
    }
    int i2 = 0;
    int i3 = 0;
    while (i2
```

这个方法接收一个字符串 `str`（我们推测是图片的完整URL），然后：
- 获取一个静态字段 `c` 的值（这个 `c` 就是核心密钥盐 `vuk`）。
- 从输入URL `str` 中截取从第三个斜杠开始的部分得到 `strSubstring`。
- 获取当前的秒级时间戳 `strValueOf`。
- 把 `strSubstring`、`strValueOf` 和 `vuk` 用 `@` 符号拼接起来。
- 对拼接后的字符串调用 `Native.k()`（即 `MD5` 函数）进行哈希。
- 最后返回 `MD5` 结果 + `:` + 秒级时间戳字符串。

这不就是图片URL里 `key`（冒号前的MD5值）和 `time`（冒号后的时间戳）的生成方式吗？！我感觉我离真相只有一步之遥了！

## 追踪核心密钥盐 `vuk` (`Native.c`) 的诞生

现在，所有的焦点都集中在了 `Native.m()` 方法中使用的那个神秘的静态字段 `c`（即 `vuk` 密钥盐）上。它是从哪里来的呢？

通过查看 `Native` 类的代码，我发现 `Native.c` 是在一个名为 `Native.l()` 的静态方法中被初始化的。这个 `l()` 方法的逻辑，简直就是一场寻盐之旅：
- `Native.l()` 会先判断 `Native.c` 是否为空，如果为空（意味着还未初始化），它才会执行后续的初始化逻辑。
- 它会调用 `f.b.a.a` 类的一个实例（这个实例在 `Native.h()` 中被创建并赋值给静态字段 `Native.a`）的 `c()` 方法，即 `a.c()`。
- `Frida` 闪亮登场：我们有动态分析工具 `Frida`！它可以让我们在App运行时“hook”住指定的方法，观察它的输入参数和返回值。这回算是一把hook住，顷刻炼化。我立刻编写了一个简单的 `Frida` 脚本来 hook `f.b.a.a.c()` 方法。

```javascript
Java.perform(function() {

  var D1X1Class = Java.use('f.b.a.a');

  D1X1Class.c.implementation = function() {

      console.log("[+] Entered f.b.a.a.c()");

      var originalResult = this.c();

      if (originalResult !== null) {

          console.log("[+] f.b.a.a.c() returned JSONObject: " + originalResult.toString());

      } else {

          console.log("[+] f.b.a.a.c() returned null");

      }

      return originalResult;

  };

});
```

运行App并附加 `Frida` 脚本后，控制台打印出了 `f.b.a.a.c()` 的返回值：

[+] `f.b.a.a.c()` returned `JSONObject`: {"a":"ajM5TCt...", "k":"ZHBKK1c...", "z":"dmV5VjN...", "p":true, "v":true}

![](https://attach.52pojie.cn/forum/202505/28/163042lt6tnexrj1berr6j.png)
