
# 1 代{过}{滤}理与抓包环境

## 1.1 启动 lamda 与配置 mitmproxy

```
#启动代{过}{滤}理
# sh /data/local/tmp/x86/bin/launch.sh

from lamda.client import *

d=Device("192.168.3.31")

profile = GproxyProfile()
profile.type = GproxyType.HTTP_CONNECT

# 此选项请根据实际情况选择你是否需要
profile.drop_udp = True
profile.host = "192.168.3.82"
profile.port = 8899
#d.install_ca_certificate("mitmproxy-ca-cert.cer")
d.start_gproxy(profile)
#d.stop_gproxy()
#d.uninstall_ca_certificate("cacert.der")

d.beep()
```

这个脚本在手机端 lamda 启动后运行一次，听到 beep 声表示运行成功。

`install_ca_certificate` 就是安装证书的命令，只需要运行一次。

手机端的配置结束，接下来 `pip` 安装 `mitmproxy`，使用 upstram 模式转发流量到上流代{过}{滤}理服务器（即 BP）即可。

命令为：

```
mitmdump --mode upstream:127.0.0.1:8080 --listen-port 8899 --ssl-insecure
```

上面的 `mitmweb` 也可以改成 `mitmproxy` 或 `mitmdump`，主要是中间流量的表现形式不同。

## 1.2 虚拟机、手机逃避代{过}{滤}理检测

有时候直接使用系统的代{过}{滤}理会被 APK 检测到。有些 APK 的包使用了参数 `NO_PROXY`，会绕过系统代{过}{滤}理发送，有些则会在检测到代{过}{滤}理的时候停止发包。

目前知道的能够绕过代{过}{滤}理的工具有两个，HTTPDroid 和 lamda。lamda 挂代{过}{滤}理参考上面的脚本，这个我测试过是可以绕过代{过}{滤}理检测的。HTTPDroid 及其他基于 iptables 的工具据称在原理上是可以绕过的。

但是 HTTPDroid 不能在夜神模拟器上安装。我的建议是普通模拟器上使用 lamda，真机模拟器上使用 HTTPDroid（因为 lamda 不能在真机模拟器上使用）或者真机宿主机安装小黄鸟（HTTPCanary）。

## 1.3 真机虚拟机逃避代{过}{滤}理检测（真机虚拟机）

有时候 APK 会校验用户所处的环境是否是真机，这个时候建议直接使用真机或在真机上安装虚拟机。这个原理跟反反爬虫是一样的：再多的掩饰都是虚的，程序员总能从你想不到的角度进行检测，所以最好直接在真机里做操作。比如安装真机虚拟机如 VMOS。

但是真机虚拟机是无法使用 lamda 的（实际的真机可以），用 VMOS 抓包的一个方式是代{过}{滤}理到宿主机的小黄鸟（HTTPCanary），导出 `.0` 证书，小黄鸟设置目标应用，虚拟机安装证书。

也可以通过转换过的 `.0` 证书 + 模拟器挂代{过}{滤}理代{过}{滤}理到 `mitmproxy` 然后中转到 `burpsuite`。一样的。

# 3 证书与双向认证

## 3.1 安装证书的小TIPS

由于虚拟机可能不能直接装证书，即使有 ROOT（具体为什么我不知道），只能将证书导出为 `.0` 格式然后复制到 `/system/etc/security/cacerts/` 下。

`cer`、`pem` 等格式的证书可以通过 `openssl` 转换为 `.0` 格式之后手动安装。

命令为（`pem` 与 `cer` 都是这个）：

```
openssl x509 -inform PEM -subject_hash_old -in mitmproxy-ca-cert.pem -out out.0
```

## 3.2 证书校验绕过

有时候，APK 会这样：

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681452845703-339f4650-59c9-4957-ba97-46a8c3a0b554.png#averageHue=%232e2d2b&clientId=u36f057d0-32bb-4&from=paste&height=144&id=u76e0f7f1&originHeight=144&originWidth=570&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11602&status=done&style=none&taskId=u00efd974-4552-43c3-9660-4562f85c6cd&title=&width=570)

