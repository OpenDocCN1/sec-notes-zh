
```
package com.zj.wuaipojie.util;

import com.github.unidbg.AndroidEmulator;
import com.github.unidbg.Emulator;
import com.github.unidbg.Module;
import com.github.unidbg.arm.HookStatus;
import com.github.unidbg.file.FileResult;
import com.github.unidbg.file.IOResolver;
import com.github.unidbg.file.linux.AndroidFileIO;
import com.github.unidbg.hook.HookContext;
import com.github.unidbg.hook.ReplaceCallback;
import com.github.unidbg.hook.hookzz.Dobby;
import com.github.unidbg.linux.android.AndroidEmulatorBuilder;
import com.github.unidbg.linux.android.AndroidResolver;
import com.github.unidbg.linux.android.SystemPropertyHook;
import com.github.unidbg.linux.android.SystemPropertyProvider;
import com.github.unidbg.linux.android.dvm.*;
import com.github.unidbg.linux.android.dvm.array.ArrayObject;
import com.github.unidbg.linux.file.ByteArrayFileIO;
import com.github.unidbg.linux.file.SimpleFileIO;
import com.github.unidbg.memory.Memory;
import com.github.unidbg.pointer.UnidbgPointer;
import java.io.File;
import java.io.FileNotFoundException;
import java.nio.charset.StandardCharsets;

public class ChallengeTenTwo implements IOResolver {

    private final AndroidEmulator emulator;
    private final VM vm;
    private final Module module;
    // 将所有需要伪造的数据定义为常量，方便管理和修改。
    private static final String PACKAGE_NAME = "com.zj.wuaipojie\0";
    private static final String EXPECTED_BOOT_ID = "8442043f-98e2-48bb-8707-525e98a66898";
    private static final String EXPECTED_BOARD_PLATFORM = "kalama";
    private static final String EXPECTED_PHONE_MODEL = "23113RKC6C";
    private static final String EXPECTED_SIG  =  "3082034f30820237a00302010202046c07fed4300d06092a864886f70d01010b05003058310f300d06035504061306303030303030310b300906035504081302626a310b300906035504071302626a310c300a060355040a13036f727a310c300a060355040b13036f727a310f300d06035504030c06e6ada3e5b7b1301e170d3232303832313132323934305a170d3437303831353132323934305a3058310f300d06035504061306303030303030310b300906035504081302626a310b300906035504071302626a310c300a060355040a13036f727a310c300a060355040b13036f727a310f300d06035504030c06e6ada3e5b7b130820122300d06092a864886f70d01010105000382010f003082010a0282010100e6f5d0394247b0a7bb02daedeb78f4d373d394cde087c8edbd9f5fa87781f88ed733f822c10e6573a4f9fa52c46fa7cbc48136f35f0a554fe6ab6c605cab00c1edaa18ab2d9d052968413053fc72c945e9be05dea00d3ebb9a2404d2fd2e5520d784d739de28b9a3611c7ccc51df75e057d054c984931627381d572ff545047ae089d36a99ac5866aa236e971fa322e500335474ae5932142b14464e000605c53ce902a53aaba2bcd576a255db6913c849e144d4cb83ca9043078160025219205ef859f2611c89a168f153dff6020a992a77fe2ab8d37ec3d65e97a4c74577f2f5c6ae690afe26a2f97de7697046ae1ef18c986d437c77a83e22c272485487bd0203010001a321301f301d0603551d0e04160414cd08d93fc2483c6afa2e182a5357acbc6cc21e17300d06092a864886f70d01010b05000382010100a46400714e97485805062c6fe392f61a52f52083b616a170affb043cc3272332132adfe55db2b13abc7bbbcbf2ae83741e767d196deb3b924cd7037ed6afa5360ea10505ed8ba37c33ce7ac5a24aa6192b49c02ad711b99c1eec185aed272671c3dfdc7df5c3c0511b41b581c94ddf1076e397b27a75a0b73e5e207f4ec636067540b85e3e750ce59599ddb1d954e4537275d27227f2db9eb2fa3a2a29ea1c8a5415d6058e798517ebec4194cfb8715e7c17c043c0309c7e9dd7c17c502b3dd27153929ba5462182a0aa58c15972bc75a7ede1849ef5c9341d0ca747c9fb41ccabfa2c04c1147ea884cf951481bca37ffe929552962b35cbffff21675e4d5581";

    public ChallengeTenTwo() {

        emulator = AndroidEmulatorBuilder.for64Bit()
                .setProcessName(PACKAGE_NAME)
                .build();

        final Memory memory = emulator.getMemory();
        memory.setLibraryResolver(new AndroidResolver(23));

        // 注册 IOResolver 来处理文件访问
        emulator.getSyscallHandler().addIOResolver(this);

        // 创建 VM
        vm = emulator.createDalvikVM();
        //vm.setVerbose(true);
//        SystemPropertyHook systemPropertyHook = new SystemPropertyHook(emulator);
//        systemPropertyHook.setPropertyProvider(new SystemPropertyProvider() {
//            @Override
//            public String getProperty(String key) {
//                switch (key){
//                    case "ro.board.platform":{
//                        return "kalama";
//                    }
//                    case "ro.product.model":{
//                        return "23113RKC6C";
//                    }
//                }
//                return null;
//            }
//        });
//        memory.addHookListener(systemPropertyHook);

        vm.setJni(new MyJni(vm));
        // 加载 so 并执行 JNI_OnLoad
        File soFile = new File("unidbg-android/src/test/java/com/zj/wuaipojie/util/lib52pojie.so");
        DalvikModule dm = vm.loadLibrary(soFile, true);
        module = dm.getModule();
        dm.callJNI_OnLoad(emulator);

        // 【关键】Hook libc.so 中的 __system_property_get 函数
        hookSystemPropertyGet();
    }

    /**
     * Hook __system_property_get 函数，用于伪造系统属性
     */

    private void hookSystemPropertyGet() {
        Module libc = emulator.getMemory().findModule("libc.so");
        if (libc == null) {
            throw new IllegalStateException("Failed to find libc.so");
        }
        long address = libc.findSymbolByName("__system_property_get").getAddress();

        Dobby dobby = Dobby.getInstance(emulator);
        dobby.replace(address, new ReplaceCallback() {
            @Override
            public HookStatus onCall(Emulator emulator, HookContext context, long originFunction) {
                UnidbgPointer keyPtr = context.getPointerArg(0);
                UnidbgPointer valuePtr = context.getPointerArg(1);

                String key = keyPtr.getString(0);
                String value = null;

                if ("ro.board.platform".equals(key)) {
                    value = EXPECTED_BOARD_PLATFORM;
                    System.out.println("[HOOK] __system_property_get: Intercepted key=" + key + ", Faking value=" + value);
                } else if ("ro.product.model".equals(key)) {
                    value = EXPECTED_PHONE_MODEL;
                    System.out.println("[HOOK] __system_property_get: Intercepted key=" + key + ", Faking value=" + value);
                }

                if (value != null) {
                    byte[] valueBytes = value.getBytes(StandardCharsets.UTF_8);
                    valuePtr.write(0, valueBytes, 0, valueBytes.length);
                    valuePtr.setByte(valueBytes.length, (byte) 0);
                    return HookStatus.LR(emulator, valueBytes.length);
                }

                return HookStatus.RET(emulator, originFunction);
            }
        });
    }

    public void callUnidbgLevel2() {
        System.out.println("====== 开始执行 unidbg_level2 函数 ======");
        DvmClass securityUtilClass = vm.resolveClass("com/zj/wuaipojie/util/SecurityUtil");
        DvmClass ctxClz = vm.resolveClass("android/content/Context");
        DvmObject mockCtx = ctxClz.newObject(null);
        StringObject result = securityUtilClass.callStaticJniMethodObject(
                emulator, "unidbg_level2(Landroid/content/Context;)Ljava/lang/String;",
                mockCtx);
        System.out.println("JNI 函数返回结果: " + (result != null ? result.getValue() : null));

    }

    public static void main(String[] args) throws FileNotFoundException {
        ChallengeTenTwo challenge = new ChallengeTenTwo();
        challenge.callUnidbgLevel2();
    }

    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {
        // 无论是否处理，都打印出来，这是发现未知文件访问的关键。
        System.out.println("[IOResolver] Intercepted file access -> Path: '" + pathname + "', Flags: " + oflags);

        // 使用 switch 语句来处理不同的文件路径
        switch (pathname) {
            // 关卡1: 校验 /proc/sys/kernel/random/boot_id
            case "/proc/sys/kernel/random/boot_id":
                // 返回一个包含预设 boot_id 的文件
                return FileResult.success(new ByteArrayFileIO(oflags, pathname, EXPECTED_BOOT_ID.getBytes()));

//            // 关卡2: 校验 /proc/self/cmdline
            case "/proc/self/cmdline":
                return FileResult.success(new ByteArrayFileIO(oflags, pathname, PACKAGE_NAME.getBytes()));

//            // 关卡3: 校验 /proc/self/status，反调试检测
            case "/proc/self/status":
                // 返回一个包含 "TracerPid: 0" 的文件内容，表示未被调试
                String statusContent = "Name:\t" + PACKAGE_NAME + "\n" +
                        "Umask:\t0077\n" +
                        "State:\tS (sleeping)\n" +
                        "Tgid:\t12345\n" +
                        "Pid:\t12345\n" +
                        "PPid:\t1\n" +
                        "TracerPid:\t0\n"; // 关键行
                return FileResult.success(new ByteArrayFileIO(oflags, pathname, statusContent.getBytes()));

//            // 关卡4: 校验 /proc/self/maps，应用完整性检测
            case "/proc/self/maps": {
                final String APK_PATH = "/data/app/com.zj.wuaipojie-1/base.apk";
                String maps = "7fbe852000-7fbe853000 r-xp 00000000 00:00 0 " + APK_PATH + "\n";
                return FileResult.success(new ByteArrayFileIO(oflags, pathname, maps.getBytes(StandardCharsets.UTF_8)));
            }

        }

        // 对于其他未处理的文件路径，返回 null，让 unidbg 使用默认的文件处理逻辑
        return null;
    }
    /** 自定义 JNI，伪造 PackageManager / PackageInfo / Signature */
    static class MyJni extends AbstractJni {
        private final VM vm;
        MyJni(VM vm) { this.vm = vm; }
        @Override
        public DvmObject callObjectMethodV(BaseVM baseVm, DvmObject obj, String signature, VaList vaList) {
            // Context.getPackageManager()
            if (signature.equals("android/content/Context->getPackageManager()Landroid/content/pm/PackageManager;")) {
                DvmClass pmClz = vm.resolveClass("android/content/pm/PackageManager");
                return pmClz.newObject("mockPM");
            }

            // PackageManager.getPackageArchiveInfo(String, int) -> PackageInfo
            if (signature.equals("android/content/pm/PackageManager->getPackageArchiveInfo(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;")) {
                StringObject apkPathObj = vaList.getObjectArg(0);
                int flags = vaList.getIntArg(1); // 64 (GET_SIGNATURES)
                // 这里不做文件解析，直接返回一个填好字段的 PackageInfo
                DvmClass pkgInfoClz = vm.resolveClass("android/content/pm/PackageInfo");
                // 用 Map 保存字段，后续在 getObjectField 里按字段名返回
                java.util.HashMap fields = new java.util.HashMap<>();
                fields.put("packageName", new StringObject(vm, PACKAGE_NAME));

                // signatures: [Landroid/content/pm/Signature;
                DvmClass sigClz = vm.resolveClass("android/content/pm/Signature");
                DvmObject sigObj = sigClz.newObject("mockSignature"); // 值随便放个占位
                ArrayObject sigArr = new ArrayObject(sigObj);
                fields.put("signatures", sigArr);

                return pkgInfoClz.newObject(fields);
            }

            // Signature.toCharsString() -> String
            if (signature.equals("android/content/pm/Signature->toCharsString()Ljava/lang/String;")) {
                return new StringObject(vm, EXPECTED_SIG);
            }

            return super.callObjectMethodV(baseVm, obj, signature, vaList);
        }

        @Override
        public DvmObject getObjectField(BaseVM baseVm, DvmObject obj, String signature) {
            // PackageInfo.packageName
            if (signature.equals("android/content/pm/PackageInfo->packageName:Ljava/lang/String;")) {
                Object v = ((java.util.Map) obj.getValue()).get("packageName");
                return (DvmObject) v;
            }
            // PackageInfo.signatures
            if (signature.equals("android/content/pm/PackageInfo->signatures:[Landroid/content/pm/Signature;")) {
                Object v = ((java.util.Map) obj.getValue()).get("signatures");
                return (DvmObject) v;
            }
            return super.getObjectField(baseVm, obj, signature);
        }
    }
}
```

