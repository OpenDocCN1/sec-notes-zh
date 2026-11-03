
```java
XposedHelpers.findAndHookMethod("com.vmos.pro.account.AccountHelper", classLoader, "getUserConf", new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        super.beforeHookedMethod(param);
    }
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);
    }
});
```

### 可以看到 `jadx` 为我们一键生成了有关的 Hook 代码，但是这样就行了吗？我可以告诉你，不行。别忘了，这是一个加壳的应用，即使它是一款免费的加壳

## 加壳应用 Hook

通过对网上公开资料的查询，发现即使应用加固也需要在运行时进行还原修复，使用 `jadx` 打开加固的 `apk` 文件，找到 `attachBaseContext`

![](https://attach.52pojie.cn/forum/202310/28/220003zxewnfxce6ahymd3.png)

```java
XposedHelpers.findAndHookMethod("com.stub.StubApp", param.classLoader, "attachBaseContext", android.content.Context.class, new XC_MethodHook() {

    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {

        super.afterHookedMethod(param);
        Context context = (Context) param.args[0];//获取运行时的Context
        ClassLoader classLoader = context.getClassLoader()//获取真正的ClassLoader
        //在此添加Hook VIP等操作，使用classLoader

    }

});
```

## 继续编写

### 1) 获取到正确的 `ClassLoader` 后，对 `getUserConf` 函数的返回值进行遍历

```java
XposedHelpers.findAndHookMethod("com.vmos.pro.account.AccountHelper", classLoader, "getUserConf", new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        Object UserBean = param.getResult();
        for (Field field : UserBean.getClass().getDeclaredFields()) {
            //设置可访问, 极其重要, 不然会崩溃
            field.setAccessible(true);
            //使用反射来获取运行时的数据
            var name = field.getName();
            var type = field.getType();
            var value = field.get(UserBean);
            Log.i("HookTag", name + ": " + value);
        }
        super.afterHookedMethod(param);
    }
});
```

### 2) Hook 完成后，能够发现 `nickName` 是正确的，能够对应上 UI 的显示

![](https://attach.52pojie.cn/forum/202310/28/220024tp0p7id5uqmyyu8k.png)

![](https://attach.52pojie.cn/forum/202310/28/220018gmv8repbzb54pf9n.png)

### 3) 接下来只需要对循环里的数据进行判断赋值，然后返回即可

```java
//修改名称，其他自行测试
for (Field field : UserBean.getClass().getDeclaredFields()) {
    ......
    if (name.equals("nickName")) {
        field.set(UserBean, "测试文字");
    } else if(......) {
        ......
    }
}
param.setResult(UserBean);//设置返回值，替换掉param.getResult()获取的
```

### 4) UI 显示和下载功能

![](https://attach.52pojie.cn/forum/202310/28/221331r6a2jvvoabtbwjmv.png)

![](https://attach.52pojie.cn/forum/202310/28/221413s8t1c141oqxhhytz.png)

## 插件下载破解

获取到 VIP 后，发现还有一个插件下载的逻辑没有效果

### 下载逻辑分析

1) 当点击 Root 或者 XPosed 的时候，会提示加载失败

![](https://attach.52pojie.cn/forum/202310/28/221442ansnqf2k22qs9k52.png)

2) 但是点击谷歌服务的时候却有效果，猜测是网络请求

3) 打开抓包工具，通过对两者的对比，发现是其中少了一些数据，所以才会加载失败

![](https://attach.52pojie.cn/forum/202310/28/221453niddz43ssdbdd3os.png)

![](https://attach.52pojie.cn/forum/202310/28/221508lvn4ca63e9aerzzi.png)

4) 在 `jadx` 中搜索 `getPluginUrl`，通过阅读此函数发现有 2 个匿名函数，`failure` 和 `success`

![](https://attach.52pojie.cn/forum/202310/28/221533hgflhzpwaqum7ohc.png)

![](https://attach.52pojie.cn/forum/202310/28/221543i8ky6ysyb4yyb883.png)

5) 使用 `jadx` 默认给我们的参数 Hook 不太行，这时候需要使用其他函数来获取 `vu`

```java
//vu.class 无法获取
//使用loadClass来获取，在参数中填写vu即可
Class vu = classLoader.loadClass("vu");
XposedHelpers.findAndHookMethod("com.vmos.pro.activities.main.fragments.PluginHelper$getPluginDownloadBean$2$1", classLoader, "success", vu, new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        Log.i("HookTag", "success: " + Arrays.toString(param.args));
        super.beforeHookedMethod(param);
    }
});
//或者使用XposedBridge.hookAllMethods
```

6) 通过对这两者的 Hook，当点击 Root 插件按钮时会进入 `success` 且参数为：

```
谷歌服务按钮：success: [CommonResult{code=0, msg='OK', data=RespPlugin{systemPluginResult=RespPluginInfo{pluginUrl='http://xxx/xxx/plugin/android71gp_plugin-64bit3.zip', pluginMd5='62c38ec6b509e546e9fe9566969f7c49', version=0}}}]

Root按钮：success: [CommonResult{code=0, msg='OK', data=RespPlugin{systemPluginResult=null}}]
```

7) 可以明显发现其中确实是少了一些数据，接下来只需要补齐下载链接即可，但是如何获取这个链接呢？

- 充值 VIP 获取其中的链接
- 扫描网站链接?或者找到一个函数获取?
- 猜测

8) 上面 2 点显然不是我能够解决的，2333，那就通过对链接的猜测吧，根据能够下载的谷歌服务链接来看，root 和 xposed 可能为：

- `http://xxx/xxx/plugin/android71root_plugin-64bit.zip`
- `http://xxx/xxx/plugin/android71xposed_plugin-64bit.zip`
