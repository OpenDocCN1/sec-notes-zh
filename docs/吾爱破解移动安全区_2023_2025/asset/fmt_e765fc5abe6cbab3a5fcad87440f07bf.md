
在这里去调用 `int res = VActivityManager.get().startActivity(args);` 此处直接将调用转发到 `VActivityManager` 中（`VirtualApp` 的虚拟 AMS）。

## 拦截 AMS 服务

“拦截 AMS 服务” 的核心目的，实际上就是为了拿到 `IActivityManager` 的实例，并用我们构造的代理对象去替换掉它。因为 Android 应用中的组件调度（包括启动 `Activity`、`Service`、广播等）全都需要通过这个接口来向系统发送请求。

```
ActivityManager.getService().startActivity(...)
ActivityManager.getService().broadcastIntent(...)
ActivityManager.getService().startService(...)
```

这些都是 `IActivityManager` 的方法。

## Native层 Hook

上面也已经说过了 Native层 HOOK 的原理，这里就执行来看看 IO 重定向，实现修改的文件读写路径的操作。

```
@SuppressLint("SdCardPath")
private void startIOUniformer() {
    ApplicationInfo info = mBoundApplication.appInfo;
    int userId = VUserHandle.myUserId();
    String wifiMacAddressFile = deviceInfo.getwifiFile(userId).getPath();
    // 重定向 WiFi 相关地址
    NativeEngine.redirectDirectory("/sys/class/net/wlan0/address", wifiMacAddressFile);
    NativeEngine.redirectDirectory("/sys/class/net/eth0/address", wifiMacAddressFile);
    NativeEngine.redirectDirectory("/sys/class/net/wifi/address", wifiMacAddressFile);
    // 重定向 data/data 路径
    NativeEngine.redirectDirectory("/data/data/" + info.packageName, info.dataDir);
    NativeEngine.redirectDirectory("/data/user/0/" + info.packageName, info.dataDir);
    if (Build.VERSION.SDK_INT > Build.VERSION_CODES.N) {
        // Android 7.0 以上
        NativeEngine.redirectDirectory("/data/user_de/0/" + info.packageName, info.dataDir);
        String libPath = VEnvironment.getAppLibDirectory(info.packageName).getAbsolutePath();
        String userLibPath = new File(
            VEnvironment.getUserSystemDirectory(userId),
            info.packageName + "/lib"
        ).getAbsolutePath();
        // 重定向 native lib 路径
        NativeEngine.redirectDirectory(userLibPath, libPath);
        NativeEngine.redirectDirectory("/data/data/" + info.packageName + "/lib/", libPath);
        NativeEngine.redirectDirectory("/data/user/0/" + info.packageName + "/lib/", libPath);
    }
    // 虚拟存储处理
    VirtualStorageManager vsManager = VirtualStorageManager.get();
    String vsPath = vsManager.getVirtualStorage(info.packageName, userId);
    boolean enable = vsManager.isVirtualStorageEnable(info.packageName, userId);
    if (enable && vsPath != null) {
        File vsDirectory = new File(vsPath);
        if (vsDirectory.exists() || vsDirectory.mkdirs()) {
            HashSet mountPoints = getMountPoints();
            for (String mountPoint : mountPoints) {
                NativeEngine.redirectDirectory(mountPoint, vsPath);
            }
        }
    }
    NativeEngine.enableIORedirect();
}
```

可以看到这里利用 `redirectDirectory` 函数，去重定向了对应可能读取的位置，替换成了虚拟机对应位置的内容。

```
//int faccessat(int dirfd,const char *pathname,int mode,int flags);
HOOK DEF(int,faccessat,int dirfd,const char *pathname,int mode,int flags){
    int res;
    const char *redirect_path relocate_path(pathname,&res);
    int ret syscall(_NR_faccessat,dirfd,redirect_path,mode,flags);
    FREE(redirect_path,pathname);
    return ret;
}
```

该宏会生成一个以 `new_` 为前缀的函数，例如：

```
int new_faccessat(...) { ... }
```

- 判断是否需要重定向（如 `/data/data/com.xxx` → `/virtual/data/com.xxx`）
- 如果是，则返回新的路径；否则返回原路径 `res` 用于记录是否发生了实际替换

举个例子：

`pathname = "/data/data/com.test/files/a.txt"`

如果这个 app 被 VA 管控了，那路径会被转向

`redirect_path = "/data/user/0/com.va.wrapper/files/a.txt"`

从这里就对于开头的那个图片进行了总体的大致讲解，但是其中虚拟机下运行应用的很多问题以及配置环境更多的是需要在实践运用上去体现。

## 参考资料

https://github.com/asLody/VirtualApp
https://juejin.cn/post/7028124957141893150#heading-11
https://blog.csdn.net/weixin_33816300/article/details/88772462
https://blog.csdn.net/ganyao939543405/article/details/76146760

文章PDF：

通过网盘分享的文件：Android虚拟机原理总.pdf

链接: https://pan.baidu.com/s/1lbpLG_SbsF21yzmJSJ6HCg 提取码: chen

--来自百度网盘超级会员v4的分享
