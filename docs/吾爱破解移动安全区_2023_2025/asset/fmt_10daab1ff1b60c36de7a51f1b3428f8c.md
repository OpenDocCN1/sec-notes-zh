
# 签到协议分析

## 加密实现

```java
public class test {
    private static PublicKey RSA() {
        String baseDir = System.getProperty("user.dir");
        byte[] bArr;
        Exception e;
        try {
            InputStream open = new FileInputStream(baseDir + "\\djc_rsa_public_key_new.der");
            try {
                bArr = new byte[open.available()];
                do {
                    try {
                    } catch (Exception e2) {
                        e = e2;
                        System.out.println("Got exception while is -> bytearr conversion: " + e);
                        return KeyFactory.getInstance("RSA").generatePublic(new X509EncodedKeySpec(bArr));
                    }
                } while (open.read(bArr) != -1);
            } catch (Exception e3) {
                e = e3;
                bArr = null;
            }
            return KeyFactory.getInstance("RSA").generatePublic(new X509EncodedKeySpec(bArr));
        } catch (Exception e4) {
            System.out.println("cuole");
            return null;
        }
    }

    public static byte[] AES(String str, String str2) throws Exception {
        if (str == null) {
            return null;
        }
        Cipher instance = Cipher.getInstance("AES/ECB/PKCS5Padding");
        instance.init(1, new SecretKeySpec(str2.getBytes(StandardCharsets.UTF_8), "AES"));
        return instance.doFinal(str.getBytes(StandardCharsets.UTF_8));
    }

    public static byte[] toRSA(byte[] bArr) throws Exception {
        Cipher instance = Cipher.getInstance("RSA/ECB/PKCS1Padding");
        instance.init(1, RSA());
        return instance.doFinal(bArr);
    }
    public static String bytesToHex(byte[] bytes) {
        StringBuilder buf = new StringBuilder(bytes.length * 2);
        for(byte b : bytes) { // 使用String的format方法进行转换
            buf.append(String.format("%02x", Integer.valueOf(b & 0xff)));
        }

        return buf.toString();
    }

    public static void main(){
         String time = String.valueOf(System.currentTimeMillis());
         System.out.println(bytesToHex3(toRSA(AES(openid + "+5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a+" + time + "+" + new Random().nextInt(1000),"se35d32s63r7m23m"))));
    }
}
```