## 四、请作者喝杯咖啡

![](https://pic.rmb.bdstatic.com/bjh/241115/0595862b33c3749fe0addb56298b2e092560.png)

## 六、视频及课件地址

[百度云](https://pan.baidu.com/s/1cFWTLn14jeWfpXxlx3syYw?pwd=nqu9)

[阿里云](https://www.aliyundrive.com/s/TJoKMK6du6x)

[哔哩哔哩](https://www.bilibili.com/video/BV1wT411N7sV/?spm_id_from=333.788&vd_source=6dde16dc6479f00694baaf73a2225452)

[教程开源地址](https://github.com/ZJ595/AndroidReverse)

PS: 解压密码都是 52 pj，阿里云由于不能分享压缩包，所以下载 exe 文件，双击自解压

## 七、其他章节

[《安卓逆向这档事》一、模拟器环境搭建](https://www.52pojie.cn/thread-1695141-1-1.html)

[《安卓逆向这档事》二、初识APK文件结构、双开、汉化、基础修改](https://www.52pojie.cn/thread-1695796-1-1.html)

[《安卓逆向这档事》三、初识smail，vip终结者](https://www.52pojie.cn/thread-1701353-1-1.html)

[《安卓逆向这档事》四、恭喜你获得广告&弹窗静默卡](https://www.52pojie.cn/thread-1706691-1-1.html)

[《安卓逆向这档事》五、1000-7=？&动态调试&Log插桩](https://www.52pojie.cn/thread-1714727-1-1.html)

[《安卓逆向这档事》六、校验的N次方-签名校验对抗、PM代{过](https://www.52pojie.cn/thread-1731181-1-1.html){滤}理、IO重定向}

[《安卓逆向这档事》七、Sorry，会Hook真的可以为所欲为-Xposed快速上手(上)模块编写,常用Api](https://www.52pojie.cn/thread-1740944-1-1.html)

[《安卓逆向这档事》八、Sorry，会Hook真的可以为所欲为-xposed快速上手(下)快速hook](https://www.52pojie.cn/thread-1748081-1-1.html)

[《安卓逆向这档事》九、密码学基础、算法自吐、非标准加密对抗](https://www.52pojie.cn/thread-1762225-1-1.html)

[《安卓逆向这档事》十、不是我说，有了IDA还要什么女朋友？](https://www.52pojie.cn/thread-1787667-1-1.html)

[《安卓逆向这档事》十二、大佬帮我分析一下](https://www.52pojie.cn/thread-1809646-1-1.html)

[《安卓逆向这档事》番外实战篇1-某电影视全家桶](https://www.52pojie.cn/thread-1814917-1-1.html)

[《安卓逆向这档事》十三、是时候学习一下Frida一把梭了(上)](https://www.52pojie.cn/thread-1823118-1-1.html)

[《安卓逆向这档事》十四、是时候学习一下Frida一把梭了(中)](https://www.52pojie.cn/thread-1838539-1-1.html)

[《安卓逆向这档事》十五、是时候学习一下Frida一把梭了(下)](https://www.52pojie.cn/thread-1840174-1-1.html)

[《安卓逆向这档事》十六、是时候学习一下Frida一把梭了(终)](https://www.52pojie.cn/thread-1859820-1-1.html)

[《安卓逆向这档事》十七、你的RPCvs佬的RPC](https://www.52pojie.cn/thread-1892127-1-1.html#/)

[《安卓逆向这档事》番外实战篇2-【2024春节】解题领红包活动，启动!](https://www.52pojie.cn/thread-1893708-1-1.html#/)

[《安卓逆向这档事》十八、表哥，你也不想你的Frida被检测吧!(上)](https://www.52pojie.cn/thread-1921073-1-1.html)

[《安卓逆向这档事》十九、表哥，你也不想你的Frida被检测吧!(下)](https://www.52pojie.cn/thread-1938862-1-1.html)

[《安卓逆向这档事》二十、抓包学得好，牢饭吃得饱(上)](https://www.52pojie.cn/thread-1945285-1-1.html)

[《安卓逆向这档事》番外实战篇3-拨云见日之浅谈Flutter逆向](https://www.52pojie.cn/thread-1951619-1-1.html)

[《安卓逆向这档事》第二十一课、抓包学得好，牢饭吃得饱(中)](https://www.52pojie.cn/thread-1967845-1-1.html)

[《安卓逆向这档事》第二十二课、抓包学得好，牢饭吃得饱(下)](https://www.52pojie.cn/thread-1982444-1-1.html)

[《安卓逆向这档事》第二十三课、黑盒魔法之Unidbg](https://www.52pojie.cn/thread-1995107-1-1.html)

[《安卓逆向这档事》第二十四课、Unidbg之补完环境我就睡(上)](https://www.52pojie.cn/thread-2058523-1-1.html)

## 八、参考文档

[白龙unidbg教程](https://blog.csdn.net/qq_38851536/category_11102404.html)

[凯神博客](https://kkkkkkkem.gitbook.io/ni-xiang-xue-xi-bi-ji/shi-yong-gong-ju/3.-unidbg/3.1-unidbg-de-ji-ben-jie-shao)

[app逆向](https://blog.csdn.net/weixin_74178589/article/details/140778060?ops_request_misc=%257B%2522request%255Fid%2522%253A%252234A4DE6C-B96F-49FE-AC98-E5655001E380%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=34A4DE6C-B96F-49FE-AC98-E5655001E380&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduend~default-2-140778060-null-null.nonecase&utm_term=unidbg&spm=1018.2226.3001.4450)

[【转载】Unidbg Hook 大全](https://blog.seeflower.dev/archives/67/)
