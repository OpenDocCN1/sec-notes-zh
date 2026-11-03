
# [Android 原创] 某克浏览器小说阅读去广告初步

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2046845
- 抓取时间: 2026-11-01T15:42:52.261Z

注意：本文仅用于技术学习与研究目的，旨在分享 Android 逆向分析思路与实践。修改第三方应用程序的行为可能违反其用户服务条款。侵删

## 前言

由于在使用该浏览器阅读小说的过程被其不断弹出的广告所困扰，所以决定尝试改善使用该浏览器看小说的体验。主要针对的是底部的横幅广告和翻页过程中出现的广告。

## 定位

使用开发者助手查看广告控件的属性以及其值

![](https://pic-one1.oss-cn-beijing.aliyuncs.com/typroa/202507200939307.png?x-oss-process=style/zzz)

然后使用 `Jadx` 查看 App 的源码，在资源文件中搜索这个控件的 `id`

![](https://pic-one1.oss-cn-beijing.aliyuncs.com/typroa/202507191252280.png?x-oss-process=style/zzz)

这里选择第一个名称看起来最符合广告的布局文件，查看它的引用，`R` 文件相关的不管，直接去看使用的类

![](https://pic-one1.oss-cn-beijing.aliyuncs.com/typroa/202507191256057.png?x-oss-process=style/zzz)

```
package com.noah.sdk.business.render.container;

import android.content.Context;
import android.view.View;
import android.view.ViewGroup;
import com.noah.api.DownloadApkInfo;
import com.noah.api.SdkRenderRequestInfo;
import com.noah.api.bean.TemplateStyleBean;
import com.noah.logger.util.RunLog;
import com.noah.sdk.business.render.g;
import com.noah.sdk.business.render.template.e;

/* compiled from: ProGuard */
/* loaded from: classes5.dex */
public class c extends b {
    private static final String TAG = "SdkInsideTemplateStyle";
    private TemplateStyleBean aMP;
    private boolean aMQ;
    private int aMd;

    public c(TemplateStyleBean templateStyleBean, boolean z) {
        this.aMQ = z;
        this.aMd = templateStyleBean.getTemplateId();
        this.aMP = templateStyleBean;
    }

    [@Override](https://www.52pojie.cn/home.php?mod=space&uid=1892347) // com.noah.sdk.business.render.container.b
    protected int T(Context context) {
        int i12 = this.aMd;
        String str = "noah_sdk_template_native_ad_layout";
        if (i12 != 1) {
            if (i12 == 3) {
                str = this.aMQ ? "noah_sdk_template_banner_apk_layout" : "noah_sdk_template_banner_ad_layout";
            } else if (i12 == 16) {
                str = "noah_sdk_template_mt_banner_ad_layout";
            } else if (i12 == 17) {
                str = "noah_sdk_template_three_combine_layout";
            } else if (i12 == 5) {
                str = "noah_sdk_template_native_app_info_ad_layout";
            } else if (i12 == 6) {
                str = "noah_sdk_template_banner_three_ad_layout";
            } else if (i12 == 9) {
                str = "noah_sdk_template_native_live_layout";
            } else if (i12 == 10) {
                str = "noah_sdk_template_native_bubble_layout";
            } else if (i12 == 11) {
                str = "noah_sdk_template_native_ad_tv1_layout";
            } else if (i12 == 12) {
                str = "noah_sdk_template_native_ad_tv2_layout";
            } else if (i12 == 13) {
                str = "noah_sdk_template_native_live_tv_layout";
            } else if (i12 == 14) {
                str = "noah_sdk_template_banner_live_layout";
            } else if (i12 == 15) {
                str = "noah_sdk_template_rect_shape";
            }
        }
        RunLog.d(TAG, "使用渲染模版 template id :".concat(str), new Object[0]);
        return g.fI(str);
    }

    @Override // com.noah.sdk.business.render.container.b
    public com.noah.sdk.business.render.a a(SdkRenderRequestInfo sdkRenderRequestInfo, DownloadApkInfo downloadApkInfo) {
        return new e(this, sdkRenderRequestInfo, downloadApkInfo);
    }

    @Override // com.noah.api.delegate.ISdkTemplateContainer
    public int getTemplateId() {
        return this.aMd;
    }

    @Override // com.noah.api.delegate.ISdkTemplateContainer
    public TemplateStyleBean getTemplateStyleBean() {
        return this.aMP;
    }

    @Override // com.noah.api.delegate.ISdkTemplateContainer
    public View getTemplateView(Context context) {
        return com.noah.sdk.business.render.e.xk().openLayoutInflater(context).inflate(T(context), (ViewGroup) null);
    }
}
```

这里有几个关键方法：

`protected int T(Context context)`，根据构造函数传入的模板 ID，从多个布局文件（也就是我们之前搜索到的）选择一个具体要使用的布局文件名，然后把布局文件名转成真正的资源 ID。

`public View getTemplateView(Context context)`，它调用了安卓的 `inflate` 方法，把上一步 `T(context)` 方法选择的那个 XML 布局文件，真正地转换成了一个可以显示在屏幕上的 `View` 对象。

我们这里可以 `hook` `getTemplateView` 让它返回空，这样就不会对广告进行渲染。

## Hook 广告渲染

```
/**

 * 目标: 在广告视图被创建的瞬间进行拦截，阻止其返回。

 * 策略: Hook com.noah.sdk.business.render.container.c 类的 getTemplateView 方法。

 */

console.log("

-  最终广告拦截脚本已加载...");

Java.perform(() => {

    console.log("

-  Java 环境就绪，准备 Hook 广告视图工厂...");

    try {

        const CLASS_TO_HOOK = "com.noah.sdk.business.render.container.c";

        const AdContainer = Java.use(CLASS_TO_HOOK);

        // Hook getTemplateView(Context context) 方法

        AdContainer.getTemplateView.overload('android.content.Context').implementation = function(context) {

            console.log(`[!!!] 成功拦截到广告视图创建调用: ${CLASS_TO_HOOK}.getTemplateView()`);

            // 我们可以通过调用 this.getTemplateId() 来获取当前准备加载的广告模板ID，用于调试

            const templateId = this.getTemplateId();

            console.log(`

-  准备加载的广告模板 ID 为: ${templateId}`);

            // === 核心拦截操作 ===

            // 策略一 (推荐): 直接返回 null，让广告视图创建失败。

            console.log("

-  已阻止广告视图创建，返回 null。");

            return null; 

        };

        console.log(`[+] 已成功部署对 ${CLASS_TO_HOOK}.getTemplateView 的 Hook。`);

    } catch (error) {

        console.error(`[!] 最终 Hook 失败: ${error.message}`);

        console.error("[!] 请确认类名和方法签名是否完全正确。");

    }

});
```

![](https://pic-one1.oss-cn-beijing.aliyuncs.com/typroa/202507191310022.png?x-oss-process=style/zzz)

`hook` 之后底部广告和广告页的广告都不在渲染，但是还有夸克自带的“开会员免广告”这个广告，同样使用开发者助手和 `Jadx` 进行定位即可（过程同上，这里就不多写了），直接给代码

```
/**
 * Frida 终极组合拦截脚本
 * 目标: 彻底去除夸克小说中的所有翻页广告（包括第三方SDK广告和App自有备用广告）。
 * 策略: 同时 Hook 两个关键入口点，形成双重保险。
 */

console.log("
-  终极组合广告拦截脚本已加载...");

Java.perform(() => {
    console.log("

-  Java 环境就绪，准备部署多点 Hook...");

    // --- 防线 1: 禁用第三方广告SDK (Noah SDK) 的顶层服务入口 ---
    try {
        const SdkService = Java.use("com.noah.sdk.business.render.container.c");

        SdkService.getTemplateView.overload('android.content.Context').implementation = function(reqInfo) {
            console.log("[!!!] 防线1: 已拦截第三方广告SDK入口 (getTemplateView)。返回 null。");
            return null;
        };
        console.log("[+] 防线1部署成功");
    } catch (error) {
        console.error(`[!] 防线1部署失败: ${error.message}`);
    }

    // --- 防线 2: 禁用App自身的广告页面渲染入口 ---
    try {
        const AdPageView = Java.use("com.uc.application.novel.ad.view.NovelMixedAdPageView");

        AdPageView.drawPageData.overload('com.uc.application.novel.reader.NovelPage').implementation = function(novelPage) {
            console.log("[!!!] 防线2: 已拦截App广告页面渲染入口 (drawPageData)。");
            // 直接返回，不执行任何绘制操作
            return;
        };
        console.log("[+] 防线2部署成功");
    } catch (error) {
        console.error(`[!] 防线2部署失败: ${error.message}`);
    }
});
```

## Xposed 模块开发

考虑到去广告需要长久 `hook`，所以这里将 `Frida` 脚本改写成一个 `Xposed` 模块。该 `Xposed` 模块我用的是 `kotlin` 语言编写，`hook` 的点是 `com.noah.sdk.business.render.container.c` 的上层引用中动态渲染广告的函数（均能实现相应效果）。

### 创建 Xposed 模块项目

使用 Android Studio 创建一个标准的 "Empty Views Activity" 项目，并进行如下配置：

- `settings.gradle.kts`：添加 `Xposed` 官方仓库地址。

```
dependencyResolutionManagement {

  repositories {

      google()

      mavenCentral()

      maven { url = uri("https://api.xposed.info/") }

  }

}
```

- `build.gradle.kts`：添加 `Xposed` API 依赖。

```
dependencies {
  compileOnly("de.robv.android.xposed:api:82")
  compileOnly("de.robv.android.xposed:api:82:sources")
}
```

- `AndroidManifest.xml`：声明这是一个 `Xposed` 模块。

```
```

### 编写 Hook 核心逻辑

我们将 `Frida` 的“双重防御”策略，用 `Xposed` API 的语法“翻译”过来。创建一个 `HookEntry.kt` 作为模块入口：
