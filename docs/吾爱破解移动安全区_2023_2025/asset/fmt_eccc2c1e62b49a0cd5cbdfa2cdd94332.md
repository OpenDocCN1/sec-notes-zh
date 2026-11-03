
```js
console.log("------------getByteHash-ret----------")

console.log(Memory.readCString(this.x1));

console.log(Memory.readCString(retval));

            }

        })

    }

});
```

## hook `NewStringUTF()`

某参数为 `xxx`：`XYAAAAAQAAAAEAAAB......`，用 `jadx` 搜 `xxx` 搜不到关键字。怀疑是在 C 层生成的。因为开头固定是 `XYAAAAAQAAAAEAAAB`，所以查找以它开头的字符串。

```js
var symbols = Module.enumerateSymbolsSync("libart.so"); // 找系统内部so
var addrNewStringUTF = null;
for (var i = 0; i = 0 && symbol.name.indexOf("CheckJNI")
```

## 寻找 so 文件

### 静态注册

```js
Java.perform() {

    var dlsymadd = Module.findExportByName("libdl.so", 'dlsym');

    Interceptor.attach(dlsymadd, {

        onEnter: function(args) {

            this.info = args[1]; // Java_包名_类名_方法名

        }, onLeave: function(retval) {

            // the so file: module.name

            var module = Process.findModuleByAddress(retval);

            if (module == null) {

                return retval;

            }

            // native method

            var funcName = this.info.readCString();

            // 修改这里的方法名即可

            if (funcName.indexOf("getHNASignature") !== -1) {

                console.log(module.name);

                console.log('\t', funcName);

            }

            return retval;

        }

    })

}
```

### 动态注册

```js
var symbols = Module.enumerateSymbolsSync("libart.so");
var addrRegisterNatives = null;
for (var i = 0; i = 0 &&
        symbol.name.indexOf("JNI") >= 0 &&
        symbol.name.indexOf("RegisterNatives") >= 0 &&
        symbol.name.indexOf("CheckJNI")
```

## 延迟 hook

### 利用系统底层检测是否加载并 hook（和下面下面重复了）

当某些 `so` 文件还没加载时，`hook` 会报错。用这个脚本可以避免报错，不用在 `hook` 的时候拼手速找时机。

```js
function do_hook() {

    var addr = Module.findExportByName("libkeyinfo.so", "getByteHash");

    console.log(addr); //0xb696387d

    Interceptor.attach(addr, {

        onEnter: function (args) {

            this.x1 = args[2];

        },

        onLeave: function (retval) {

            console.log("--------------------")

            console.log(Memory.readCString(this.x1));

            console.log(Memory.readCString(retval));

        }

    })

}

function load_so_and_hook() {

    var dlopen = Module.findExportByName(null, "dlopen");

    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");

    Interceptor.attach(dlopen, {

        onEnter: function (args) {

            var path_ptr = args[0];

            var path = ptr(path_ptr).readCString();

            console.log("[dlopen:]", path);

            this.path = path;

        }, onLeave: function (retval) {

            if (this.path.indexOf("libkeyinfo.so") !== -1) { // 如果包含我想要的so文件

                console.log("[dlopen:]", this.path);

                do_hook();

            }

        }

    });

    Interceptor.attach(android_dlopen_ext, {

        onEnter: function (args) {

            var path_ptr = args[0];

            var path = ptr(path_ptr).readCString();

            this.path = path;

        }, onLeave: function (retval) {

            if (this.path.indexOf("libkeyinfo.so") !== -1) {

                console.log("\nandroid_dlopen_ext加载：", this.path);

                do_hook();

            }

        }

    });

}

load_so_and_hook();

// frida -U -f com.achievo.vipshop -l delay_hook.js
```

如果上面还是报错找不到，就在 `do_hook()` 再延迟一下。

```js
function do_hook() {
    setTimeout(function () { // 延时
        Java.perform(function () {
            var XhsHttpInterceptor = Java.use('com.xingin.shield.http.XhsHttpInterceptor');
            XhsHttpInterceptor.initialize.implementation = function (str) {
                console.log("str=", str);
                return this.initialize(str);
            };
        })
    }, 10); // delay 10 ms
}
```

