
# [Android 原创] OK影视pro的奖励算法分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2052174
- 抓取时间: 2026-11-01T15:41:48.860Z

## 前言

教程中的软件版本是最新的,从2.7.9开始到3.4.2版本算法没变

软件有签名校验,本期只对算法进行分析

校验自行研究

破解的是mpv播放和弹幕搜索功能

## 思路

这个软件看完广告之后会获得8小时的解锁功能时间,并且不需要登录账号

所以大胆猜测它生成了一个时间戳在本地(`当前时间+8小时`)

写了一个 `frida` 脚本,通过hook `FileOutputStream` 监控生成文件

## Hook 监控文件生成

```javascript
Java.perform(function () {
    var FileOutputStream = Java.use("java.io.FileOutputStream");

    FileOutputStream.$init.overload('java.io.File').implementation = function(file) {
        var filePath = file.getAbsolutePath();
        console.log(`[生成文件]: ${filePath}`);
        console.log("调用栈：\n" + 
                  Java.use("android.util.Log").getStackTraceString(
                      Java.use("java.lang.Exception").$new()));
        return this.$init(file);
    };

    FileOutputStream.$init.overload('java.lang.String').implementation = function(path) {
        console.log(`[生成文件]: ${path}`);
        console.log("调用栈：\n" + 
                  Java.use("android.util.Log").getStackTraceString(
                      Java.use("java.lang.Exception").$new()));
        return this.$init(path);
    };
});
```

### 执行日志

```text
生成文件: /data/user/0/com.layaboxhmhz.gamehmhz.okys/files/tr.tmp

调用栈：

java.lang.Exception

at java.io.FileOutputStream.(Native Method)

at OoOo0o0oO0o0O0oO.oOoOoOo0O0O0oO0o.OoOo0O0oOo0oO0Oo(SourceFile:7)

at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.oOoOoO0O0Oo0o0o0(Unknown Source:47) at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.OoOo0O0OoOo0O0oO(Unknown Source:698)

at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.OoOoOoO0O0O0O0O0(Unknown Source:14) at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.onVideoRewarded(Unknown Source:2410)

at com.hzhj.openads.HJAdsSdkReward$4.onVideoRewarded(Unknown Source:124) at com.windmill.sdk.reward.WMRewardAd.onVideoAdReward(SourceFile:29) at com.windmill.sdk.a.m$11.run(SourceFile:13)

at android.app.Activity.runOnUiThread(Activity.java:7463)

at com.windmill.sdk.a.m.adapterDidRewardAd(SourceFile:198)

at com.windmill.sdk.custom.WMCustomRewardAdapter.callVideoAdReward(SourceFile:35)

at com.windmill.kuaishou.KuaiShouRewardVideoAdapter$2.onRewardVerify(SourceFile:1) at com.kwad.components.ad.reward.e.h.onRewardVerify(SourceFile:2) at com.kwad.components.ad.reward.e.i.hA(SourceFile:75) at com.kwad.components.ad.reward.KSRewardVideoActivityProxy$7.doTask(SourceFile:7)

at com.kwad.sdk.utils.bh.run(SourceFile:1)

at com.kwad.sdk.utils.bx.runOnUiThread(SourceFile:11)

at com.kwad.components.ad.reward.KSRewardVideoActivityProxy.handleNotifyVerify(SourceFile:90)

at com.kwad.components.ad.reward.KSRewardVideoActivityProxy.notifyRewardVerify(SourceFile:101)

at com.kwad.components.ad.reward.KSRewardVideoActivityProxy.access$400(SourceFile:1) at com.kwad.components.ad.reward.KSRewardVideoActivityProxy$2.onRewardVerify(SourceFile:28)

at com.kwad.components.ad.reward.presenter.platdetail.a.a.notifyRewardVerify(SourceFile:5)

at com.kwad.components.ad.reward.presenter.platdetail.a.a.a(SourceFile:29)

at com.kwad.components.ad.reward.presenter.platdetail.a.a.a(SourceFile:13)

at com.kwad.components.ad.reward.presenter.platdetail.a.a$1.onMediaPlayProgress(SourceFile:18) at com.kwad.components.core.video.b.tj(SourceFile:31) at com.kwad.components.core.video.b.m(SourceFile:1) at com.kwad.components.core.video.b$3.doTask(SourceFile:3)

at com.kwad.sdk.utils.bh.run(SourceFile:1)

at android.os.Handler.handleCallback(Handler.java:942)

at android.os.Handler.dispatchMessage(Handler.java:99)

at android.os.Looper.loopOnce(Looper.java:210)

at android.os.Looper.loop(Looper.java:299)

at android.app.ActivityThread.main(ActivityThread.java:8123)

at java.lang.reflect.Method.invoke(Native Method)

at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:559)

at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:950)
```

