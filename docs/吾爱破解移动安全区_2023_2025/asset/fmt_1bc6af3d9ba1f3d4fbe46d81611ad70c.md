

# [Android 原创] `DroidPlugin`项目技术分析与逆向工程指南

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2055507
- 抓取时间: 2026-11-01T13:59:56.141Z

## 一、项目概述

`DroidPlugin`是由360手机助手团队开发的Android插件化框架，于2015年首次发布，目前处于终止更新状态。核心目标是在免Root环境下实现APK文件的“免安装运行”，即无需经过系统安装流程即可直接加载并运行插件中的四大组件`Activity`、`Service`、`BroadcastReceiver`、`ContentProvider`。

### 核心技术特性

- 免Root的应用层Hook：通过动态代理技术拦截系统关键API，无需依赖Root权限即可实现插件组件加载，突破系统对组件注册的硬性限制。
- 零安装与系统目录零占用：插件APK无需安装到系统应用目录`/data/app`，可直接从存储加载，支持“独立安装应用”与“纯插件”两种运行模式。
- 完全隔离机制：插件间、插件与宿主间实现代码与资源的沙箱化隔离，避免组件冲突与数据干扰，保障运行独立性。

## 二、技术原理分析

`DroidPlugin`的核心实现围绕“逆向绕过系统组件管理机制”展开，通过“`Manifest`欺骗→`AMS`拦截→资源路由→插件`Service`前台化”的完整调用链，实现插件组件的非法加载与运行。以下为关键技术链路的代码与原理解析：

完整调用链解析：`Manifest`欺骗→`AMS`拦截→资源路由→插件`Service`前台化。下面把`DroidPlugin`实现「`Manifest`欺骗→`AMS`拦截→资源路由→插件`Service`前台化」的完整调用链、关键类和核心代码片段一次性梳理给你。所有行号以2016-12-20最后一次主干代码为准，类路径保持不变，可直接在仓库里`grep`验证。

### 1. `Manifest`欺骗（让未注册的组件合法通过`AMS`检查）

核心目标：绕过Android系统对组件必须在`Manifest`中注册的静态检查，为插件组件提供“合法身份”。

#### 1.1 宿主占坑

实现文件：`host/src/main/AndroidManifest.xml`

关键代码：

```xml

```

释义：宿主在`Manifest`中预先注册100个无实际业务逻辑的“占位组件”（`StubActivity`、`StubService`等），命名遵循“`C0-C99`”规则，满足系统对组件注册的静态校验要求，为后续插件组件替换提供“壳身份”。

#### 1.2 运行时偷梁换柱

Hook点：`IActivityManager`代理类`HookedActivityManagerHandler`

实现文件：`com/morgoo/droidplugin/hook/handle/HookedActivityManagerHandler.java`

关键代码：

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  

    if ("startActivity".equals(method.getName())) {  

        Intent raw = (Intent) args[2];  

        // 把插件真实Component替换为未占用的Stub组件  

        ComponentName stub = StubSelector.selectStubActivity(raw);  

        raw.setComponent(stub);            // 欺骗AMS，使其认为启动的是宿主注册的Stub组件  

    }  

    return method.invoke(mBase, args);  

}
```

释义：通过动态代理拦截系统`startActivity()`等方法，将插件的真实组件（如`com.example.plugin.MainActivity`）替换为宿主中未被占用的Stub组件（如`StubActivityC0`），使`AMS`（`ActivityManagerService`）误认为启动的是已注册的合法组件，绕过注册检查。

#### 1.3 `AMS`放行后换真身

Hook点：`ActivityThread.mH`的`LAUNCH_ACTIVITY`事件

实现文件：`HookedActivityThreadHandler.java`

关键代码：

```java
case LAUNCH_ACTIVITY:  
    ActivityClientRecord r = (ActivityClientRecord) msg.obj;  
    Intent intent = r.intent;  
    // 从Intent中提取插件真实组件，替换Stub组件  
    ComponentName plugin = IntentCompat.getSelector(intent);  
    r.intent.setComponent(plugin);  
    // 后续通过反射正常创建插件Activity实例  
    break;  
```

释义：在`AMS`放行组件启动请求后，通过Hook `ActivityThread`的消息处理机制，将Intent中的Stub组件换回插件真实组件，确保最终启动的是插件中的实际业务组件，完成“欺骗-还原”闭环。

### 2. `AMS`拦截（所有Binder入口都被动态代理替换）

核心目标：拦截系统服务（`AMS`、`PMS`）的`Binder`调用，伪造插件“已安装”的假象，实现组件生命周期管理。

#### 2.1 Hook时机

实现文件：`PluginHelper.java`

关键代码：

```java
@Override  

