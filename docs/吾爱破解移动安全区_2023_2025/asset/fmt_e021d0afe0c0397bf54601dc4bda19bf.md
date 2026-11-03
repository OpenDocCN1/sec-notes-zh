
# 示例代码

```java
private static final String BASE_URL = "[//server.dailiantong.com.cn/API/APPService.ashx" target="_blank" rel="noopener noreferrer nofollow"](https://server.dailiantong.com.cn/API/APPService.ashx:rel="nofollow noopener" href="https)";

public static void main(String[] args) {
    String timestamp = getCurrentTimestamp();
    String paramsValueStr = Stream.of(
                    "UserLogin", "UserTipForChangePass返回的LoginID值", "pass的加密值", "Android", "", "bdcba826-8ff0-4b5d-9428-7424a13c51fb",
                    "HUAWEI CAZ-AL10", "0", timestamp, "1.0", "Android", "DLTAndroid", "4.8.7", "huawei")
            .reduce("", String::concat);

    String fullStr = SIGN_KEY + paramsValueStr;
    String calculatedSign = getSign(fullStr);
    System.out.println("Sign: " + calculatedSign);

    try {
        String query = "?Action=UserLogin&LoginID=LoginID值&Pass=pss加密值&OS=Android&verifystr=&HD=bdcba826-8ff0-4b5d-9428-7424a13c51fb&PhoneType=HUAWEI%20CAZ-AL10&UserID=0&TimeStamp=" + timestamp + "&Ver=1.0&AppOS=Android&AppID=DLTAndroid&AppVer=4.8.7&ODM=huawei&Sign=" + calculatedSign;
        URL url = new URL(BASE_URL + query);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("GET");

        int responseCode = conn.getResponseCode();
        System.out.println("Response Code : " + responseCode);
        BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        String inputLine;
        StringBuilder response = new StringBuilder();
        while ((inputLine = in.readLine()) != null) {
            response.append(inputLine);
        }
        in.close();
        System.out.println("Response Body : " + response.toString());
    } catch (Exception e) {
        System.err.println("Error during HTTP request: " + e.getMessage());
    }
}

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

public static String getCurrentTimestamp() {
    Date currentTime = new Date();
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    formatter.setTimeZone(TimeZone.getTimeZone("GMT+8"));
    String formattedDate = formatter.format(currentTime);

    try {
        Date parsedDate = formatter.parse(formattedDate);
        String date = String.valueOf(parsedDate.getTime() / 1000);
        return date; // 转换为秒
    } catch (Exception e) {
        e.printStackTrace();
        return "0";
    }
}
```

逆向小白，大佬勿喷感谢
