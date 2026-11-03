
```
function hook_java() {
    Java.perform(function () {
        let NativeHelper = Java.use("com.zuoyebang.baseutil.NativeHelper");
        NativeHelper["nativeInitBaseUtil"].implementation = function (context, str) {
            console.log(`NativeHelper.nativeInitBaseUtil is called: context=${context}, str=${str}`);
            let result = this["nativeInitBaseUtil"](context, str);
            console.log(`NativeHelper.nativeInitBaseUtil result=${result}`);
            return result;
        };

        NativeHelper["nativeSetToken"].implementation = function (context, str, str2, str3) {
            console.log(`NativeHelper.nativeSetToken is called: context=${context}, str=${str}, str2=${str2}, str3=${str3}`);
            let result = this["nativeSetToken"](context, str, str2, str3);
            console.log(`NativeHelper.nativeSetToken result=${result}`);
            return result;
        };

        NativeHelper["nativeGetSign"].implementation = function (str) {
            console.log(`NativeHelper.nativeGetSign is called: str=${str}`);
            let result = this["nativeGetSign"](str);
            console.log(`NativeHelper.nativeGetSign result=${result}`);
            return result;
        };
    })
}

function hook_dlopen() {
    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");
    Interceptor.attach(android_dlopen_ext, {
        onEnter: function (args) {
            var path_ptr = args[0];
            var path = ptr(path_ptr).readCString();
            if (path && path.indexOf("libbaseutil.so") !== -1) {
                console.log("[android_dlopen_ext:]", path);
                this.flag = true;
            }
        },
        onLeave: function (retval) {
            if (this.flag) {
                const module = Process.findModuleByName("libbaseutil.so");
                let jni_onload = module.findExportByName("JNI_OnLoad")
                console.log("JNI_OnLoad--->>>" + jni_onload);
                Interceptor.attach(jni_onload, {
                    onEnter: function (args) {
                        console.log("JNI_OnLoad onEnter")

                    },
                    onLeave: function (retval) {
                        console.log("JNI_OnLoad onLeave")
                        hook_java()

                    }
                })
            }
        }
    });

}
setImmediate(hook_dlopen)
```

- 这里需要把握好时机的问题，一般来说直接用常规hook没输出的话就要考虑自己hook的时机是不是够早了，不行的话还可以往前找更早的时机；我这里是hook `JNI_OnLoad`后，时机是比较早的了；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114141453309.png)

- 可以发现，顺序分别是`nativeInitBaseUtil`、`nativeSetToken`、`nativeGetSign`；分别去`unidbg`调用，这里放出一份日志：

```
NativeHelper.nativeInitBaseUtil is called: 

context=com.kuaiduizuoye.scan.base.BaseApplication@90993f0, 

str=E0DA888804E90201780A9C0D17D0B050|0

NativeHelper.nativeInitBaseUtil result=0d040c0d0e0f01050c02040a0f0707040207050e05050a0c0b090d0f090f030f020400030a0f030903070e0708020300070f0f080306060b00000f0d04070d0d02070b0f020a01010c0d030a09080e090b040f02080c0b040c0e0e060c0d02010e0b0c050b02020c07050a06020a0c0e0c030b000e0f0f0907080c0c0e0b0404000d0b0903010f05080c040f0c0f0f060102080a050509040606040a0d0f0f0e0f05050a05000e0001010b020701000e

NativeHelper.nativeSetToken is called: 

context=com.kuaiduizuoye.scan.base.BaseApplication@90993f0, 

str=E0DA888804E90201780A9C0D17D0B050|0, str2=0d040c0d0e0f01050c02040a0f0707040207050e05050a0c0b090d0f090f030f020400030a0f03090307

0e0708020300070f0f080306060b00000f0d04070d0d02070b0f020a01010c0d030a09080e090b040f02080c0b040c0e0e060c0d02010e0b0c050b02020c07050a06020a0c0e0c030b000e0f0f0907080c0c0e0b0404000d0b0903010f05080c040f0c0f0f060102080a050509040606040a0d0f0f0e0f05050a05000e0001010b020701000e, str3=0c000a01090d0e04020c0a0e06010b0d050a080e0c0f03050c0a0602060e0c0207090c0a0d090f060a060e08030a0003

NativeHelper.nativeSetToken result=true
```

- 初步观察一下，`init`函数执行后得到一个值，`nativeSetToken`用到了它，并且`init`的参数`settoken`也用到了，那么实际上只需要执行`nativeSetToken`就可以跑通了，这里为了固定数据，我这里先只call `nativeSetToken`函数；

