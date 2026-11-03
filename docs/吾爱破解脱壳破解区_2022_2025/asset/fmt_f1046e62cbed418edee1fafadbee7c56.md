
# [Web逆向] 某校图书馆预约系统分析

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=2037247`
- 抓取时间: `2026-11-01T22:08:31`

## 某校图书馆预约系统分析

登录界面如图

![](https://oss.flt6.top/imgs/202506081117287.png?x-oss-process=style/init)

输入任意账号、密码，正确填验证码，F12抓包。注意到访问`http://[URL]/rest/auth?answer=bmx8&captchaId=qxtd5hl5h8wz`，返回

```
{"status":"fail","code":"13","message":"登录失败: 用户名或密码不正确","data":null}
```

且无其他访问，确定为登录API。

观察`url`参数，未发现账号密码；请求为`GET`不存在`data`。仔细观察请求体，注意到`headers`中有`username`和`password`，测试证明为账号密码。

![](https://oss.flt6.top/imgs/202506081117743.png?x-oss-process=style/init)

显然账号密码加密，尝试直接重放请求失败，注意到`headers`中的`x-hmac-request-key`等，推测存在加密。

### headers中的`X-hmac-request-key`等参数

全局搜索关键字`x-hmac-request-key`，找到唯一代码

```
{
    var t = "";
    t = "post" === e.method ? g("post") : g("get"),
    e.headers = c()({}, e.headers, {
        Authorization: sessionStorageProxy.getItem("token"),
        "X-request-id": t.id,
        "X-request-date": t.date,
        "X-hmac-request-key": t.requestKey
    })
}
```

动态调试找到`g`的实现

```
function g(e) {
    var t = function() {
        for (var e = [], t = 0; t 
```

注意到`o = p.a.decrypt(h.default.prototype.$NUMCODE)`；未知，确定`h.default.prototype.$NUMCODE`是常量，且`p.a.decrypt`与时间无关，直接动态调试拿到解密结果`o=leos3cr3t`.

其他算法实现均未引用其他代码，转写为`python`

```
def generate_uuid():
    hex_digits = '0123456789abcdef'
    e = [random.choice(hex_digits) for _ in range(36)]
    e[14] = '4'
    e[19] = hex_digits[(int(e[19], 16) & 0x3) | 0x8]
    for i in [8, 13, 18, 23]:
        e[i] = '-'
    return ''.join(e)

def g(e: str):
    uuid = generate_uuid()
    timestamp = int(time.time() * 1000)
    r = f"seat::{uuid}::{timestamp}::{e.upper()}"
    secret_key = b"leos3cr3t"
    hmac_obj = hmac.new(secret_key, r.encode('utf-8'), hashlib.sha256)
    request_key = hmac_obj.hexdigest()
    return {
        "id": uuid,
        "date": timestamp,
        "requestKey": request_key
    }
```

至此，使用上述`g`动态生成`hmac`后，其他内容不变条件下成功重放请求，可以拿到登录`token`。

注意到账号密码均加密，尝试逆向。

### 账号密码加密

从`http://[URL]/rest/auth?answer=bmx8&captchaId=qxtd5hl5h8wz`请求的启动器中找到关键函数`handleLogin`

![](https://oss.flt6.top/imgs/202506081117170.png?x-oss-process=style/init)

```
handleLogin: function() {
    var t = this;
    if ("" == this.form.username || "" == this.form.password)
        return this.$alert(this.$t("placeholder.accountInfo"), this.$t("common.tips"), {
            confirmButtonText: this.$t("common.confirm"),
            type: "warning",
            callback: function(t) {}
        });
    this.loginLoading = !0,
    Object(m.y)({
        username: d(this.form.username) + "_encrypt",
        password: d(this.form.password) + "_encrypt",
        answer: this.form.answer,
        captchaId: this.captchaId
    }).then(function(s) {
        "success" == s.data.status ? (sessionStorageProxy.setItem("loginType", "login"),
        sessionStorageProxy.setItem("token", s.data.data.token),
        t.loginLoading = !1,
        t.getUserInfoFunc()) : (t.$warning(s.data.message),
        t.refreshCode(),
        t.clearLoginForm(),
        t.loginLoading = !1)
    }).catch(function(s) {
        console.log("出错了:", s),
        t.$error(t.$t("common.networkError")),
        t.loginLoading = !1
    })
},
```

注意到

```
username: d(this.form.username) + "_encrypt",
password: d(this.form.password) + "_encrypt",
```

动态调试找到`d`的实现

```
var ....
    , l = "server_date_time"
    , u = "client_date_time"
    , d = function(t) {
    var s = arguments.length > 1 && void 0 !== arguments[1] ? arguments[1] : l
        , e = arguments.length > 2 && void 0 !== arguments[2] ? arguments[2] : u
        , a = r.a.enc.Utf8.parse(e)
        , i = r.a.enc.Utf8.parse(s)
        , n = r.a.enc.Utf8.parse(t);
    return r.a.AES.encrypt(n, i, {
        iv: a,
        mode: r.a.mode.CBC,
        padding: r.a.pad.Pkcs7
    }).toString()
}
```

显然`arguments.length`恒为`1`，故`s=l="server_date_time"`，`e=u="client_date_time"`，至此`AES`加密的参数均已知，`python`实现该`AES`加密后验证使用的`AES`为标准`AES`加密。

故`python`实现账号密码的加密：

```
def encrypt(t, s="server_date_time", e="client_date_time"):
    key = s.encode('utf-8')
    iv = e.encode('utf-8')
    data = t.encode('utf-8')
    cipher = AES.new(key, AES.MODE_CBC, iv)
    ct_bytes = cipher.encrypt(pad(data, AES.block_size))  # Pkcs7 padding
    ct_base64 = b64encode(ct_bytes).decode('utf-8')
    return ct_base64+"_encrypt"
```

关于验证码，`http://[URL]/auth/createCaptcha`返回验证码`base64`，提交时为`URL`参数，均未加密。

至此，完成整个登录过程的逆向。

### 其他API

所有其他API请求`headers`均带有`X-hmac-request-key`，逻辑同上。`URL`参数中的`token`为登录API返回值。

### 总结

该网站较简单，均为标准加密方法，`AES`密码和`iv`均为常数。`js`代码未加密未混淆难度小。

值得一提的是登录的用户名和密码通过`headers`传这是我第一次见。

### 完整demo

这是一个座位检查`demo`，查询某个区域是否有余座，如有则通过`serverchan_sdk`发生通知。

需按照实际情况修改常量。

为了识别验证码，调用了某验证码识别`API`，该平台为收费`API`，与本人无关，不对此负责，如有违规烦请告知。

Github账号出问题了只能直接贴代码了
