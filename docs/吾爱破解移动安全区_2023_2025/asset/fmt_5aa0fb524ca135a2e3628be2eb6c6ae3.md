
# [Android 原创] 某练通登录`sign`算法逆向

- 原帖：`https://www.52pojie.cn/forum.php?mod=viewthread&tid=1915954`
- 抓取时间：2026-11-02T01:15:13

## 抓包与脱壳

我们先抓一下包，然后准备apk逆向。

![](https://attach.52pojie.cn/forum/202404/21/083006olryv92y3r8dzx1d.png)

`91fee738c95dcec9f53047d4ad45944.png`（65.71 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDc5NXw2Mjc3NDFmNHwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:30 上传

一开始没有想到有壳的问题，拖入`jadx`分析后看目录类很少，凭少有的经验应该是加壳了，然后我们查一下壳。

![](https://attach.52pojie.cn/forum/202404/21/083053gmhaffatlbavmbnm.jpg)

`7fc146687c870e80182486af505e05c.jpg`（350.88 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDc5NnwwY2JkNjI0MHwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:30 上传

果然，百度的壳，但是后来用`mt`分析时发现是伪百度，所以直接用黑盒进行脱壳，顺利脱掉。然后我们搜索`UserLogin`（其实最开始我搜的是`sign`，没有找到结果）。

![](https://attach.52pojie.cn/forum/202404/21/083238vql389qp0z3pjqqr.png)

`b3f173aa9f86bac765954e37c2eab60.png`（31.17 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDc5OHxjM2Y5YTE5MHwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:32 上传

## 定位登录函数与参数

这里我们右键点击查找用例，只有一个结果，直接跟进去，来到`login`函数的定义处。

![](https://attach.52pojie.cn/forum/202404/21/083322hu3sv5a8ofxof09r.png)

`b311c06816fcf90fdab374b7761fbca.png`（40.19 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDc5OXw2MzYxYmQxN3wxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:33 上传

这里其实我们就已经可以清晰地看到`sign`加密所需要的参数了，分别是`Action`、`LoginID`、`Pass`、`OS`、`verifystr`、`HD`、`PhoneType`，以及`addFixedParams(params)`所额外添加的参数。

根据抓包分析，`Action`是路由地址，这里是`UserLogin`；`LoginID`后来分析是请求`UserTipForChangePass`服务器返回的`LoginID`值；`Pass`是调用`encryptionPs`得到的加密值，这里是将用户密码以及请求`UserTipForChangePass`服务器返回的`LoginID`进行处理。

![](https://attach.52pojie.cn/forum/202404/21/083444wpod1dlog31dzpzd.png)

`b3f173aa9f86bac765954e37c2eab60.png`（31.17 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDgwMHw1NGNkZDMyMnwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:34 上传

## 实现密码加密

我们自己实现一下这个加密逻辑。

![](https://attach.52pojie.cn/forum/202404/21/083632bhq82pq58v3rkczq.png)

`7eecd3c01a7e84fd9b592e297b6f015.png`（117.3 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDgwMXw2Y2MwODE3OXwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:36 上传

加密后与我们抓包得到的`Pass`一致，算法正确，这里贴一下代码：

```java
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class EncryptionHelper {
    public static String encryptionPs(String ps, String id) {
        String md5Ps = getMD5(ps);
        try {
            String en_psg = URLEncoder.encode(md5Ps + id, "GB2312");
            String unescape4 = java.net.URLDecoder.decode(en_psg, "GB2312");
            return getMD5(unescape4);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return getMD5(md5Ps + id);
        }
    }

    public static String getMD5(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] messageDigest = md.digest(input.getBytes());
            return convertByteToHex(messageDigest);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        }
    }

    private static String convertByteToHex(byte[] data) {
        StringBuilder sb = new StringBuilder();
        for (byte b : data) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        String ps = ""; //用户密码
        String id = ""; // 请求UserTipForChangePass服务器返回的LoginID值
        String encryptedPs = encryptionPs(ps, id);
        System.out.println("加密后的密码为: " + encryptedPs);
    }
}
```

## 处理固定参数与时间戳

`OS`、`verifystr`这两个参数可以固定，`HD`是随机生成的`uuid`值，那么我们其实也可以固定。

![](https://attach.52pojie.cn/forum/202404/21/083702rl6bwn69ibf258ii.png)

`79c2f9e7bc2bc06a55235b67f45a7af.png`（9 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDgwMnxlYjJkZDA3MXwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:37 上传

最后一个就是`addFixedParams`，我们看看它往我们的参数中添加了哪些额外参数，右键跟进去。

![](https://attach.52pojie.cn/forum/202404/21/084045wxib4xinl9yir44l.png)

`f028e0e2f373ea60a9a4fe26dd635cc.png`（30.28 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDgwNXw5Y2FlNDA4NXwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:40 上传

其实也很清晰，这里面除了`timestamp`，剩下参数值都可以固定（`UserID`固定为零）。这里的`timestamp`从长短能看出，是对当前时间戳经过处理后得到的。这里直接给出实现后的时间戳处理代码：

```java
import java.util.Date;
import java.text.SimpleDateFormat;
import java.util.TimeZone;

public class TimeUtil {
    // 设定时间格式和时区
    private static final String FORMAT_DATE_TIME_STR = "yyyy-MM-dd HH:mm:ss";
    private static final TimeZone TIME_ZONE = TimeZone.getTimeZone("GMT+8");  // GMT+8 时区

    // 获取当前时间的时间戳
    public static long getCurrentTimestamp() {
        Date currentTime = new Date();
        SimpleDateFormat formatter = new SimpleDateFormat(FORMAT_DATE_TIME_STR);
        formatter.setTimeZone(TIME_ZONE); // 设置时区
        String formattedDate = formatter.format(currentTime);
        try {
            Date parsedDate = formatter.parse(formattedDate);
            return parsedDate.getTime() / 1000; // 转换为秒
        } catch (Exception e) {
            e.printStackTrace();
            return 0L; // 出现错误时返回0
        }
    }
}
```

## 获取密钥

还有一个密钥，这个很简单，一直点跳转到声明就跟进去了。

![](https://attach.52pojie.cn/forum/202404/21/084203s9dyd2z1jysjg6cz.png)

`55c729a4491075c16965abd85d2c4cd.png`（19.35 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDgwNnw3ZDA0Y2Y0M3wxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:42 上传

## 生成`sign`并测试

最后实现以下生成`sign`的代码进行测试：

```java
private static String getSign(String data) {
    try {
        MessageDigest md = MessageDigest.getInstance("MD5");
        byte[] hashInBytes = md.digest(data.getBytes(StandardCharsets.UTF_8));
        StringBuilder sb = new StringBuilder();
        for (byte b : hashInBytes) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    } catch (NoSuchAlgorithmException e) {
        System.err.println("MD5 Algorithm not found: " + e.getMessage());
        return null;
    }
}
```

![](https://attach.52pojie.cn/forum/202404/21/084300cw2ujz7n2he32iuz.png)

`12758148a648c5ea22f2309c9f0cac0.png`（153.59 KB，下载次数：0）

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MDgwN3wzYmE0ZGZlNHwxNzkzNTgyMTE0fDE2ODA4NnwxOTE1OTU0&nothumb=yes)

2024-4-21 08:43 上传

登录成功，`sign`算法正确，最后附上完整测试代码：

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.TimeZone;
import java.util.stream.Stream;

public class Demo {
    private static final String SIGN_KEY = "9c7b9399680658d308691f2acad58c0a";
```
