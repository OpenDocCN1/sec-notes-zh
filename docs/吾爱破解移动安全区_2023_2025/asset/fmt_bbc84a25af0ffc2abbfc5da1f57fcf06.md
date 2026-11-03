
```
Java.perform(function () {
    var XhsHttpInterceptor = Java.use('p.d0.v1.e0.n0.h');
    var Buffer = Java.use("okio.Buffer");
    var Charset = Java.use("java.nio.charset.Charset");

    XhsHttpInterceptor.intercept.overload('okhttp3.Interceptor$Chain').implementation = function (chain, j2) {
        console.log('\n--------------------请求来了--------------------');
        var request = chain.request();

        var urlString = request.url().toString();
        console.log("网址：")
        console.log(urlString)
        console.log("\n请求头：")
        console.log(request.headers().toString());

        var requestBody = request.body();
        if (requestBody) {
            var buffer = Buffer.$new();
            requestBody.writeTo(buffer);
            console.log("请求体：")
            // 有可能是乱码。用h
            console.log(buffer.readString(Charset.forName("utf8")));
        }

        var res = this.intercept(chain);
        return res;
    };
})
//  frida -UF  -l next_request.js
```

#### 4> hook `Map`

`TreeMap`

```
Java.perform(function () {

    var TreeMap = Java.use('java.util.TreeMap');

    var Map = Java.use("java.util.Map");

    TreeMap.put.implementation = function (key,value) {

        if(key=="xxx"){ // 根据需要 看抓包

            console.log(key,value);

        }

        var res = this.put(key,value);

        return res;

    }

});

// frida -UF -l hook.js

// frida -U -l hook.js -f com.xx.xx --no-pause
```

#### 通用脚本

```
Java.perform(function () {
    function showMap(title,map){
        var result = "{";
        var keyset = map.keySet();
        var it = keyset.iterator();
        while(it.hasNext()){
            var keystr = it.next().toString();
            var valuestr = map.get(keystr).toString();
            result += '"' + keystr + '"';
            result += ":";
            result +=  '"' + valuestr + '"';
            result += ",";
        }
        result += "}";
        console.log(title, result);
    }

    // 这里是使用的例子，不要全抄过去了
    var EncryptTool = Java.use("com.jingdong.common.network.encrypt.EncryptTool");
    EncryptTool.encrypt.implementation = function(map){
        console.log("-----------------------");
        showMap("map字典->", map);
        var res = this.encrypt(map);
        console.log('返回值-->',res);
        // console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));
        return res;
    }
});

// frida -UF -l hook.js
// frida -U -l hook.js -f com.xx.xx --no-pause
```

#### 使用cast转换

```
Java.perform(function () {

    var EncryptTool = Java.use("com.jingdong.common.network.encrypt.EncryptTool");

    var HashMap = Java.use("java.uril.HashMap");

    EncryptTool.encrypt.implementation = function(map){

        console.log("-----------------------");

                console.log("map=", Json.stringfy(map));//map=""

        console.log("map=", map.toString());//map=[object Object]

        var real_object = Java.cast(map, HashMap);

        console.log("map=", real_object.toString());//map={d_bodel=Pixel4,wifiBssidunknown...}

        var res = this.encrypt(map);

        console.log('返回值-->',res);

        // console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));

        return res;

    }

});
```

#### 4> hook `StringBuilder`

```
Java.perform(function () {
    var StringBuilder = Java.use("java.lang.StringBuilder");
    StringBuilder.toString.implementation = function () {
        var res = this.toString();
        console.log(res); 
        return res;
    }
});
```

#### 4> hook `Base64`

AES常与base64搭配

```
Java.perform(function () {

    var Base64 = Java.use("android.util.Base64");

    Base64.encodeToString.overload('[B', 'int').implementation = function (bArr,val) {

        var res = this.encodeToString(bArr,val);

        console.log(" base64 res ->",res);

        // console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));

        return res;

    }

});
```

#### 4> hook构造方法