![image (1).png](https://attach.52pojie.cn/forum/202403/07/102214ttx5ekyoxd8kqvp5.png)

`image (1).png` (11.39 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDE5MHwwYWE5OTBlM3wxNzkzNTY4NTY1fDE2ODA4NnwxODk3NjM2&nothumb=yes)

2024-3-7 10:22 上传

## 抓包验证

这时候我们得到了一条加密因为存在`RSA`了，我们需要发包验证才行，先对比一下长度对不对吧。

![image (2).png](https://attach.52pojie.cn/forum/202403/07/102223j7zwzr8rfck6rrrc.png)

`image (2).png` (16.21 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDE5MXw5MzMyNDE0N3wxNzkzNTY4NTY1fDE2ODA4NnwxODk3NjM2&nothumb=yes)

2024-3-7 10:22 上传

经过做个对比，长度是一样的，那我们直接复现一下签到协议吧，看看对不对

![image (3).png](https://attach.52pojie.cn/forum/202403/07/102240s9mq43k9ed11fmv9.png)

`image (3).png` (46.3 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDE5Mnw4MTZmZTBmNHwxNzkzNTY4NTY1fDE2ODA4NnwxODk3NjM2&nothumb=yes)

2024-3-7 10:22 上传

这个是签到包。

### 签到请求包

```http
:method: POST

:path: /ams/ame/amesvr?ameVersion=0.3&sServiceType=dj&iActivityId=11117&sServiceDepartment=djc&set_info=newterminals&&appSource=android&appVersion=148&ch=10010&sDeviceID=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a&osVersion=Android-29&p_tk=1713520954&sVersionName=v4.7.2.0

:authority: comm.ams.game.qq.com

:scheme: https

user-agent: TencentDaojucheng=v4.7.2.0&appSource=android&appVersion=148&ch=10010&sDeviceID=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a&firmwareVersion=10&phoneBrand=Xiaomi&phoneVersion=MI+8&displayMetrics=1080 * 2115&cpu=AArch64 Processor rev 13 (aarch64)&net=wifi&sVersionName=v4.7.2.0&plNo=133

charset: UTF-8

referer: https://daoju.qq.com/index.shtml

content-type: application/x-www-form-urlencoded

content-length: 910

accept-encoding: gzip

cookie: djc_appSource=android; djc_appVersion=148; acctype=qc; appid=1101958653; openid=8C571443167953BCC67294D143B97FDA; access_token=1348438E26324A8A76C19222FF57A062

djcRequestId=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a-1709772046351-874&appVersion=148&sign_version=1.0&ch=10010&iActivityId=11117&sDjcSign=0ff184ac0b2b6b7c0ee76f7daf9c4f6bbafd7d1610d58eee92add748bd1af18e644ba8d2766393017c928e0f1193d19b8c5d7d08c2a6071e229c2a62cd8b0434ec5b0d068b9f0cbe880fb74580408632eb2f28f5daec685aa37d7685fd5b2cfb42bea5615d86396c9f128096292b10e4ad5077caaa6bd1642d03ebb5f1e8f9e9b982ec96415d24b335a73f5ad09d4859fb6af45b404e71db7a4fbda9e0a49862192a35935cdbab75612eea8707dafb2f2380f3e2a5e67587f162b2bb3456ca00a63c4b12c47d7ad59914d3910d4f83ca6462d48a4030a66ac695255d5f6de1bc2077383ec6bbe00f52f7a881365977228658401a0f256417b04e7785d258b1ee&sDeviceID=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a&p_tk=1713520954&month=202403&osVersion=Android-29&iFlowId=96939&sVersionName=v4.7.2.0&sServiceDepartment=djc&sServiceType=dj&appSource=android&g_tk=1842395457
```

### 返回结果

```json
{
        "flowRet": {
                "iRet": "0",
                "sMsg": "MODULE OK",
                "iAlertSerial": "0",
                "sLogSerialNum": "AMS-DJ-0307084047-hYkHMd-11117-96939"
        },
        "modRet": {
                "msg": "\u7b7e\u5230\u6210\u529f",
                "sMsg": "\u7b7e\u5230\u6210\u529f",
                "action_id": "7932",
                "ret": "0",
                "iRet": "0"
        },
        "ret": "0",
        "msg": ""
}
```

![image (4).png](https://attach.52pojie.cn/forum/202403/07/102320vfsfe9n940ece6s0.png)

`image (4).png` (18.59 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4MDE5M3xjNWM3NjI1ZnwxNzkzNTY4NTY1fDE2ODA4NnwxODk3NjM2&nothumb=yes)

2024-3-7 10:23 上传

### POST Body

我们看这个包吧。先看`post body`。

```http
djcRequestId=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a-1709772046351-874

&appVersion=148

&sign_version=1.0

&ch=10010

&iActivityId=11117

&sDjcSign=0ff184ac0b2b6b7c0ee76f7daf9c4f6bbafd7d1610d58eee92add748bd1af18e644ba8d2766393017c928e0f1193d19b8c5d7d08c2a6071e229c2a62cd8b0434ec5b0d068b9f0cbe880fb74580408632eb2f28f5daec685aa37d7685fd5b2cfb42bea5615d86396c9f128096292b10e4ad5077caaa6bd1642d03ebb5f1e8f9e9b982ec96415d24b335a73f5ad09d4859fb6af45b404e71db7a4fbda9e0a49862192a35935cdbab75612eea8707dafb2f2380f3e2a5e67587f162b2bb3456ca00a63c4b12c47d7ad59914d3910d4f83ca6462d48a4030a66ac695255d5f6de1bc2077383ec6bbe00f52f7a881365977228658401a0f256417b04e7785d258b1ee

&sDeviceID=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a

&p_tk=1713520954

&month=202403

&osVersion=Android-29

&iFlowId=96939

&sVersionName=v4.7.2.0

&sServiceDepartment=djc

&sServiceType=dj

&appSource=android

&g_tk=1842395457
```

`djcRequestId`：这个我们不就发现这不就是`AES`加密的时候那个参数么。在外面拼接一下`Openid`在做一层`RSA`不就得到`sDjcSign`。

剩下的参数`p_tk`同样也是根据`openid`还有`access_token`得来的，我们要是想实现签到，那就先抓包，得到`openid`，`access_token`，`p_tk`。好像这几个有效期是三个月。再看`url`中的参数

### URL 参数

```text
/ams/ame/amesvr?ameVersion=0.3
&sServiceType=dj
&iActivityId=11117
&sServiceDepartment=djc
&set_info=newterminals
&&appSource=android
&appVersion=148
&ch=10010
&sDeviceID=5d73310d28a1a02929aea12667e64edb0dd356c592261249d47bf75e134efe2a
&osVersion=Android-29
&p_tk=1713520954
&sVersionName=v4.7.2.0
```

`url`中没有什么特别的参数，就是`p_tk`要跟`post body`中一致

## 签到代码

终于到了最后一步，签到代码

```java
public class test {

    private static PublicKey RSA() {

        String baseDir = System.getProperty("user.dir");
```
