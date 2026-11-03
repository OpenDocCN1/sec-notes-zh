
# [Android 原创] android端手机虚拟机实现过程解析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2025427
- 抓取时间: 2026-11-01T15:52:35.925Z

## android端手机虚拟机实现过程

以下是`VirtualAPP`的执行流程图：

![](https://attach.52pojie.cn/forum/202504/18/190850f4g4jguglu7gwp6f.png)

我们通过这样的一个开源的`virtualApp`来了解一下，一个应用程序是怎么去实现在虚拟机里面运行的。

### Virtual space

首先是应用程序的下载安装。实际上，在虚拟机中运行的应用程序是下载到的虚拟机空间（`virtual space`）中的。

### Virtual Framework

虚拟机中的 App（比如 `App1`、`App2`）原本会直接调用系统 `Framework`，但是在虚拟机中的 `APP` 需要的就不同

- “以为”自己访问的是系统服务，
- 实际上却访问的是 `VA` 模拟的服务（`Hook` 之后的）。

#### Android Framework

`Android Framework` 是 `Android` 系统提供的一整套 `API`，它是 系统级服务（如 `AMS`、`PMS`、`LocationService`） 的接口，比如：

- `ActivityManagerService` (`AMS`)：管理 `Activity` 的启动与调度。
- `PackageManagerService` (`PMS`)：管理应用包的安装与信息。
- `LocationManager`：提供位置信息。
- `ClipboardManager`：剪贴板服务。

这些都属于 系统原生的 `Android Framework`，运行在系统进程（如 `system_server`）中，不是用户 `App` 能随意改动的。而在虚拟机中的 `Framework`，则是自开发的可改动的 `Framework`。

而在 `Virtual Framework` 中，需要做的事情就是让 `APP` 觉得自己是在原生系统下执行的，根据 `Android Frameword` 中的应用程序的执行流程，我们可以知道的是

#### 一个应用程序的执行流程大体是这样的：

- 点击桌面程序
- 将 `Activity` 顶上第一个设置为暂停状态，等待启动的应用
- 判断进程，先进程再程序
- 通过 `Zygote` 开始 `fork()` 应用程序进程了
- 执行 `ActivityThread.main()`
- `ActivityThread` 通过 `Binder` 将自身的 `ApplicationThread` 传给 `AMS`，方便 `AMS` 通过 `ActivityThread` 启动应用
- `AMS` 通过 `ApplicationThread.scheduleLaunchActivity()` 通知启动 `Activity`
- `ActivityThread` 通过反射加载目标 `Activity` 类
- 调用 `Instrumentation.callActivityOnCreate()`，执行程序的 `OnCreate()` 函数

应用的真正启动是 `ActivityThread` 执行的，但整个过程是由 `AMS` 控制调度的，双方通过 `Binder` 建立通信桥梁。

### 虚拟 App vs 原生安装 App

| 方面 | 虚拟机中的 App（VA App） | 系统安装的 App（原生 App） |
|---|---|---|
| 安装方式 | 并未真正通过系统 `PMS` 安装，仅在 `VA` 虚拟空间中注册 | 通过系统 `PMS` 安装，注册了 `Activity`、`Service` 等 |
| 运行路径 | 安装路径、数据路径等都是被 重定向/虚拟的（如 `/data/data/com.xxx` -> `/data/data/io.virtual.app/space/0/...`） | 使用 `Android` 系统真实的 `/data/data/com.xxx` 路径 |
| 访问系统服务 | 所有访问系统服务的调用（如 `Location`、`Clipboard`）都被 `VA Hook` 拦截和“伪造” | 调用系统原生 `ServiceManager` 提供的服务，直接连接 `system_server` |
| AMS / PMS 管理 | 由 `VA` 模拟的 `AMS`/`PMS` 进行调度（`Activity` 启动、`Service` 调用等） | 由系统的 `ActivityManagerService` 和 `PackageManagerService` 控制 |
| 权限模型 | 权限请求和判断被 `VA` 接管（可以伪造授予，也可以强制拦截） | 权限由 `Android` 系统控制，用户手动授权 |
| 多开与隔离能力 | 可以任意多开，同一 `App` 多个实例之间相互隔离 | 系统默认一个包名只允许一个实例，数据也无法隔离 |
| 文件/IO 控制 | `VA Hook` 底层文件系统访问，可实现只读/禁止/伪造/隔离等策略 | 系统文件访问直接作用于真实文件系统 |
| 运行时感知 | 可以“欺骗” `App`，使其以为自己运行在正常系统中（比如感知不到虚拟环境） | 真实环境，`App` 可以访问所有支持的设备资源 |
| 安全性控制 | 更灵活，可以模拟环境、劫持函数、限制行为 | 安全性高但开放度小，无法灵活控制运行环境 |

那么实际上在虚拟机下应用程序的执行流程也应该是这样的，但是不同的是，这里的 `Server` 服务不是 `Android` 原生下的 `framework`，而是虚拟机的 `framework`。

而这里就应用到的是 `APP HOOK` 的主要实现了

### APP HOOK

#### Hook 的目标是 系统服务类，主要分两种：

##### 1. Java 层服务（运行在 system_server）

`VA` 会通过 反射、代理（比如 `Binder` 代理）或动态注入，替换系统服务对象。

举个例子：

- `App` 调用 `Context.getSystemService("location")` 拿到 `LocationManager`。
- 正常系统下，它返回的是 `Binder` 连接到 `system_server` 的服务。
- `VA Hook` 后返回的是它自己实现的“假 `LocationManager`”，比如总是返回一个虚假的地理位置。

##### 2. Binder 接口 Hook

- `VA` 使用类似 “`Binder IPC` 劫持” 的方式，在 `Java` 层模拟 `AMS`、`PMS` 等 `Binder` 服务。
- 这就像劫持了一条“电话线”，`App` 以为在和系统通信，其实是和 `VA` 自己通信。

细节的说一下可能出现的具体实现：

- `App` 调用 `ActivityManager.startActivity()`
- `APP Hook` 中的 `AMS Hook` 通过代理劫持系统服务调用（比如修改 `IActivityManager` 的 `Binder`）
- 它并不真的去调用系统 `AMS`，而是把调用重定向到了 `VirtualApp` 自己实现的 `VA Server` → `AMS`
- `VA Server` 的 `AMS` 执行启动流程（比如在 `VirtualActivityStack` 中注册一个 `Activity` 记录）
- 最后，伪装成系统返回正常的 `ActivityResult`，`App` 觉得自己“真的启动了 `Activity`”

而在 `APP HOOK` 拦截完成之后，`VA Server` 就需要开始去提供对应的服务来实现拦截之后的服务请求。

所以：`APP Hook` 是用来“拦截”`App` 调用系统服务行为的；而 `VA Server` 是用来“虚拟提供”这些服务的实现端。

### Virtual Native

在这一层主要为了完成 `2` 个工作，`IO` 重定向和 `VA APP` 与 `Android` 系统交互的请求修改。

#### IO重定向

比如在一些 `APP` 中，会把一些配置、缓存、数据库等文件的路径写死在代码里

```
new File("/data/data/com.example.app/shared_prefs/config.xml")
fopen("/data/data/com.example.app/files/config.dat", "r");
```

这种情况下的，用虚拟机打开的程序就无法获取适配对应路径的文件和配置信息。

所以说出现了 `VA Native` 的存在的意义：

- 拦截所有 `open`, `fopen`, `access`, `stat` 等系统调用
- 判断路径是否是虚拟 `App` 的目标路径
- 替换为虚拟路径，到真实的程序的虚拟路径的位置

不过这里应该算得上是 `Native` 层的 `HOOK` 了。

#### JNI Native函数

以上的 `APP HOOK` 都是基于 `Java` 层进行的，但是相应的 `Android` 有大量的系统行为是通过 `Native` 实现的。这时候对应的应用程序适配就需要 `Native HOOK` 来实现

| 情况 | 举例 | 解决方法 |
|---|---|---|
| 直接访问文件 | `open`, `fopen`, `access` | `hook libc` |
| 直接调用系统服务 | `getuid()`, `getpid()` | `hook syscall` |
| 访问特殊设备 | `/dev/*`, `/proc/*` | 重定向路径 |
| ART Runtime 优化 | `libart.so` 调用 | `hook libart` |
| 动态链接时 resolve | `dlsym` 取函数地址 | `hook linker` |

这里举例的程序应用就是需要通过 `HOOK` `so` 层来实现分析和获取适配对应路径的文件和配置信息。

### 源码分析部分

#### mirror

在虚拟机中很多的细节其实是通过 `HOOK`，或者通过反射的操作调用实现的。比如拦截真实的 `AMS`,`PMS` 等服务，返回自写的各种服务。而在虚拟机框架中为了使得调用拦截等更加方便则出来对于反射等的封装。而在这里的封装就叫 `mirror`

正常执行下去拦截改定位:

```
private void hookInstrumentation() {
    try {
        // 加载 ActivityThread 类
        Class activityThreadClass = Class.forName("android.app.ActivityThread");
        // 获取 'currentActivityThread' 方法
        Method currentActivityThreadMethod = activityThreadClass.getDeclaredMethod("currentActivityThread");
        currentActivityThreadMethod.setAccessible(true);
        // 调用该方法获取当前的 ActivityThread 实例
        Object currentThread = currentActivityThreadMethod.invoke(null);
        // 获取 'mInstrumentation' 字段
        Field instrumentationField = activityThreadClass.getDeclaredField("mInstrumentation");
        instrumentationField.setAccessible(true);
        // 获取原始的 Instrumentation 实例
        Instrumentation originalInstrumentation = (Instrumentation) instrumentationField.get(currentThread);
        // 创建 InstrumentationDelegate 并替换原始的 Instrumentation
        InstrumentationDelegate instrumentationDelegate = new InstrumentationDelegate(originalInstrumentation);
        instrumentationField.set(currentThread, instrumentationDelegate);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

`mirror` 拦截改定位

```
private void hookInstrumentation(){
    Object currentThread ActivityThread.currentActivityThread.call();
    Instrumentation originInstrumentation         ActivityThread.mInstrumentation.get(currentThread);
    ActivityThread.mInstrumentation.set(currentThread,new InstrumentationDelegate(originInstrumentation));
}
```

这里实际上就去做了对应的函数封装，从而更加快速实现的代码逻辑

#### Java层 HOOK

举例应用调用 `AMS` 执行流程的过程来看看 `Java` 层 `HOOK`

`Android` 系统中，所有四大组件的管理都要通过系统的 `ActivityManagerService`（`AMS`） 完成，客户端通过 `IActivityManager` 接口 调用 `AMS`。

所以 `Hook AMS` = 控制所有组件的启动流程！得到 `IActivityManager` 接口就可以拦截执行流程

```
Activity.startActivity(Intent)
    ↓
Instrumentation.execStartActivity(...)
    ↓
ActivityManager.getService().startActivity(...)
```

这里会走 `ActivityManager.getService()` ，而在 `android8.0` 的位置

```
// frameworks/base/core/java/android/app/ActivityManager.java
public static IActivityManager getService() {
    return IActivityManagerSingleton.get();
}

// frameworks/base/core/java/android/util/Singleton.java
private static final Singleton IActivityManagerSingleton =
    new Singleton() {
        @Override
        protected IActivityManager create() {
            // 注意这行
            return ActivityManagerNative.getDefault();
        }
    };
```

`getService()` 会返回 `IActivityManagerSingleton.get()`；

所以在

```
public class ActivityManagerStub extends MethodInvocationProxy> {
    public ActivityManagerStub() {
        // 基于 mirror 获取 AMS 的 IActivityManager 接口对象，并创建其动态代理对象
        super(new MethodInvocationProxy<>(ActivityManagerNative.getDefault.call()));//这里通过反射去获取系统 IActivityManager 实例。
    }  
    public void inject() throws Throwable {
        if (BuildCompat.isOreo()) {
            // Android 8.x 及以上版本
            Object singleton = ActivityManager.IActivityManagerSingleton.get();
            // 替换原 AMS 对象为我们的动态代理对象
            Singleton.mInstance.set(singleton, getInvocationStub().getProxyInterface());
        } else {
            // Android 8.x 以下的实现（略）
        }
    }
}
```

用反射将其 `mInstance` 替换成我们刚才构造的 代理对象，应用对 `AMS` 的所有调用（比如 `startActivity()`），都会经过我们这个 `Hook` 代理对象。

```
// 方法 hook 实现
static class StartActivity extends MethodProxy {
    @Override
    public String getMethodName() {
        return "startActivity";
    }
    @Override
    public Object call(Object who, Method method, Object... args) throws Throwable {
        // 可以对参数进行处理
        int res = VActivityManager.get().startActivity(args);
        return res;
    }
}
```
