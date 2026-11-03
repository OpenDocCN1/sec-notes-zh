
# [原创] Java 应用安全之 JEB Floating License 绕过

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1920103`
- 抓取时间: 2026-11-01T22:29:50

最近一朋友单位采购了 JEB Pro 用于 Android 逆向，但使用的是 Floating License，因此只能在公司内网中使用。这样一来朋友在节假日就没法卷了，于是找到了我看有没有兴趣研究一下。虽然笔者之前搞过一段时间 Java 逆向，但那主要针对 Android 应用，对于 PC 应用那是大姑娘坐花轿 —— 头一回。本着学习新知识的心态，就接下了这个任务。

## 前言

以防有朋友不了解，[https://www.pnfsoftware.com/jeb/](https://www.pnfsoftware.com/jeb/) 是一个逆向工程工具，主要用于 Android APK 的逆向分析，针对 smali 字节码反编译的功能类似于 JADX，但是也支持对 SO 等二进制程序的逆向分析。

一般来说 JEB Pro 采用订阅机制，根据使用的机器进行收费，一机一密。但对于企业而言通常采用浮动授权，即 Floating License，允许一个或者多个不固定的机器同时使用。[https://www.pnfsoftware.com/jeb/manual/floating/](https://www.pnfsoftware.com/jeb/manual/floating/) 是官方用于运行在服务端的私有浮动授权服务器，其本质上是一个 HTTP 服务器，运行后直接访问会显示授权信息。

![](https://attach.52pojie.cn/forum/202405/03/141518ectca8h7h0rzf5hg.png)

jeb.png (137.59 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MzY0OHw3MjM1NWM1Y3wxNzkzNTcyMTg4fDE2ODA4NnwxOTIwMTAz&nothumb=yes)

2024-5-3 14:15 上传

客户端则需要在 `jeb-client.cfg` 中配置 `.ControllerInterface` 和 `.ControllerPort` 等信息，或者在启动 JEB Pro 的时候填入服务端地址，从而实现授权。

## 逆向分析

对于静态分析而言，和 Android 应用中的静态分析流程基本一致，都是拖到 JADX 中然后搜索某些关键字，比如授权失败的提示信息，或者配置的信息。

首先我们将授权服务的地址指定为 `127.0.0.1:23477`，然后用 `NC` 监听该地址，启动后很快收到了请求：

```
Listening on 0.0.0.0 23477
Connection received on localhost 52330
POST /probe HTTP/1.1
User-Agent: PNF Software UP
Content-Type: application/x-www-form-urlencoded
Content-Length: 189
Host: 127.0.0.1:23477
Connection: Keep-Alive
Accept-Encoding: gzip

data=5400000035E5...
```

可见 JEB Pro 和浮动授权服务器是通过 HTTP 请求进行通讯的。

```
$ zipgrep /probe bin/app/jeb.jar
grep: (standard input): binary file matches
```

直接搜索请求的路径 `/probe`，可以定位到下面的代码：

```
/* loaded from: jeb.jar:com/pnfsoftware/jebglobal/ga.class */
public class ga {
    private static String nz = cns.nz("1150...");
    private static String Fj = cns.nz(new byte[]{117, 90, 69, 74, 69}, 2, 120);
    private Net jU;
    private String Fx;
    private int tQ;
    private String Fh;

    public ga(Net net, String str, int i, int i2) {
        if (net == null) {
            throw new IllegalArgumentException();
        }
        if (str == null || str.isEmpty()) {
            throw new IllegalArgumentException("Controller Interface is not set");
        }
        if (i = 65535) {
            throw new IllegalArgumentException("Illegal Controller Port value must be between 0 and 65535");
        }
        this.jU = net;
        this.Fx = str;
        this.tQ = i;
        Object[] objArr = new Object[3];
        objArr[0] = i2 == 1 ? "https" : NetProxyInfo.TYPE_HTTP;
        objArr[1] = str;
        objArr[2] = Integer.valueOf(i);
        this.Fh = Strings.ff("%s://%s:%d/probe", objArr);
    }
    // ...
    public String nz(String str) {
        return JebNet.post(this.jU, this.Fh, str);
    }

    public void Fj(String str) {
        JebNet.post(this.jU, this.Fh, str);
    }
}
```

看起来像是通过 `JebNet.post` 发送请求的。接下来有两个策略，一是继续静态分析不断查看交叉应用来来定位请求发送点，二是使用动态分析来验证该处是否确实被调用。懒人首选必然是后者。

### 动态分析

说起动态分析就有点怀念移动端常用的 `frida` 了。针对 Android 应用可以通过 hook 某些特定函数去检查参数、调用栈，并且对参数和返回值进行修改。理论上 `frida` 在 PC 端也是可以使用的，但是只支持了一部分版本的 OpenJDK，实测笔者的 JDK 17 支持并不是很好。

由于前一段时间研究过 Java Web 应用，那时用得最多的工具要数 [`arthas`](https://github.com/alibaba/arthas) 了。这是一个阿里巴巴开源的线上 Java 诊断利器，基于 JVMTI 注入应用，避免了调试器中断的繁琐流程。该工具使用 [ASM](https://asm.ow2.io/) 来动态修改 Java 字节码，不同于 `frida` 基于二进制的注入，ASM 对于 JDK 而言有很高的兼容性。

由于 `arthas` 使用 OGNL 作为表达式，因此对于 hook 的过滤和修改也具备很强的灵活性。关于其具体的使用方法可以参考官方文档，这里就不再赘述了。

直接使用 `as.sh` 指定 JEB 的进程 ID，然后 hook `JebNet.post` 方法，查看调用堆栈：

```
[arthas@70354]$ stack com.pnfsoftware.jeb.client.JebNet post params.length==3