校验通信中证书的签发 `HOST`。这种情况需要用户反编译找到 `hostname`，然后自行制作一个此 `hostname` 的证书。证书可以通过 `openssl` 生成。

## 3.3 双向认证（双签）绕过

双向认证，指的是 APK 与服务器各自有一个私钥，每次通信时，通信包会经过两次加密。或者换句话说，就是 APK 发送到服务器的包要被客户端证书签名，服务器发送到 APK 的包要被服务器证书签名（如果 APK 本身还设置了）。这就导致 BP 等使用的自签名证书不会被服务器端接受。

SSL 双向认证体系中，客户端与服务端各拥有一对证书和密钥，在通信过程中：

1. 客户端建立通信，发送 SSL 版本等信息给服务端。
2. 服务端发送服务端证书给客户端，客户端会校验服务端的证书是否在许可范围内。
3. 客户端校验通过后，发送客户端证书给服务端，服务端也会校验客户端证书是否在许可范围内。

综上所述，要成功抓到经过双向签名的包，需要同时绕过客户端和服务端双端的认证。客户端的认证可以通过 Xposed 模块 `SSLUnpinning` 或者 `JustTrustMe` 绕过，而服务端认证则需要获得 APK 文件中的证书。

反编译 APK，找到 APK 中保存的客户端证书，还有它的密码。公私钥一般作为单独文件存储（也有 APK 直接写死在代码里，不过证书是有过期时间的，所以应该没什么 APK 会这么做），密钥则会写在代码里，需要反编译查找。

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681454773819-cddea71f-7bbe-43a6-a09b-4948c5cfd433.png#averageHue=%234d463b&clientId=u36f057d0-32bb-4&from=paste&height=586&id=ubd4979ad&originHeight=586&originWidth=1302&originalType=binary&ratio=1&rotation=0&showTitle=false&size=446487&status=done&style=none&taskId=ubdcf869f-6496-4f1d-a7a6-05086500836&title=&width=1302)

如果代码未经混淆，直接搜索 `ClientKeyStore.load` 即可找到打开 `bks` 证书的语句，第二个参数即为密码，第一个参数为证书文件名。

由于 Android 证书端的证书格式为 `bks`，而 BP 需要的证书格式为 `p12`，所以我们还需要转换证书的格式。下载 `portecle` 后转换证书的格式，然后在 BP 中导入即可抓到包：

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681455793418-12595c5f-00b8-42f0-bb2d-b2220ff3840f.png#averageHue=%23f7f6f6&clientId=u36f057d0-32bb-4&from=paste&height=1047&id=u47f31b94&originHeight=1047&originWidth=1716&originalType=binary&ratio=1&rotation=0&showTitle=false&size=116686&status=done&style=none&taskId=u1820bc1f-ced5-4a1d-8ef8-c0aa5d33632&title=&width=1716)

# 4 使用 Drozer 扫描 Android 本地漏洞

`drozer` 主要用来查找 Android APK 本身的漏洞，而非对应后端的漏洞。

## 4.1 安装与启动