```text
生成文件: /data/user/0/com.layaboxhmhz.gamehmhz.okys/files/ra.txt

调用栈：

java.lang.Exception

at java.io.FileOutputStream.(Native Method)

at OoOo0o0oO0o0O0oO.oOoOoOo0O0O0oO0o.OoOo0O0oOo0oO0Oo(SourceFile:7)

at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.oOoOoO0O0Oo0o0o0(Unknown Source:47) at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.OoOo0O0OoOo0O0oO(Unknown Source:698)

at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.OoOoOoO0O0O0O0O0(Unknown Source:14) at tv.danmaku.ijk.media.player.tbd.RewoardAd$1.onVideoRewarded(Unknown Source:2297)

at com.hzhj.openads.HJAdsSdkReward$4.onVideoRewarded(Unknown Source:124) at com.windmill.sdk.reward.WMRewardAd.onVideoAdReward(SourceFile:29) at com.windmill.sdk.a.m$11.run(SourceFile:13)

at android.app.Activity.runOnUiThread(Activity.java:7463)

at com.windmill.sdk.a.m.adapterDidRewardAd(SourceFile:198)

at com.windmill.sdk.custom.WMCustomRewardAdapter.callVideoAdReward(SourceFile:35)

at com.windmill.kuaishou.KuaiShouRewardVideoAdapter$2.onRewardVerify(SourceFile:1) at com.kwad.components.ad.reward.e.h.onRewardVerify(SourceFile:2) at com.kwad.components.ad.reward.e.i.hA(SourceFile:75) at com.kwad.components.ad.reward.KSRewardVideoActivityProxy$7.doTask(SourceFile:7)

at com.kwad.sdk.utils.bh.run(SourceFile:1)

at com.kwad.sdk.utils.bx.runOnUiThread(SourceFile:11)

at com.kwad.components.ad.reward.KSRewardVideoActivityProxy.handleNotifyVerify(SourceFile:90)

at com.kwad.components.ad.reward.KSRewardVideoActivityProxy.notifyRewardVerify(SourceFile:101)

at com.kwad.components.ad.reward.KSRewardVideoActivityProxy.access$400(SourceFile:1) at com.kwad.components.ad.reward.KSRewardVideoActivityProxy$2.onRewardVerify(SourceFile:28)

at com.kwad.components.ad.reward.presenter.platdetail.a.a.notifyRewardVerify(SourceFile:5)

at com.kwad.components.ad.reward.presenter.platdetail.a.a.a(SourceFile:29)

at com.kwad.components.ad.reward.presenter.platdetail.a.a.a(SourceFile:13)

at com.kwad.components.ad.reward.presenter.platdetail.a.a$1.onMediaPlayProgress(SourceFile:18) at com.kwad.components.core.video.b.tj(SourceFile:31) at com.kwad.components.core.video.b.m(SourceFile:1) at com.kwad.components.core.video.b$3.doTask(SourceFile:3)

at com.kwad.sdk.utils.bh.run(SourceFile:1)

at android.os.Handler.handleCallback(Handler.java:942)

at android.os.Handler.dispatchMessage(Handler.java:99)

at android.os.Looper.loopOnce(Looper.java:210)

at android.os.Looper.loop(Looper.java:299)

at android.app.ActivityThread.main(ActivityThread.java:8123)

at java.lang.reflect.Method.invoke(Native Method)

at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:559)

at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:950)
```

可以看到生成了两个文件：

- `[生成文件]: /data/user/0/com.layaboxhmhz.gamehmhz.okys/files/tr.tmp`
- `[生成文件]: /data/user/0/com.layaboxhmhz.gamehmhz.okys/files/ra.txt`

这两个都是再弹出领取奖励的时候生成的,其它大部分都是sdk文件

## 初步判断

由此初步判断这里面就是时间轴

`ra.txt` 的内容：`bac2f54569ee82e90a1e21d2cfc3311bd9`

`tr.tmp` 的内容：`d2818fda49b6c6360d7abc4cffda9902b59947ce3088b1907f0b3e2fbb0bdfeee61784a3cb05f550e2524fd602a5052f2c7080dac99555`

可以看到并不是以时间轴的形式显示的,是一个加密过的文本

## 生成文件方法分析

根据刚刚打印到的,生成文件的方法 `OoOo0o0oO0o0O0oO.oOoOoOo0O0O0oO0o.OoOo0O0oOo0oO0Oo` 跳转过去看看代码

```smali
.method public static OoOo0O0oOo0oO0Oo(Ljava/io/File;[B)Ljava/io/File;
.registers 4

.line 1
:try_start_0
new-instance v0, Ljava/io/FileOutputStream;
.line 3
invoke-static {p0}, LOoOo0o0oO0o0O0oO/oOoOoOo0O0O0oO0o;->oOoOo0O0Oo0o0OoO(Ljava/io/File;)Ljava/io/File;
.line 6
move-result-object v1
.line 7
invoke-direct {v0, v1}, Ljava/io/FileOutputStream;->(Ljava/io/File;)V
.line 10
invoke-virtual {v0, p1}, Ljava/io/FileOutputStream;->write([B)V
.line 13
invoke-virtual {v0}, Ljava/io/OutputStream;->flush()V
.line 16
invoke-virtual {v0}, Ljava/io/FileOutputStream;->close()V
:try_end_12
.catch Ljava/lang/Exception; {:try_start_0 .. :try_end_12} :catch_13
.line 19
return-object p0
.line 20
:catch_13
move-exception p1
.line 21
invoke-virtual {p1}, Ljava/lang/Throwable;->printStackTrace()V
.line 24
return-object p0
.end method
```

可以看到,这里的代码主要是写入文件,并不是加密时间戳算法的地方,所以上面的代码只能打印出生成文件的方法

并不能找到加密算法的位置,直接用mt管理器对软件所有的类进行插桩

```text
3> 14:37:19.649 OoOo0o0oO0o0O0oO.OoOoOo0O0o0oO0o0
->oOo0oOo0Oo0oO0Oo(0CBB.java:13)
```