Press Q or Ctrl+C to abort.

Affect(class count: 1 , method count: 2) cost in 53 ms, listenerId: 4

ts=2024-04-02 13:21:06;thread_name=Thread-9;id=1a;is_daemon=true;priority=5;TCCL=jdk.internal.loader.ClassLoaders$AppClassLoader@531d72ca

    @com.pnfsoftware.jeb.client.JebNet.post()
        at com.pnfsoftware.jebglobal.ga.nz(SourceFile:71)
        at com.pnfsoftware.jebglobal.aW.jU(SourceFile:194)
        at com.pnfsoftware.jebglobal.aW.run(SourceFile:81)
        at java.lang.Thread.run(Thread.java:833)
```

果然是这里，核心的 `jU` 方法如下：

```
/* loaded from: jeb.jar:com/pnfsoftware/jebglobal/aW.class */
public class aW implements Runnable {
    private long jU() {
        Fg Fj2;
        try {
            ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
            LEDataOutputStream lEDataOutputStream = new LEDataOutputStream(byteArrayOutputStream);
            lEDataOutputStream.writeInt(0);
            lEDataOutputStream.writeLong(this.sM);
            lEDataOutputStream.writeLong(this.Pm);
            lEDataOutputStream.close();
            String nz2 = this.Fh.nz(Fg.nz(Fg.nz(te.Fj(), byteArrayOutputStream.toByteArray(), new int[1])));
            if (nz2 == null || (Fj2 = Fg.Fj(Fg.nz(nz2.trim()))) == null) {
                return -1L;
            }
            this.fR = Fj2.tQ();
            LEDataInputStream lEDataInputStream = new LEDataInputStream(new ByteArrayInputStream(Fj2.dv()));
            if (lEDataInputStream.readInt() != 1) {
                return -1L;
            }
            return lEDataInputStream.readLong();
        } catch (Exception e) {
            nz.catching(e);
            return -1L;
        }
    }
    // ...
}
```

`aW` 这个类本身是个 `Runnable`，是使用一个额外的线程进行启动的，这里请求被调用的地方如下：

```
[https://www.52pojie.cn/home.php?mod=space&uid=1892347](https://www.52pojie.cn/home.php?mod=space&uid=1892347) // java.lang.Runnable

public void run() {
    int i = 0;
    int i2 = 0;
    boolean z = false;
    while (true) {
        long jU2 = jU();
        try {
            if (jU2 == 0 || jU2 == 2) {
                if (jU2 == 2 && !z) {
                    nz.warn(cns.nz(new byte[]{26, 54, 26, 7, 82, 106, 15, 7, 98, 67, 15, 5, 12, 11, 26, 84, 73, 26, 83, 79, 3, 8, 1, 23, 82, 84, 28, 9, 15, 78, 84, 28, 13, 69, 67, 12, 1, 26, 6, 29, 3, 0, 9, 23, 82, 89, 22, 26, 85, 65, 19, 23, 69, 67, 12, 1, 0, 11, 6, 23, 17, 1, 68, 84, 27, 78, 1, 116, 59, 79, 65, 23, 25, 6, 13, 68, 85, 27, 11, 29, 8, 21, 6, 23, 17, 1, 68, 66, 7, 13, 9, 23, 31, 6, 29, 94, 12, 89, 22, 26, 85, 83, 27, 7, 26, 25, 8, 68, 85, 5, 20, 5, 21, 17, 69, 89, 22, 26, 7, 82, 83, 28, 9, 18, 3, 22, 19, 23}, 1, 67), new Object[0]);
                    z = true;
                }
                if (i != 0 || i2 != 0) {
                    nz.warn(...);
                }
                i = 0;
                i2 = 0;
            } else {
                nz.warn(...);
                boolean z2 = false;
                if (jU2 = 3) {
                    z2 = true;
                } else {
                    z2 = true;
                }
                if (z2) {
                    //...
                    nz.info("trycnt=%d", Integer.valueOf(i3));
                    AbstractContext.terminate();
                }
            }
            Thread.sleep(10000L);
        } catch (InterruptedException unused2) {
            return;
        }
    }
}
```

简单来说：

- 这个线程会一直循环执行，且每隔 10 秒执行一次；
- 当 `jU` 的返回结果不为 0 时，且到达了最大的重试次数后，会发送失败事件，并关闭 JEB；