### 用时间延迟

如果上面的方法还是 `hook` 不到。

```js
setTimeout(function() {

    // code

}, 1000) // delay one second

setTimeout((x, y)=>{
    console.log(x + y);
}, 3000, 1, 2);
```

第一个参数是一个匿名函数，在第三个参数位置传递了两个参数值 `1`, `2`，分别代表 `x` 和 `y` 的值，因此在 3 秒后会打印出 `3` 这个信息。

### 检测文件是否加载

```js
function do_hook() {

        ... // hook脚本

}

function delay_hook(so_name, hook_func) {

    var dlopen = Module.findExportByName(null, "dlopen");

    var android_dlopen_ext = Module.findExportByName(null, "android_dlopen_ext");

    Interceptor.attach(dlopen, {

        onEnter: function (args) {

            var path_ptr = args[0];

            var path = ptr(path_ptr).readCString();

            // console.log("[dlopen:]", path);

            this.path = path;

        }, onLeave: function (retval) {

            if (this.path.indexOf(so_name) !== -1) {

                console.log("[dlopen:]", this.path);

                hook_func();

            }

        }

    });

    Interceptor.attach(android_dlopen_ext, {

        onEnter: function (args) {

            var path_ptr = args[0];

            var path = ptr(path_ptr).readCString();

            this.path = path;

        },

        onLeave: function (retval) {

            if (this.path.indexOf(so_name) !== -1) {

                console.log("\nandroid_dlopen_ext加载：", this.path);

                hook_func();

            }

        }

    });

}

delay_hook("libkeyinfo.so", do_hook);

// frida -U -f  com.achievo.vipshop -l  hook.js
```

## Java 层

### 调用栈

```js
console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));
```

### hook 拦截器

使用 `JSON.stringify(inter)` 把对象是什么打印出来，可以知道是哪些类实例化了拦截器。`app` 需要重启：拦截器的注册在 `app` 初始的时候执行，而不是每次发送请求都注册一遍。

```js
Java.perform(function () {

    var Builder = Java.use('okhttp3.OkHttpClient$Builder');

    Builder.addInterceptor.implementation = function (inter) {

        //console.log("实例化：");

        console.log(JSON.stringify(inter));

        //console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));

        return this.addInterceptor(inter);

    };

})

// spawn 不要用 attach Interceptor可能在app启动的时候加载

// frida -U -l hook.js -f com.xxx.xxxx --no-pause

// frida -Uf com.hupu.shihuo -l hook.js -o all_interceptor.txt  15.2.2版本不可用
```

输出 eg. 从下往上找，记得手机上点击触发。

```
            ...
            ""
            ""
```

```js
Java.perform(function () {

   // ... 根据上面的输出 找到为止

    var a9 = Java.use('cn.shihuo.modulelib.startup.core.c.b');

    a9.intercept.implementation = function (chain) {

        var request = chain.request();

        var urlString = request.url().toString();

        if(urlString.indexOf("https://sh-gateway.shihuo.cn/v4/services/sh-goodsapi/app_swoole_zone/getAttributes/v")!= -1){ // 过滤目标url

            console.log("拦截器9-->", urlString);

        }

        var response = chain.proceed(request); // 不执行当前拦截器 而走下一个拦截器

        return response;

    };

    var a10 = Java.use('cn.shihuo.modulelib.startup.core.c.a');

    a10.intercept.implementation = function (chain) {

        //console.log("拦截器10", chain);

        var request = chain.request();

        var urlString = request.url().toString();

        if(urlString.indexOf("https://sh-gateway.shihuo.cn/v4/services/sh-goodsapi/app_swoole_zone/getAttributes/v") != -1){

            console.log("拦截器10-->", urlString);

        }

        //console.log("拦截器",this.b.value);

        var res = this.intercept(chain);

        //console.log(Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new()));

        return res;

    }

})

// frida -UF -l hook.js
```

#### 查看拦截器中 `Request` 的 `header`/`body` 等（`okhttp3` 的通法）
