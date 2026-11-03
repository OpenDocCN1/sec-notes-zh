
```java
public void check(String hostname, List peerCertificates)

      throws SSLPeerUnverifiedException {

    // 查找与主机名匹配的哈希值列表（证书固定列表）。

    List pins = findMatchingPins(hostname);

    // 如果没有找到任何匹配的哈希值，则直接返回，表示无需进一步检查。

    if (pins.isEmpty()) return;

    // 如果存在证书链清理器，则先清理证书链中的冗余证书。

    if (certificateChainCleaner != null) {

      peerCertificates = certificateChainCleaner.clean(peerCertificates, hostname);

    }

    // 遍历每一个证书进行检查。

    for (int c = 0, certsSize = peerCertificates.size(); c 
```

### 实现方案:

```kotlin
//指纹检测  
fun check_SSL_PINNING_key() {
    // 使用CoroutineScope和Dispatchers.IO在后台线程中执行网络操作
    CoroutineScope(Dispatchers.IO).launch {
        // 定义需要固定证书的域名
        val caDomain = "www.52pojie.cn"
        // 使用CertificatePinner.Builder构建一个证书固定器
        // 添加一个sha256哈希值，这个值是服务器证书的指纹
        // 这个值需要通过openssl工具获取
        val pinner = CertificatePinner.Builder()
            .add(caDomain, "sha256/WnsD5UGdP5/a65xO1rpH8ru2EjyxkmPEaiNtKixhJLU=") // 添加证书指纹
            .build() // 构建完成证书固定器
        // 使用newBuilder创建一个新的OkHttpClient实例，用于覆盖默认的客户端配置
        val pClient = client.newBuilder()
            .certificatePinner(pinner) // 设置证书固定器
            .build() // 构建新的OkHttpClient实例
        // 构建一个网络请求，访问指定的URL
        val request = Request.Builder()
            .url("https://www.52pojie.cn/?q=SSLPinningCode") // 设置请求的URL
            .build() // 构建请求对象
        try {
            // 执行网络请求，并获取响应
            val response = pClient.newCall(request).execute()
            // 如果响应状态码是200，表示指纹检测通过
            Log.d(TAG, "指纹检测通过，状态码:${response.code()}")
        } catch (e: IOException) {
            // 如果发生IOException，表示指纹检测不通过
            Log.d(TAG, "指纹检测不通过")
            e.printStackTrace() // 打印异常堆栈信息，用于调试
        }
    }
}
```