```
import javax.crypto.spec.SecretKeySpec;
import javax.crypto.spec.IvParameterSpec
...
public final class b {
    public final byte[] a(String body) {
        x.q(body, "body");
        try { // private static final String b
            String str = b; // private static final String b
            Charset charset = com.bilibili.commons.c.b;
            x.h(charset, "Charsets.UTF_8");
            if (str != null) {
                byte[] bytes = str.getBytes(charset); // bytes from str
                // 将此String解码使用平台的默认字符集，并将结果存储到一个新的字节数组中的字节序列
                x.h(bytes, "(this as java.lang.String).getBytes(charset)");
                SecretKeySpec secretKeySpec = new SecretKeySpec(bytes, "AES"); // 系统包 from bytes
                String str2 = f22911c;
                Charset charset2 = com.bilibili.commons.c.b;
                x.h(charset2, "Charsets.UTF_8");
                if (str2 != null) {
                    byte[] bytes2 = str2.getBytes(charset2); // bytes2 from str2
                    x.h(bytes2, "(this as java.lang.String).getBytes(charset)");
                    IvParameterSpec ivParameterSpec = new IvParameterSpec(bytes2); // iv from bytes2
                    Charset charset3 = com.bilibili.commons.c.b;
                    x.h(charset3, "Charsets.UTF_8");
                    byte[] bytes3 = body.getBytes(charset3);
                    x.h(bytes3, "(this as java.lang.String).getBytes(charset)");
                    byte[] i = com.bilibili.droid.g0.a.i(secretKeySpec, ivParameterSpec, bytes3); // key, iv, byte
                    x.h(i, "AES.encryptToBytes(Secre…yteArray(Charsets.UTF_8))");
                    return i;
                }
                throw new TypeCastException("null cannot be cast to non-null type java.lang.String");
            }
            throw new TypeCastException("null cannot be cast to non-null type java.lang.String");
        } catch (Exception e2) {
            BLog.e(a, e2);
            Charset charset4 = com.bilibili.commons.c.b;
            x.h(charset4, "Charsets.UTF_8");
            byte[] bytes4 = body.getBytes(charset4);
            x.h(bytes4, "(this as java.lang.String).getBytes(charset)");
            return bytes4;
        }
    }
}
```

#### 5> hook key

```
Java.perform(function () {

    var ByteString = Java.use('com.android.okhttp.okio.ByteString');

    var SecretKeySpec = Java.use('javax.crypto.spec.SecretKeySpec');

    SecretKeySpec.$init.implementation = function (key, name) {

        console.lot("key->", ByteString.of(key).utf8());// 字节转成字符串

        var res = this.$init(key, name);

        return res;

    }

})
```

#### 5> hook iv

```
Java.perform(function () {
    var ByteString = Java.use('com.android.okhttp.okio.ByteString');
    var IvParameterSpec = Java.use('javax.crypto.spec.IvParameterSpec');
    IvParameterSpec.$init.implementation = function (bytes) {  // iv
        console.lot("iv->", ByteString.of(key).utf8());// 字节转成字符串
        var res = this.$init(bytes);
        return res;
    }
})
```

#### 4> aes相关

与上面重复了。只是格式化输出做得比较好。可用于下方提到的c构造类，在so中调用的java方法，在hook的时候直接按照hook java的方法来操作。`$init.overload`不一定有重载，这里是特例。看上面例子就没有重载。

```
Java.perform(function () {

    var ByteString = Java.use("com.android.okhttp.okio.ByteString");

    var SecretKeySpec = Java.use("javax.crypto.spec.SecretKeySpec");

    SecretKeySpec.$init.overload('[B', 'java.lang.String').implementation = function (key, name) {

        if (name === 'AES') {

            console.log("-----------------------SecretKeySpec---------------------------");

            console.log("AES key bytes=", JSON.stringify(key));

            console.log("AES key hex=", ByteString.of(key).hex());

            console.log("AES key str=", ByteString.of(key).utf8());

        }

        var res = this.$init(key, name);

        return res;

    }

    var IvParameterSpec = Java.use("javax.crypto.spec.IvParameterSpec");

    IvParameterSpec.$init.overload('[B').implementation = function (iv) {
```
