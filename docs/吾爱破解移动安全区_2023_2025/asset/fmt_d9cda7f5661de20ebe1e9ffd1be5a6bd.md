
```javascript
function readInputStream(inputStream) {
    var BufferedReader = Java.use("java.io.BufferedReader");
    var InputStreamReader = Java.use("java.io.InputStreamReader");
    var StringBuilder = Java.use("java.lang.StringBuilder");
    var reader = BufferedReader.$new(InputStreamReader.$new(inputStream));
    var builder = StringBuilder.$new();
    var line;
    while ((line = reader.readLine()) !== null) {
        builder.append(line);
    }
    reader.close();
    return builder.toString();
}
});
```

目标应用使用 `OkHttp`

```javascript
Java.perform(function() {
    var OkHttpClient = Java.use("okhttp3.OkHttpClient");
    var Request = Java.use("okhttp3.Request");
    var Response = Java.use("okhttp3.Response");

    // Hook OkHttpClient.newCall
    OkHttpClient.newCall.implementation = function(request) {
        console.log("Request URL:", request.url().toString());
        console.log("Request Headers:", request.headers().toString());

        var response = this.newCall(request).execute();
        console.log("Response Code:", response.code());
        console.log("Response Body:", response.body().string());
        return response;
    };
});
```

## 4.4 绕过 SSL Pinning

SSL pinning（SSL 证书绑定）安全机制，防止中间人攻击，核心是将服务器的 SSL\TLS 证书或公钥硬编码到客户端应用中，客户端与特定的服务器通信，不是与拥有有效证书的服务器通信。

### 引入机制

1. 证书或公钥绑定，客户端在建立连接，检查服务器返回的证书或公钥是否与硬编码的值匹配
2. 防止中间人攻击，攻击者的证书与硬编码值不匹配

### 实现方式

1. 证书绑定：将服务器的完整证书（通常是 DER 或 PEM 格式）硬编码到客户端应用中。
2. 公钥绑定：将服务器的公钥（通常是公钥的哈希值）硬编码到客户端应用中。
3. 证书链绑定：将服务器证书链中的某个中间证书或根证书硬编码到客户端应用中。

### 绕过方式

1. 修改客户端代码：反编译应用代码，删掉 SSL Pinning 的实现代码并修改删除
2. 使用 `frida` 或者 `objection` 绕过：

#### 使用 Frida

`Frida` 使用脚本 Hook SSL Pinning 相关的函数；

```javascript
Java.perform(function() {
    var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var TrustManager = Java.registerClass({
        name: "com.example.TrustManager",
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() {
                return [];
            }
        }
    });

    var SSLContext = Java.use("javax.net.ssl.SSLContext");
    SSLContext.init.implementation = function(keyManager, trustManager, secureRandom) {
        this.init(keyManager, [TrustManager.$new()], secureRandom);
    };
});
```

#### 使用 objection

使用 `objection` 命令：

```
android sslpinning disable
```

## 4.5 动态修改内存

```javascript
var ptr = Module.findBaseAddress("libnative.so").add(0x1234);
Memory.writeUtf8String(ptr, "new_value");
```

## 4.6 处理反调试

目标应用检测 `Frida`，可以 hook 反调试函数

```javascript
Java.perform(function() {
    var System = Java.use("java.lang.System");
    System.getProperty.implementation = function(key) {
        if (key === "java.vm.name") {
            return "Dalvik"; // 伪装成 Dalvik 虚拟机
        }
        return this.getProperty(key);
    };
});
```