安装openssl，[OpenSSL Windows 版本](https://slproweb.com/products/Win32OpenSSL.html?spm=5176.28103460.0.0.3a32572c9fMUi9)

![](https://pic.rmb.bdstatic.com/bjh/240824/860cb5670ca1da633d4f7cd69a9b024a8599.png)

### 配置环境变量

![](https://pic.rmb.bdstatic.com/bjh/240824/b66485d2d34c392a8f599b496a59f0fc3877.png)

`cmd`窗口输入以下命令获取

```bash
openssl s_client -connect www.52pojie.cn:443 -servername www.52pojie.cn | openssl x509 -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
```

![](https://pic.rmb.bdstatic.com/bjh/240824/693f228eb6a69046a950dd8b35992c6f3820.png)

### anti脚本

```javascript
function anti_ssl_key() {
        //check方法置空即可
    var okhttp3_Activity_1 = Java.use('okhttp3.CertificatePinner');   
    okhttp3_Activity_1.check.overload('java.lang.String', 'java.util.List').implementation = function(a, b) {                             
    console.log('[+] Bypassing SSL key pinning: ' + a);
    return;
}}
```

## 4> 2.证书校验

### 原理:

通过`trustManager`类实现的`checkServerTrusted`接口，核心在于验证服务器证书的公钥。具体步骤包括：获取服务器返回的证书，将其公钥编码为 `Base64` 字符串；同时从本地资源加载预存的可信客户端证书，并将其公钥也编码为 `Base64` 字符串。然后，比较这两个公钥是否匹配，以此确认服务器的身份是否合法。最后，使用自定义的 `SSLSocketFactory` 发起 HTTPS 请求，确保通信过程中只信任预定义的服务器证书，从而有效抵御中间人攻击。

### 实现方案:

```kotlin
// 定义一个函数用于检查SSL证书

private fun check_SSL_PINNING_CA() {

    // 创建一个X509TrustManager的匿名对象，用于自定义信任管理器

    val trustManager: X509TrustManager = object : X509TrustManager {

        // 客户端证书信任检查，这里不实现任何逻辑

        @Throws(CertificateException::class)

        override fun checkClientTrusted(chain: Array?, authType: String?) {

        }

        // 服务器证书信任检查

        @Throws(CertificateException::class)

        override fun checkServerTrusted(chain: Array, authType: String?) {

            // 获取服务器返回的第一个证书

            val cf: X509Certificate = chain[0]

            // 将服务器证书的公钥编码为Base64字符串

            val ServerPubkey: String = Base64.encodeToString(cf.publicKey.encoded, 0)

            Log.e(TAG, "服务器端返回的证书:" + ServerPubkey)

            // 从客户端资源中读取证书

            val client_input = resources.openRawResource(R.raw.wuai)

            val certificateFactory = CertificateFactory.getInstance("X.509")

            // 生成客户端证书

            val realCertificate: X509Certificate = certificateFactory.generateCertificate(client_input) as X509Certificate

            // 将客户端证书的公钥编码为Base64字符串

            val realPubkey: String = Base64.encodeToString(realCertificate.publicKey.encoded, 0)

            Log.e(TAG, "客户端资源目录中的证书:" + realPubkey)

            // 检查证书有效期

            cf.checkValidity()

            // 比较服务器证书和客户端证书的公钥是否相同

            val expected = realPubkey.equals(ServerPubkey, ignoreCase = true)

            if (!expected) {

                Log.e(TAG, "证书检测不通过")

            } else {

                Log.e(TAG, "证书检测通过")

            }

        }

        // 返回受信任的CA证书数组，这里返回空数组

        override fun getAcceptedIssuers(): Array {

            return arrayOfNulls(0)

        }

    }

    // 声明SSLSocketFactory变量

    var factory: SSLSocketFactory? = null

    try {

        // 获取SSL上下文

        val sslContext = SSLContext.getInstance("SSL")

        // 初始化SSL上下文，使用自定义的信任管理器

        sslContext.init(null, arrayOf(trustManager), SecureRandom())

        // 获取SSLSocketFactory

        factory = sslContext.socketFactory

    } catch (e: java.lang.Exception) {

        e.printStackTrace()

    }

    // 确保factory不为空

    val finalFactory: SSLSocketFactory? = factory

    // 创建并启动一个新线程来执行网络请求

    object : Thread() {

        override fun run() {

            try {

                // 使用自定义的SSLSocketFactory创建OkHttpClient

                val client =

                    OkHttpClient.Builder().sslSocketFactory(finalFactory, trustManager).build()

                // 构建请求

                val req = Request.Builder().url("https://www.52pojie.cn/forum.php").build()

                // 发送请求并获取响应

                val call: okhttp3.Call = client.newCall(req)

                val res: Response = call.execute()

                // 打印响应状态码

                Log.e("请求发送成功", "状态码：" + res.code())

            } catch (e: IOException) {

                // 打印网络异常信息

                Log.e("请求发送失败", "网络异常$e")

            }

        }

    }.start()

}
```

`cmd`窗口输入以下命令获取证书信息

```bash
openssl s_client -connect 52pojie.cn:443 -servername 52pojie.cn | openssl x509 -out wuai.pem
```

![](https://pic.rmb.bdstatic.com/bjh/240824/616433f141a1a49e76ff6be600dfcde46951.png)

### 证书信知识补充:

| 名词 | 含义 |
| --- | --- |
| `X.509` | 一种通用的证书格式，包含证书持有人的公钥、加密算法等信息 |
| `PKCS1~PKCS12` | 公钥加密（非对称加密）的一系列标准（Public Key Cryptography Standards），`.p12` 是包含证书和密钥的封装格式 |
| `*.der` | 证书的二进制存储格式（不常用） |
| `*.pem` | 证书或密钥的 Base64 文本存储格式，可以单独存放证书或密钥，也可以同时存放证书和密钥 |
| `*.key` | 单独存放的 `pem` 格式的私钥文件，一般保存为 `*.key` |
| `.cer / .crt` | 两者指的都是证书，Linux 下叫 `crt`，Windows 下叫 `cer`；存储格式可以是 `pem`，也可以是 `der` |
| `*.csr` | 证书签名请求（Certificate Signing Request），包含证书持有人的信息，如：国家、邮件、域名等 |
| `*.pfx` | 微软 IIS 的实现，包含证书和私钥 |

有的证书内容是只包含公钥（服务器的公钥），如`.crt`、`.cer`、`.pem`

有的证书既包含公钥又包含私钥（服务器的私钥），如`.pfx`、`.p12`

另外有些app的证书不走寻常路，不是上面所罗列到的格式，它有可能伪装成`png`等其他格式

![](https://pic.rmb.bdstatic.com/bjh/240824/08bd74f1969f4ad3e07239687aace37d1767.png)

### anti脚本

思路：实例化一个`trustManager`类，然后里面什么都不写，当上面两处调用到这个类时hook这两个地方，把自己定义的空`trustManager`类放进去

```javascript
function anti_ssl_cert() {

        // 使用Frida获取Java类X509TrustManager的引用
    var X509TrustManager = Java.use('javax.net.ssl.X509TrustManager');

    // 使用Frida获取Java类SSLContext的引用
    var SSLContext = Java.use('javax.net.ssl.SSLContext');

    // 注册一个自定义的TrustManager类
    var TrustManager = Java.registerClass({

        // 指定自定义TrustManager的全名
        name: 'dev.asd.test.TrustManager',

        // 指定自定义TrustManager实现的接口
        implements: [X509TrustManager],

        // 定义自定义TrustManager的方法实现
        methods: {

            // 客户端证书信任检查，这里不实现任何逻辑
            checkClientTrusted: function(chain, authType) {},

            // 服务器证书信任检查，这里不实现任何逻辑
            checkServerTrusted: function(chain, authType) {},

            // 返回受信任的CA证书数组，这里返回空数组
            getAcceptedIssuers: function() {return []; }

        }

    });

    // 准备一个TrustManager数组，用于传递给SSLContext.init()方法
    var TrustManagers = [TrustManager.$new()];

    // 获取SSLContext.init()方法的引用，该方法用于初始化SSL上下文
    var SSLContext_init = SSLContext.init.overload(
```