下载地址：[github.com/WithSecureLabs/drozer/releases](https://github.com/WithSecureLabs/drozer/releases)

需要先额外安装一个 python2.7，再运行安装程序。如果环境中已经有其他版本的 python 了，请先看下节。

在运行 `drozer.bat` 文件无报错后，我们开始配置手机端的 `agent` 和依赖。搜索 `python2` 文件夹下的 apk 文件，找到 `standard-agent.apk`，将其装在手机上，然后打开 APK，点击右下角的开启打开 `agent`。`adb` 方面转发端口，使用命令：

```
.\adb.exe forward tcp:31415 tcp:31415
```

然后电脑处使用命令：

```
.\drozer.bat console connect
```

运行成功应该如下：

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1680848640471-34656233-1283-47dc-a67c-e1e414bc745c.png#averageHue=%23092b5b&clientId=u46198781-2fd3-4&from=paste&height=383&id=ue7d3a54b&originHeight=383&originWidth=962&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12815&status=done&style=none&taskId=u9fbc21cd-f9e4-4260-a2de-75fd72ad11b&title=&width=962)

## 4.2 额外安装 py2 下的各种问题

现在基本没人会使用 py2 编程了，所以很多人会先装 py3。

在 4.1 安装 python2 时，如果想将 py2 不冲突地放进环境变量里，可以将 `py2` 文件夹里的 `python.exe` 更名为 `python2.exe`，然后将路径放在 `PATH` 下（不放在 `PATH` 也可以，后面的 `python2` 替换为绝对路径）。

点击安装，选择之前安装的 python2 文件夹，进入 `\Scripts` 文件夹下，将 `drozer.bat` 文件中的 `python` 替换为 `python2` 再运行即可启动 `dozer`。

然后我们开始配置依赖。移动到 `pip2.exe` 所在的文件夹，使用与平时使用 `pip` 相同的命令安装 `service_identity`、`twisted`、`pyopenssl`、`pyyaml`、`protobuf` 即可。

有时候下载的 py2 里没有 pip 文件，需要在这里获取 `get-pip.py`：[bootstrap.pypa.io/pip/2.7/get-pip.py](https://bootstrap.pypa.io/pip/2.7/get-pip.py)

全文复制后保存为 python 文件，然后使用 py2 运行就行了。但是这样获取的 pip 有个问题就是没有能访问的镜像源，所以每次安装时带上选项，使用国内的镜像源即可：

```
-i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
```

## 4.3 常见命令（dz控制台下）

查看 APK 列表：

```
run app.package.list
```

`-f` 选项可以按照关键词查找 APK。

查看 APK 暴露的 activity 列表：

```
run app.activity.info -a 包名
```

拉起 Activity：

```
run app.activity.start --component 包名 活动名（activity）
```

枚举 Provider：

```
run app.provider.info -a 包名
```

SQL 注入扫描：

```
run scanner.provider.injection -a 包名
```

# 5 插件框架

这两个框架的模块安装方式都是一样的：模块本身就是一个 APK，直接安装 APK 即会被识别。

## 5.1 安装使用 Magisk

电脑端模拟器安装 Magisk 可以参照此文章：[www.52pojie.cn/thread-1583586-1-1.html](https://www.52pojie.cn/thread-1583586-1-1.html)

使用 Magisk 而不使用 XPOSED 的原因是 Magisk 对文件系统的破坏性小，且不需要事先 ROOT，适用范围广一些。并且 Magisk 上可以集成 Xposed。

对于真机模拟器而言，已知光速模拟器可以安装 Magisk，但是一般情况下需要付费 VIP。VMOS 据说安装之后并不好用，可能还是要用其自带的 XPOSED。

可以使用 Magisk Manager 安装 Magisk。安装后，如果 Manager 提示更新但一直下载失败，请打开设置换源为 [qianyegroup.gitee.io/magiskbuilds/updates/custom_md2.json](https://qianyegroup.gitee.io/magiskbuilds/updates/custom_md2.json)。

## 5.2 安装使用 Xposed

这是真机模拟器的不得已之选。使用 VMOS PRO 的虚拟机会自带一个 XPOSED。

# 6 使用 Frida

由于 lamda 中自带了 frida，本节分为使用 lamda 与不使用 lamda 两种方式讲解（可惜不能兼容真机模拟器，不然是想默认使用 lamda 讲解的）。

## 6.1 安装

Frida 与 Drozer 一样，也是基于 python 的分布式框架。分为服务端与客户端两个。客户端（本机）使用 `pip` 安装：

```
pip install frida
pip install frida-tools
pip install objection
```

服务端（安装过 lamda 的手机不需要安装，也就是可以跳过下面的内容直接到使用部分）访问 github 页面选择下载：[github.com/frida/frida/releases](https://github.com/frida/frida/releases)

服务端和客户端的版本要一致，由于 `pip` 安装时会自动选择版本，我们这里以客户端版本为准。

使用

```
pip list | findstr frida
```

查看 frida 的版本，`adb` 命令行中使用

```
getprop ro.product.cpu.abi
```

获取架构，然后选择对应的 `frida-server`，如：

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681181312785-e573bef0-e830-4232-8329-a55ad50f4f31.png#averageHue=%23131313&clientId=u1f85cc59-315a-4&from=paste&height=88&id=u318807c6&originHeight=88&originWidth=571&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2468&status=done&style=none&taskId=u603223a2-d995-4ce8-a344-5906c5f4963&title=&width=571)

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681181403770-599dd803-cc6e-4a75-95f3-25c94d48a39d.png#averageHue=%23161616&clientId=u1f85cc59-315a-4&from=paste&height=67&id=u3b199f52&originHeight=67&originWidth=347&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1495&status=done&style=none&taskId=u6b06ea03-f65e-4461-b780-07455c8834d&title=&width=347)

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681181432260-4a8a46b4-41d3-4a5b-87e4-0158b478548f.png#averageHue=%230f151f&clientId=u1f85cc59-315a-4&from=paste&height=252&id=uab47d796&originHeight=252&originWidth=913&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35798&status=done&style=none&taskId=u8df66be6-e409-4a3a-a0e3-527b5c87156&title=&width=913)

解压文件，`adb push` 到 `/data/local/tmp`，对 `frida-server-xxx` 文件 `chmod 777` 然后执行即可。

### 6.1.1 lamda 使用 frida 的适配

以当前最新的 lamda 版本（3.150）为例。首先，需要运行命令：

```
pip3 install -U --force-reinstall lamda[full]
```

然后，确定 `frida` 版本为 `15.X`，`frida-tools` 为 `12.1.X`。

否则将会出现 `frida` 命令执行反复横跳，一会成功一会提示版本不匹配的问题。

## 6.2 基础使用

使用 lamda 中的 frida 与普通情况相似，但是需要在每一条 `frida` 命令下添加参数：`-H 192.168.0.2:65000`（ip 请改成手机 ip），而原生 frida 需要让 `adb` 转发 frida 端口出来：

```
adb forward tcp:27043 tcp:27043
adb forward tcp:27042 tcp:27042
```

在已经连接 `adb` 的情况下，`cmd` 使用：

```
frida-ps -U
```

来进行一个热身。这个命令显示的是手机进程列表。

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681192280521-6c8ea554-b3d6-4b39-a654-179a81949475.png#averageHue=%23151515&clientId=u1f85cc59-315a-4&from=paste&height=439&id=u67e7fa9d&originHeight=439&originWidth=693&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15054&status=done&style=none&taskId=u99d3f937-bb36-4992-854a-61816f273bb&title=&width=693)

使用命令（如果程序太多，考虑使用 `frida-ps -Ua`，它只会显示运行中 APK）

```
frida-ps -Uai
```

获取当前运行所有 APK 的包名：

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681192482019-10cd67cb-f1ce-4df3-8d4a-4dad8b63d83b.png#averageHue=%231a1919&clientId=u1f85cc59-315a-4&from=paste&height=247&id=Sdjrs&originHeight=247&originWidth=439&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7932&status=done&style=none&taskId=u3019fca6-bd3e-46ae-83d0-568d7b195e0&title=&width=439)

进入 `frida-shell`：

```
frida -H 192.168.3.61:65000 -f com.busuu.android.enc
```

![](https://cdn.nlark.com/yuque/0/2023/png/32358243/1681195242020-c4929eae-97c5-446c-812a-eb5cc87a842a.png#averageHue=%23141414&clientId=u1f85cc59-315a-4&from=paste&height=231&id=v41Ok&originHeight=231&originWidth=752&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7581&status=done&style=none&taskId=u906b3fd3-de0f-41d4-b333-2dee493a5a5&title=&width=752)

### 6.2.1 lamda 中 frida 的另一种使用方式

即在 python 脚本中进行。我给一个示例代码：