```
public void call_nativeSetToken(){
    List list = new ArrayList<>(10);
    list.add(vm.getJNIEnv());
    list.add(0);

    StringObject str1 = new StringObject(vm,"E0DA888804E90201780A9C0D17D0B050|0");
    StringObject str2 = new StringObject(vm,"0d040c0d0e0f01050c02040a0f0707040207050e05050a0c0b090d0f090f030f020400030a0f030903070e0708020300070f0f080306060b00000f0d04070d0d02070b0f020a01010c0d030a09080e090b040f02080c0b040c0e0e060c0d02010e0b0c050b02020c07050a06020a0c0e0c030b000e0f0f0907080c0c0e0b0404000d0b0903010f05080c040f0c0f0f060102080a050509040606040a0d0f0f0e0f05050a05000e0001010b020701000e");
    StringObject str3 = new StringObject(vm,"0c000a01090d0e04020c0a0e06010b0d050a080e0c0f03050c0a0602060e0c0207090c0a0d090f060a060e08030a0003|0");
    DvmObject context = vm.resolveClass("android/content/Context").newObject(null);// context
    list.add(vm.addLocalObject(context));
    list.add(vm.addLocalObject(str1));
    list.add(vm.addLocalObject(str2));
    list.add(vm.addLocalObject(str3));
    Number number = module.callFunction(emulator, 0xe90 + 1, list.toArray());
    StringObject str = vm.getObject(number.intValue());
}
```

- 不需要不环境，直接得到结果：

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114143106237.png)

- 但是`call_nativeSetToken`的三个参数我们均未知，所以先去看看到底是什么；先看`init`函数；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114143439386.png)

- 参数一自然不用多说，参数二实际上是`cuid`，和抓包上传的也对得上；这里我清除数据发现依旧是这个值，可能与设备有关，这里我不去追究来源了；返回值怎么算的这里先不管，去看`nativeSetToken`函数；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114144428713.png)

- 前面说了，它的参数是`context`、`cuid`，还有`a2`、`a3`，从代码语义上看，`a2`、`a3`原先是没有值的，`a2`就是`init`函数算出来的结果，后续应该是存在了`xml`文件里，因为上面在取；`a3`是`plutoAntispam.data`；看起来`postSync`可能是和请求有关，问了问ai说是同步网络POST请求，那我们去搜一下，这里幸好抓包没停；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114144850006.png)

- 确实找到了，参数还是那一些，另外多说一句，我是重装+清数据过的，但是前面有疑问的`adid`、`token`之类的请求体还是固定的，那就证明与设备有关，所以后面不会再提这些参数了，统一当做已知数据；

- 现在大概清楚了数据来源，现在暂时停下脚步，看看我们还有什么是未知的；

- 一共出现了三个函数，我们称为`init`、`setToken`、`getsign`；

- `init`有两个参数`context`、`cuid`均为已知，结果来源未知，而结果后续用到了，所以需要分析算法，记为`init_res`；

- 而后发起一个请求，参数主要有`init_res`，响应记为`req_res`；

- `setToken`函数有四个参数`context`、`cuid`、`init_res`、`req_res`，依旧需要解决`init_res`，结果为`true`；

- `getsign`函数参数已知，结果来源未知，需要分析；

- 所以需要分析的有`init`函数的返回值、`getsign`函数的返回值；

### 3.2 `nativeInitBaseUtil`

- 开始分析第一部分，`nativeInitBaseUtil`函数，它的偏移是`0xd15`，我们跳过去看看；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114150353020.png)

- 这里的符号是我修改过的，看起来依旧清爽，我稍微修改了一样参数1的类型；函数不多，看样子需要先看`sub_3C6C`；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114150808948.png)

- 生成随机10位字符串，再看第二个函数 `sub_1C18`；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114150906891.png)

- 这个函数熟悉的读者可能会很熟悉，有点像说废话了，实际上是一个`md5`函数；`MD5`初始常量摆在那里，后续的两个函数分别是`md5_update`和`final`函数；标不标准我们先不管，继续往下看，现在只是预分析而已；

- `sub_1EF0`不去看了，基本上是格式化字符串的函数，`sub_3688`可能与`des`有关；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114152606308.png)

- 最后大概是这些函数；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114152643420.png)

- 下面开始分析整个算法，先call吧；

```
public void call_init() {

    List list = new ArrayList<>(10);

    list.add(vm.getJNIEnv());

    list.add(0);

    list.add(vm.addLocalObject(vm.resolveClass("android/content/Context").newObject(null)));

    list.add(vm.addLocalObject(new StringObject(vm, "E0DA888804E90201780A9C0D17D0B050|0")));

    Number ret = module.callFunction(emulator, 0xd14 + 1, list.toArray());

    StringObject str = vm.getObject(ret.intValue());
}
```