protected void attachBaseContext(Context base) {  

    PluginHelper.getInstance().applicationAttachBaseContext(base);  

    // 在宿主初始化时Hook系统服务  

    AMSHook.hookActivityManager(context);   // Hook AMS  

    PMSHook.hookPackageManager(context);   // Hook PMS  

    super.attachBaseContext(base);  

}
```

释义：在宿主`Application`初始化阶段，通过反射Hook `AMS`（`ActivityManagerService`）和`PMS`（`PackageManagerService`）的`Binder`接口，替换为框架自定义的代理对象，实现对系统服务调用的全程拦截。

#### 2.2 IActivityManager代理链

代理链路：`AMSHook`→`HookedActivityManagerHandler`

拦截方法：`startActivity()`、`startService()`、`stopService()`、`bindService()`、`unbindService()`、`getContentProvider()`、`registerReceiver()` 等。

关键逻辑：将插件的组件调用请求（如`startService(pluginServiceIntent)`）替换为宿主Stub组件的调用请求，使系统始终认为操作的是宿主内部组件，规避跨应用组件调用限制。

#### 2.3 IPackageManager代理链

代理链路：`PMSHook`→`HookedPackageManagerHandler`

拦截方法：`getPackageInfo()`、`queryIntentActivities()`、`getApplicationInfo()` 等。

关键逻辑：当系统查询插件包信息时，返回伪造的“已安装”数据（如插件包名、版本、权限等），使系统误认为插件已通过正常流程安装，从而允许其组件被调用。

### 3. 资源路由（插件独立`Resources`，与宿主隔离）

核心目标：实现插件与宿主的资源完全隔离，避免资源ID冲突（如`R.drawable.icon`重名导致的显示异常）。

#### 3.1 创建插件LoadedApk

实现文件：`PluginProcessManager.java`

关键代码：

```java
LoadedApk pluginApk = Reflector.with(host).field("mPackages")  
                 .call("get", pluginPkgName).get();  
// 为插件创建独立的AssetManager  
AssetManager am = AssetManager.class.newInstance();  
Reflector.with(am).method("addAssetPath", String.class)  
         .call(pluginApk.getResDir());  // 加载插件APK的资源路径  
// 基于独立AssetManager创建插件Resources  
Resources pluginRes = new Resources(am,  
        host.getResources().getDisplayMetrics(),  
        host.getResources().getConfiguration());  
```

释义：通过反射为每个插件创建独立的`AssetManager`和`Resources`实例，仅加载插件APK中的资源文件（`res`、`assets`目录），使插件资源与宿主资源完全隔离，避免因资源ID重复导致的冲突。

#### 3.2 Context注入

实现类：`PluginContext`（继承`ContextWrapper`）

关键重写方法：

```java
@Override  

public Resources getResources() {  

    return mPluginResources;  // 返回插件独立Resources  

}  

@Override  

public AssetManager getAssets() {  

    return mPluginResources.getAssets();  // 返回插件独立AssetManager  

}  

@Override  

public String getPackageName() {  

    return mPluginPkgName;  // 返回插件包名，而非宿主包名  

}
```

释义：通过自定义`PluginContext`覆盖资源获取方法，确保插件代码中调用`getResources()`、`getAssets()`等接口时，访问的是插件自身的资源，而非宿主资源，实现资源路由的彻底隔离。

#### 3.3 Activity启动时挂接

实现文件：`HookedInstrumentation.java`

关键代码：

```java
@Override  
public Resources getResources() {  
    return mPluginResources;  // 返回插件独立Resources  
}  
@Override  
public AssetManager getAssets() {  
    return mPluginResources.getAssets();  // 返回插件独立AssetManager  
}  
@Override  
public String getPackageName() {  
    return mPluginPkgName;  // 返回插件包名，而非宿主包名  
}
```

释义：在插件`Activity`实例化时，将其内部的`Context`（`mBase`）替换为插件专属的`PluginContext`，确保`Activity`生命周期中所有资源访问均指向插件自身，彻底切断与宿主资源的关联。

### 4. 插件`Service`前台化（防止被系统回收）

核心目标：通过复用宿主前台服务身份，提升插件`Service`的进程优先级，避免因系统内存不足被杀死。

#### 4.1 仍然用StubService占位

实现文件：`host/src/main/AndroidManifest.xml`

关键代码：

```xml

```

释义：宿主预注册的`StubService`在`Manifest`中声明为前台服务（`foregroundServiceType`），使其具备系统赋予的高优先级，为插件`Service`复用前台身份提供基础。

#### 4.2 StubService.onStartCommand()里二次转发

实现文件：`StubService.java`

关键代码：

```java

```

