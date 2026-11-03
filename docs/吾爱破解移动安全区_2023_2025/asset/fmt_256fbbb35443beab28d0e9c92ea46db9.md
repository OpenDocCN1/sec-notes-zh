
# [Android 分享] 九彩盒子固件升级提示关闭思路

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1975588
- 抓取时间: 2026-11-01T16:29:35.381Z

近期，我弄了一台电信的 MV310 机顶盒，并依照网上的教程对其进行了刷机操作，期间下载了九彩固件。然而，在使用一段时间后，每次开机都会弹出升级界面，而且无法关掉，导致无法正常使用。

![](https://data.znds.com/attachment/forum/202409/08/134224dyqzbmbe7xm9ssrs.png)

为寻求解决之法，我找到了一篇名为“九彩固件升级提示揭秘”的帖子。该帖通过对固件进行反编译，深入分析了固件升级的原因及具体代码。简而言之，九彩在代码中设置了一个计数器，当机顶盒运行一段时间后，便会弹出升级窗口。

![](https://attach.52pojie.cn/forum/202401/23/111429d43jkgugrxlu4vsz.png)

图片引用自文章：https://www.52pojie.cn/thread-1883700-1-1.html

依此思路，需对刷机包进行反编译并修改相应代码。但本人缺少反编译工具，数次尝试刷机均未成功。于是，我思考能否通过 `ADB` 来解决这一问题，首先想到的便是尝试禁止白屏应用的启动。

首先，将机顶盒开启开发者模式，开启 `ADB` 调试，并确保其与电脑处于同一局域网内进行连接。接着，将界面停留在白屏位置，运行以下代码：`PowerShell` 纯文本查看 复制代码

```
adb shell dumpsys activity activities
```

以下是输出的内容：`PowerShell` 纯文本查看 复制代码

```
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)

  Stack #0:

    Task id #1

    * TaskRecord{41ee52a0 #1 A=com.tvpusher.gklauncher U=0 sz=1}

      numActivities=1 rootWasReset=false userId=0 mTaskType=1 numFullscreen=1 mOnTopOfHome=false

      affinity=com.tvpusher.gklauncher

      intent={act=android.intent.action.MAIN cat=[android.intent.category.HOME] flg=0x10000000 cmp=com.tvpusher.gklauncher/.mvp.ui.activity.WelcomeActivity}

      realActivity=com.tvpusher.gklauncher/.mvp.ui.activity.WelcomeActivity

      Activities=[ActivityRecord{42051038 u0 com.tvpusher.gklauncher/.mvp.ui.activity.MainActivty t1}]

      askedCompatMode=false

      lastThumbnail=android.graphics.Bitmap@41ce2258 lastDescription=null

      lastActiveTime=8142 (inactive for 994s)

      * Hist #0: ActivityRecord{42051038 u0 com.tvpusher.gklauncher/.mvp.ui.activity.MainActivty t1}

          packageName=com.tvpusher.gklauncher processName=com.tvpusher.gklauncher

          launchedFromUid=10013 launchedFromPackage=com.tvpusher.gklauncher userId=0

          app=ProcessRecord{41ee6650 2387:com.tvpusher.gklauncher/u0a13}

          Intent { flg=0x400000 cmp=com.tvpusher.gklauncher/.mvp.ui.activity.MainActivty }

          frontOfTask=true task=TaskRecord{41ee52a0 #1 A=com.tvpusher.gklauncher U=0 sz=1}

          taskAffinity=com.tvpusher.gklauncher

          realActivity=com.tvpusher.gklauncher/.mvp.ui.activity.MainActivty

          baseDir=/system/app/yyzm8.9.apk

          dataDir=/data/data/com.tvpusher.gklauncher

          stateNotNeeded=false componentSpecified=true mActivityType=1

          compat={160dpi always-compat} labelRes=0x7f100029 icon=0x7f0e0044 theme=0x7f11018a

          config={1.0?mcc?mnc zh_CN ldltr sw720dp w1280dp h720dp 160dpi xlrg long land television -touch -keyb/v/h dpad/v s.4}

          launchFailed=false launchCount=0 lastLaunchTime=-16m35s738ms

          haveState=true icicle=Bundle[mParcelledData.dataSize=3632]

          state=STOPPED stopped=true delayedResume=false finishing=false

          keysPaused=false inHistory=true visible=false sleeping=false idle=true

          fullscreen=true noDisplay=false immersive=false launchMode=2

          frozenBeforeDestroy=false thumbnailNeeded=false forceNewConfig=false

          mActivityType=HOME_ACTIVITY_TYPE

          thumbHolder: 41ee52a0 bm=android.graphics.Bitmap@41ce2258 desc=null

    Running activities (most recent first):

      TaskRecord{41ee52a0 #1 A=com.tvpusher.gklauncher U=0 sz=1}

        Run #0: ActivityRecord{42051038 u0 com.tvpusher.gklauncher/.mvp.ui.activity.MainActivty t1}

    mLastPausedActivity: ActivityRecord{42051038 u0 com.tvpusher.gklauncher/.mvp.ui.activity.MainActivty t1}

  Stack #1:

    Task id #3

    * TaskRecord{41f3bfd0 #3 A=com.ex.service U=0 sz=1}

      numActivities=1 rootWasReset=false userId=0 mTaskType=0 numFullscreen=1 mOnTopOfHome=true

      affinity=com.ex.service

      intent={flg=0x10000000 cmp=com.ex.service/.Txtactivity}

      realActivity=com.ex.service/.Txtactivity

      Activities=[ActivityRecord{41d14248 u0 com.ex.service/.Txtactivity t3}]

      askedCompatMode=false

      lastThumbnail=null lastDescription=null

      lastActiveTime=870241 (inactive for 132s)

      * Hist #0: ActivityRecord{41d14248 u0 com.ex.service/.Txtactivity t3}

          packageName=com.ex.service processName=com.ex.service

          launchedFromUid=10011 launchedFromPackage=com.ex.service userId=0

          app=ProcessRecord{41fe73e8 2405:com.ex.service/u0a11}

          Intent { flg=0x10000000 cmp=com.ex.service/.Txtactivity }

          frontOfTask=true task=TaskRecord{41f3bfd0 #3 A=com.ex.service U=0 sz=1}

          taskAffinity=com.ex.service

          realActivity=com.ex.service/.Txtactivity

          baseDir=/system/priv-app/SystemManager.apk

          dataDir=/data/data/com.ex.service

          stateNotNeeded=false componentSpecified=true mActivityType=0

          compat={160dpi always-compat} labelRes=0x7f0e001b icon=0x0 theme=0x7f0f0186

          config={1.0?mcc?mnc zh_CN ldltr sw720dp w1280dp h720dp 160dpi xlrg long land television -touch -keyb/v/h dpad/v s.4}

          launchFailed=false launchCount=0 lastLaunchTime=-16m36s771ms

          haveState=false icicle=null

          state=RESUMED stopped=false delayedResume=false finishing=false

          keysPaused=false inHistory=true visible=true sleeping=false idle=true

          fullscreen=true noDisplay=false immersive=false launchMode=0

          frozenBeforeDestroy=false thumbnailNeeded=false forceNewConfig=false

          mActivityType=APPLICATION_ACTIVITY_TYPE

          thumbHolder: 41f3bfd0 bm=null desc=null

          waitingVisible=false nowVisible=true lastVisibleTime=-2m12s490ms

    Running activities (most recent first):

      TaskRecord{41f3bfd0 #3 A=com.ex.service U=0 sz=1}

        Run #0: ActivityRecord{41d14248 u0 com.ex.service/.Txtactivity t3}

    mResumedActivity: ActivityRecord{41d14248 u0 com.ex.service/.Txtactivity t3}

    mLastPausedActivity: ActivityRecord{41f82368 u0 com.fiberhome.settings/.MainActivity t9 f}

  mFocusedActivity: ActivityRecord{41d14248 u0 com.ex.service/.Txtactivity t3}

  mDismissKeyguardOnNextActivity=false

  mFocusedStack=ActivityStack{4200d170 stackId=1, 1 tasks} mStackState=STACK_STATE_HOME_IN_BACK

  mSleepTimeout=false

  mCurTaskId=9

  mUserStackInFront={}

  Recent tasks:

  * Recent #0: TaskRecord{41f3bfd0 #3 A=com.ex.service U=0 sz=1}

    numActivities=1 rootWasReset=false userId=0 mTaskType=0 numFullscreen=1 mOnTopOfHome=true

    affinity=com.ex.service

    intent={flg=0x10000000 cmp=com.ex.service/.Txtactivity}

    realActivity=com.ex.service/.Txtactivity

    Activities=[ActivityRecord{41d14248 u0 com.ex.service/.Txtactivity t3}]

    askedCompatMode=false

    lastThumbnail=null lastDescription=null

    lastActiveTime=870241 (inactive for 132s)

  * Recent #1: TaskRecord{420ab148 #9 I=com.fiberhome.settings/.MainActivity U=0 sz=0}

    intent={cmp=com.fiberhome.settings/.MainActivity}

    realActivity=com.fiberhome.settings/.MainActivity

    Activities=[]

    askedCompatMode=false

    lastThumbnail=android.graphics.Bitmap@41ce2258 lastDescription=null

    lastActiveTime=870236 (inactive for 132s)

  * Recent #2: TaskRecord{420e3978 #8 I=com.fiberhome.settings/.AuthActivity U=0 sz=0}

    intent={act=android.intent.action.MAIN flg=0x10000000 cmp=com.fiberhome.settings/.AuthActivity}

    realActivity=com.fiberhome.settings/.AuthActivity

    Activities=[]

    askedCompatMode=false

    lastThumbnail=android.graphics.Bitmap@41ce2258 lastDescription=null

    lastActiveTime=853653 (inactive for 149s)

  * Recent #3: TaskRecord{41ee52a0 #1 A=com.tvpusher.gklauncher U=0 sz=1}

    numActivities=1 rootWasReset=false userId=0 mTaskType=1 numFullscreen=1 mOnTopOfHome=false

    affinity=com.tvpusher.gklauncher

    intent={act=android.intent.action.MAIN cat=[android.intent.category.HOME] flg=0x10000000 cmp=com.tvpusher.gklauncher/.mvp.ui.activity.WelcomeActivity}

    realActivity=com.tvpusher.gklauncher/.mvp.ui.activity.WelcomeActivity
```
