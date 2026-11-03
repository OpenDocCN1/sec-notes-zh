
# [Android 原创] Xposed模块注入资源原理以及思路

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2018443
- 抓取时间: 2026-11-01T16:05:07.862Z

在使用`xposed`进行hook宿主App的时候，有时候需要调用模块内的`Resources`。默认下访问的是宿主App的进程也就是宿主App的资源。

## 知识储备
`Lsposed`原理
`android`插件化技术

## 资源加载原理
在`android`的插件化技术中得知，`android`资源是通过`AssetManager`类的`addAssetPath()`函数去加载的。

![](https://attach.52pojie.cn/forum/202503/25/141836o2tgdgmyumutpmkn.png)

![](https://attach.52pojie.cn/forum/202503/25/141935u59dn9dns4dt6689.png)

在`loadAssets()`函数中通过资源管理器加载APK资源。

![](https://attach.52pojie.cn/forum/202503/25/142009pfe6w2y6baabt2zy.png)

并在`nativeSetApkAssets()`去解析`apk`中的资源。

## 注入思路
那么思路很简单我们主动调用`addAssetPath()`函数去吧模块的资源加载到宿主App。需要用到`xposed`的`IXposedHookZygoteInit`接口，重写`initZygote()`函数。

```
@Override 
public void initZygote(StartupParam startupParam) throws Throwable {  
    modulePath=startupParam.modulePath; 
}
```

这个函数用来获取模块`apk`路径(`/data/app/.....`)，并保存到`modulePath`。

在`handleLoadPackage()`函数里，通过去hook获取到宿主App的`Context`去主动调用`addAssetPath()`。

```
@Override  
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {  
    XposedHelpers.findAndHookMethod(Application.class, "attach", Context.class, new XC_MethodHook() {  
        @Override  
        protected void afterHookedMethod(MethodHookParam param) throws Throwable {  
            super.afterHookedMethod(param);  
            Context context = (Context) param.args[0];  
            ClassLoader classLoader = context.getClassLoader();  
            XposedHelpers.callMethod(context.getResources().getAssets(),"addAssetPath",modulePath);  
        }  
    });  
}
```

然后后面我们可以随意去使用我们的资源(`drawable`,`mipmap`,`values`,`layout`....)。注意使用的时候，调用的`context`是使用的宿主App的。

## 模块路径获取原理
`initZygote()`是怎么获取的模块路径

![](https://attach.52pojie.cn/forum/202503/25/142129zr0mddr900z0pjv4.png)

![](https://attach.52pojie.cn/forum/202503/25/142131wt75cqgqqv5gxcq5.png)

可以看到是通过`SELinuxHelper.getAppDataFileService`服务读取了`/data/data/de.robv.android.xposed.installer/conf/modules.list`路径的文件读取
