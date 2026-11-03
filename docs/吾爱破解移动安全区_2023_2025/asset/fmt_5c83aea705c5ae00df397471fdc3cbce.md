
# [Android 原创] 有 云笔记 v7.5.512 以正向开发的角度进行逆向分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1969351
- 抓取时间: 2026-11-01T16:30:21.274Z

2024年10月1日

声明: 仅用于个人学习，不提供成品，如有侵权请私信联系删除

以正向开发的视角进行逆向，全文较长，按需学习，跟着步骤小白也可以一步一步分析下去

仅能对 app 的本地验证会员的功能进行破解，对需要在服务器验证会员并在服务器生成下发数据的操作无法破解

完成的内容有:

- 去除广告
- 去除设备登录限制
- 实现本地的 VIP 功能

## 使用工具

- [firda](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1823118&highlight=frida)
- [jadx](https://github.com/skylot/jadx/releases)
- [jeb](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1957634&highlight=jeb)
- [Reqable](https://reqable.com/zh-CN/)

## 分析过程

### 1. 查询是否加固

查询加固可以任选 [ApkCheckPack](https://github.com/moyuwa/ApkCheckPack)/[pkid](https://bbs.kanxue.com/thread-225120.htm)/MT管理器 进行查询，发现没有加固，直接使用 `jadx` 分析代码。

![](http://forumupload.ru/uploads/001c/43/d3/2/332293.jpg)

### 2. 解决开屏广告的两种方法

#### 方案① 模拟点击跳过

这是一个几乎通用的方法，只需要分析少量代码就可以做到。app 的广告页面要么是放在首个 activity，要么经由首个 activity 跳转，所以分析开屏广告一定要先看首页。而首页的是在 `AndroidManifest.xml` 中定义的

通过 `AndroidManifest.xml` 可以看到 `NormalSplashActivity` 就是首页，进去看：

```
package com.youdao.note.activity2;

import com.alibaba.android.arouter.facade.annotation.Route;
import com.youdao.note.lib_router.AppRouter;
import com.youdao.note.splash.SplashActivity;

/* compiled from: Proguard */
@Route(path = AppRouter.NORMAL_SPLASH_PATH)
/* loaded from: classes4.dex */
public final class NormalSplashActivity extends SplashActivity {
}
```

竟然是空的，那就看它的父类 `SplashActivity`

代码很长，需要分析里面的关键方法

思路讲解: 这里面的代码很复杂，有一些 VIP 判断，但是不知道有什么用，可以先暂时不管，完成目标——干掉开屏广告

```
package com.youdao.note.splash;

public class SplashActivity extends FragmentSafeActivity implements CancelAdapt, Runnable {
    @Override
    public void onCreate(Bundle bundle) {
        ...
        initView();
        showVideo();
        ...
    }

    public void onActivityResult(int i2, int i3, Intent intent) {
        super.onActivityResult(i2, i3, intent);
        if (i2 == 84 || i2 == 51) {
            launchMainIfReady();
        } else if (i2 == 112) {
            showSplash();
        }
    }

    public void afterSystemPermissionChecked(boolean z) {
        if (needDataTransferForAndroid10()) {
            startActivityForResult(new Intent(this, (Class) DataTransferActivity.class), 112);
        } else if (z) {
            showSplash();
        }
    }

    private void showSplash() {
        ...
        showSplashAd();
        ...
    }

    @Override
    public void run() {
        YNoteLog.d(TAG, "过了3秒还没拉取到广告，关闭页面");
        launchMainIfReady();
    }

    public synchronized void launchMainIfReady() {
        if (this.mPendingCount.get()
```

通过上面截取的部分代码可以知道，代码大部分的流程都是指向了一个方法 `launchMainIfReady()`，而这个方法就是点击跳过广告后执行的方法，所以我们可以在广告加载的过程中，或者直接在 `SplashActivity` 类生成的时候就调用这个方法实现跳过广告。以下是 `frida` 方法

```
function jumpAdSplash() {
    var targetClass = "com.youdao.note.splash.SplashActivity";
    Java.choose(targetClass, {
        onMatch: function (instance) {
            console.log("Found instance: " + instance);
            //获取到实例对象后执行跳过广告的方法，这个方式更简单直观，但是还是会有加载广告的前置行为
            instance.launchMainIfReady();
        },
        onComplete: function () {
        }
    });
}
```

⚠️注意: 不能直接调用 `finish()` 方法，会导致 app 闪退

#### 方案② 禁止广告生成

上面那个方法实现了在广告没有用户没有看到广告之前就把广告关闭了，但广告其实还是加载了（浪费了流量？），这一步的目标就是不让广告加载

在进行这一步之前先思考一下需要做什么才能禁止广告加载，不外乎是：禁止广告初始化、把广告内容置空、禁止拉取广告、获取会员……

有了上面的思路就可以开始尝试分析代码了。在上一个方案中，我们确定了广告就是在初始页面进行加载的，那么广告内容会是在哪里生成呢？根据加载广告的方式去看一下 `AdManager.INSTANCE.loadSplashAd()`，分析该类

```
package cn.flying.sdk.openadsdk.ad;

public final class AdManager implements AdvertManager {
    @Override // cn.flying.sdk.openadsdk.ad.AdvertManager
    public void loadSplashAd(LifecycleOwner lifecycleOwner, AdConfig adConfig, AdView adView, AdvertListener.SplashAdListener splashAdListener) {
        s.f(lifecycleOwner, "lifecycleOwner");
        verifyAdConfig(adConfig, adView);
        s.d(adConfig);
        s.d(adView);
        l.d(LifecycleOwnerKt.getLifecycleScope(lifecycleOwner), null, null, new AdManager$loadSplashAd$1(adConfig, new SplashAdvert(lifecycleOwner, adConfig, adView, splashAdListener), splashAdListener, null), 3, null);
    }

    public static final class Builder {
        ...
        public Builder(List list0, a a0, AdAppCallBack adAppCallBack0, boolean z, boolean z1) {
            s.f(list0, "networkInterceptors");
            s.f(a0, "userId");
            super();
            this.networkInterceptors = list0;
            this.userId = a0;
            this.adAppCallBack = adAppCallBack0;
            this.isTestingModeEnable = z;
            this.isYouDaoTestService = z1;
        }
        ...
    }
}
```

在该类中看到了一个 `Builder` 建造者模式，根据名称可以判断 ad 业务就是通过这个 `Builder` 方法来创建的，通过 `jadx` 查看该方法的用例

![](http://forumupload.ru/uploads/001c/43/d3/2/194461.jpg)

是在 `YNoteApplication` 中进行调用的，而 `YNoteApplication` 又是在 `AndroidManifest.xml` 中自定义的 `Application`，所以可以推断出，在 `Application` 中初始化广告业务，然后在需要的 activity 中调用这个单例 `AdManager` 来生成 ad 内容

经过尝试，返回 `AdManager.Builder` 为空或者 new 一个新对象的时候会闪退，所以不能通过禁止初始化的方法来完成

按逻辑来说可以在设置广告参数的时候让参数值不正确，实现找不到广告的目的。这个逻辑入口在 `AdManager.doInit()` 中，但我尝试了一番发现没有试出效果，所以选择在获取广告的时候找突破点

在分析 `AdManager` 类的时候有几个方法 `jadx` 无法很好地解析，需要用 `jeb` 来看，最关键的是 `loadAdvert()` 方法

`AdManager.java`

```
    private final void loadAdvert(AdConfig adConfig0, AdvertParser advertParser0, BaseAdListener advertListener$BaseAdListener0) {
        AdLogUtils.d("AdManager", "调用广告接口loadAdvert");
        AdManager.loadAdvert.callback.1 adManager$loadAdvert$callback$10 = new AdManager.loadAdvert.callback.1(advertListener$BaseAdListener0, advertParser0, adConfig0);
        if(adConfig0.getUseNewApi()) {
            AdManager.loadAdvert.1 adManager$loadAdvert$10 = new AdManager.loadAdvert.1(adConfig0, adManager$loadAdvert$callback$10, null);
            l.d(AdManager.managerScope, null, null, adManager$loadAdvert$10, 3, null);
            return;
        }
        AdvertClient.getInstance().get().getAdvert(adConfig0.getSpaceId(), adConfig0.getOperationType().getType()).enqueue(adManager$loadAdvert$callback$10);
    }
```

代码分析到这里可以判断这就是广告接入的入口了，按照正常的逻辑只要我们将这个方法给屏蔽掉就可以实现禁止拉取广告。但我一顿实操下来发现，通过 `jadx` 复制的 `frida` 代码片段并没有在加载广告的时候被执行，而我使用 [raptor_frida_android_trace.js](https://github.com/0xdea/frida-scripts/blob/master/raptor_frida_android_trace.js) 查看的时候确实是在广告开始前有调用这个方法的，暂时不清楚具体的原因

![](http://forumupload.ru/uploads/001c/43/d3/2/279045.jpg)

在 `loadAdvert()` 中继续分析代码，跳转到 `AdvertClient` 类查看，这个代码很明显就是网络接口地址，通过调用 `okhttp` 的网络接口实现网络请求

```
package cn.flying.sdk.openadsdk.api;

import cn.flying.sdk.openadsdk.bean.AdvertConfig;
import cn.flying.sdk.openadsdk.bean.AdvertListModel;
import cn.flying.sdk.openadsdk.http.ApiResponse;
import retrofit2.Call;
import retrofit2.http.GET;
import retrofit2.http.Query;

/* compiled from: Proguard */
/* loaded from: classes.dex */
public interface AdvertService {
    @GET("/api/open/advertOpen/getAdvertResources")
    Call> getAdvert(@Query("advertSpaceCode") String str, @Query("operationType") String str2);

    @GET("/fly-api/advert/config")
    Call> getAdvertConfig();

    @GET("/api/open/advertOpen/v2/getAdvertResources")
    Call> getAdvertV2(@Query("advertSpaceCode") String str, @Query("operationType") String str2, @Query("banAdvertIds") String str3);
}
```

使用 `Reqable` 查看 `getAdvertResources` 接口的请求数据

![](http://forumupload.ru/uploads/001c/43/d3/2/972737.jpg)

这时候骚操作就来了，因为广告业务是需要经过网络拉取的，如果我将这个拉取接口的地址改变掉的话，自然就拉取不了广告，直接向 `claude.ai` 询问 hook 所有网络请求的方法就可以实现了

> me: 在 Android 中 hook 所有网络请求的方法
>
> ai: 我们可以使用更底层的方法来 hook 所有的网络请求，而不仅限于 `OkHttp`。在 Android 中，几乎所有的网络请求最终都会通过 `java.net.URL` 或 `java.net.Socket` 来实现。让我为您展示一个更通用的解决方案。

```
function hookHttp() {
    // Hook URL类的openConnection方法
    var URL = Java.use("java.net.URL");
    URL.openConnection.overload().implementation = function() {
        var url = this.toString();
        if (shouldBlockUrl(url)) {
            var invalidURL = URL.$new("https://yuu.com");
            console.log("[URL] Redirecting to invalid domain", url);
            return invalidURL.openConnection();
```
