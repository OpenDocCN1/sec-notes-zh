
# [Android 原创] 记一次使用 Xposed RPC 和 BurpGuard 应对金融APP参数签名及加密的详细过程

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1964051
- 抓取时间: 2026-11-01T16:31:23.069Z

## 前言

本文首发于先知论坛：[https://xz.aliyun.com/t/15533](https://xz.aliyun.com/t/15533)

在一次金融渗透测试项目中需要对 APP 进行渗透，发现 APP 对参数进行了签名以及加密，于是便逆向 APP 并通过 Xposed RPC 和 BurpGuard 解决了问题，从中学到了许多，因此记录过程并分享一下思路，代码和图片均做了脱敏处理，同时省略了代码调试的过程。

阅读完本文，你可以学习到：

- APP 请求逆向思路
- 如何查阅资料
- APP 请求签名和加密的原理
- 了解白盒 `WbSM4`
- Xposed RPC 的实现思路
- BurpGuard 如何使用

声明：本文不针对任何 APP，仅对加密及签名技术进行研究，如有侵权，请联系删除。

## 详细过程

测试环境：`mumu`模拟器、`Magisk`、`Lsposed`

### 初探签名及加密

APP 的 HTTPS 验证就不阐述了，常规方式均可绕过。

查看请求包，发现了`MsgId`、`AppSign`以及密文`cTxt`：

![](https://attach.52pojie.cn/forum/202409/12/230035zgrv1zxla6lrrgt3.png)

修改`MsgId`，返回 401，很明显做了请求签名：

![](https://attach.52pojie.cn/forum/202409/12/230100ej5laj15egvpe1mp.png)

### 加密分析

这里 APP 采用了爱加密企业版加固，于是使用 算法助手Plus + frida 脚本脱壳，使用 Jadx 反编译，搜索关键词，发现没搜到：

![](https://attach.52pojie.cn/forum/202409/12/230120jf212qqx1n1u01do.png)

随后翻了翻，发现代码都抽空了，到 native 层去了？似乎是没脱完整：

![](https://attach.52pojie.cn/forum/202409/12/230122htf2njsjwx5za8pa.png)

求助某大佬帮忙脱了个壳，继续搜索关键词，找到两处，看到 ECB，莫非是常见的 AES-ECB 加密（心中窃喜）：

![](https://attach.52pojie.cn/forum/202409/12/230124rzzww2xn2ccfdrn4.png)

跟进去：

![](https://attach.52pojie.cn/forum/202409/12/230126reo2oseesjeeuoou.png)

继续跟 encryptECB：

![](https://attach.52pojie.cn/forum/202409/12/230129bh7qb8orx7h4hbnf.png)

跟进：

![](https://attach.52pojie.cn/forum/202409/12/230133tfhfhrfhfhruhjhh.png)

最终来到`xxx.WbSM4Util$Companion`类的`encryptDataECB`方法：

![](https://attach.52pojie.cn/forum/202409/12/230136k83718jnj10gb3gz.png)

这是对应的解密函数：

![](https://attach.52pojie.cn/forum/202409/12/230142dobmrbrprtrpc4ir.png)

接着看看加密函数中的关键部分`WbSm4().encode()`，这个函数传入的就是明文的字节对象和长度，并没有密钥相关，最后做了一层 Base64：

```
String encodeToString = Base64.encodeToString(new WbSm4().encode(bytes2.length, bytes2), 0, bytes2.length, 2);
```

跟进`WbSm4`一探究竟，发现是 native 的：

![](https://attach.52pojie.cn/forum/202409/12/230146dwwc89wbbyzazcdc.png)

网上搜寻一下看看是什么，似乎是 SM4 国密：

![](https://attach.52pojie.cn/forum/202409/12/230152gvn3nzxne1bgd83d.png)

问一下 GPT，白盒`SM4`是什么，看起来是很难复现的算法，密钥也很难提取，后续可能需要采用 RPC 远程调用 ：

![](https://attach.52pojie.cn/forum/202409/12/230201wrm25hdysyd2dsys.png)

根据上面的代码分析简单画一个图：

![](https://attach.52pojie.cn/forum/202409/12/230207muuk33a5ziyeeai7.png)

### 签名分析

接下来看看签名，搜索签名关键词`appsign`：

![](https://attach.52pojie.cn/forum/202409/12/230230hjh5dhyyo8ttl4jr.png)

跟进第一个的`WelfareTaskRequestService.APPSIGN`，是个常量：

![](https://attach.52pojie.cn/forum/202409/12/230238lyri0i0im0j9l0yi.png)

搜索常量名，看看哪一处使用了它，这里发现第二个是添加请求头，比较可疑：

![](https://attach.52pojie.cn/forum/202409/12/230302s5wfs84zsgvd4t86.png)

跟进代码如下：

```
String uuid = UUID.randomUUID().toString();

Request.Builder addHeader = new Request.Builder().url(ServiceUrls.getAccessKeyListUrl(ServerInfoMgr.getInstance().getDefaultServerInfo(204))).addHeader(RemoteMessageConst.MSGID, uuid).addHeader("deviceId", SysConfigs.DEVICE_ID).addHeader("agent", "android-" + YBHelper.getAppVersionName()).addHeader(WelfareTaskRequestService.APPSIGN, HeaderUtils.generateAppSign(ServiceUrls.getAccessKeyListUrl(ServerInfoMgr.getInstance().getDefaultServerInfo(204)), uuid, HeaderUtils.requestBodyToString(create)));
```

分析上面代码可知，`RemoteMessageConst.MSGID`是个字符串常量，值为`"MsgId"`，而签名是由静态方法`HeaderUtils.generateAppSign(url,uuid.requestBody)`生成的，其中三个参数分别为：完整 URL、`UUID`、完整请求 Body，可以看出请求 body 与`msgid`和`appsign`强相关，任何一个参数错误都可能导致请求不合法，以下是简单示意图：

![](https://attach.52pojie.cn/forum/202409/12/230309jiriub55rwa66nx9.png)

### Xposed RPC 实现

加密和签名都分析完毕，剩下的就是使用 RPC （远程过程调用）了，也就是注入代码，这里我用的是 Xposed 框架，没有检测，但 frida 有检测，过检测的方式参考下方公众号的文章（理论无脑过爱加密企业版），自己试了下确实可以：

[https://mp.weixin.qq.com/s/34c5JVJzSCEfqlPanV1FtA](https://mp.weixin.qq.com/s/34c5JVJzSCEfqlPanV1FtA)

对 Xposed 的 RPC 不太熟悉，网上查阅资料，找到如下文章：

[https://www.52pojie.cn/thread-1519322-1-1.html](https://www.52pojie.cn/thread-1519322-1-1.html)

发现使用`NanoHTTPD`在 APP 内部起一个 HTTP 服务器来与外部通信来实现 RPC 比较方便，写一个 demo，代码可以直接运行，会开启一个 50000 端口的 HTTP 服务器，开放`/encrypt`接口用于加密数据和生成签名，`/decrypt`接口用于解密数据：

```
import com.google.gson.JsonObject;  
import fi.iki.elonen.NanoHTTPD;  

import java.io.IOException;  
import java.util.HashMap;  
import java.util.Map;  

class HTTPServer extends NanoHTTPD {  

    public HTTPServer(int port) {  
        super(port);  
    }  

    @Override  
    public Response serve(IHTTPSession session) {  
        JsonObject responseJson = new JsonObject();  
        String encryptData = "";  
        String decryptData = "";  
        Map map = new HashMap<>();  
        try {  
            session.parseBody(map);  
        } catch (Exception e) {  
            return newFixedLengthResponse(Response.Status.INTERNAL_ERROR, "text/plain", e.getMessage());  
        }  
        if (session.getMethod() == Method.POST) {  
            switch (session.getUri()) {  
                case "/encrypt":  
                    responseJson.addProperty("encryptData", encryptData);  
                    return newFixedLengthResponse(Response.Status.OK, "application/json", responseJson.toString());  
                case "/decrypt":  
                    responseJson.addProperty("decryptData", decryptData);  
                    return newFixedLengthResponse(Response.Status.OK, "application/json", responseJson.toString());  
                default:  
                    return newFixedLengthResponse(Response.Status.NOT_FOUND, "text/plain", "Not Found");  
            }  
        } else {  
            return newFixedLengthResponse(Response.Status.NOT_FOUND, "text/plain", "Not Found");  
        }  
    }  
}  

public class Demo {  
    public static void main(String[] args) throws IOException {  
        HTTPServer httpServer = new HTTPServer(50000);  
        httpServer.start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);  
    }  
}
```

Xposed 模块使用的模板：[https://github.com/yinsel/XposedProjectTemplate](https://github.com/yinsel/XposedProjectTemplate)

完整 Xposed 代码实现如下（代码写的有点烂，勿喷）：

```
package com.example.xposed;  

import android.util.Log;  

import com.google.gson.JsonObject;  

import com.google.gson.JsonParser;  

import java.io.IOException;  

import java.lang.reflect.Constructor;  

import java.lang.reflect.InvocationTargetException;  

import java.lang.reflect.Method;  

import java.util.HashMap;  

import java.util.Map;  

import java.util.UUID;  

import de.robv.android.xposed.IXposedHookLoadPackage;  

import de.robv.android.xposed.callbacks.XC_LoadPackage;  

import fi.iki.elonen.NanoHTTPD;  

public class Hooker implements IXposedHookLoadPackage {  

    private HTTPServer httpServer = new HTTPServer(50000);  

    @Override  

    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {  

        try {  

            Class clazz = Class.forName("xxx.WbSM4Util$Companion", true, loadPackageParam.classLoader);  

            Method methods[] = clazz.getDeclaredMethods();  

            Constructor constructor = clazz.getDeclaredConstructor();  

            constructor.setAccessible(true);  

            Crypto crypto = new Crypto();  

            crypto.setClassLoader(loadPackageParam.classLoader);  

            crypto.setObject(constructor.newInstance());  

            for (int i = 0; i  {  

                try {  

                    httpServer.start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);  

                    Log("HTTP Server started on port " + httpServer.getListeningPort());  

                } catch (IOException e) {  

                    Log("Error starting HTTP Server: " + e.toString());  

                }  

            }).start();  

        }  

    }  

    public static class HTTPServer extends NanoHTTPD {  

        private Crypto crypto;  

        public HTTPServer(int port) {  

            super(port);  

        }  

        public Crypto getCrypto() {  

            return crypto;  
```
