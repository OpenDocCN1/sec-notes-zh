
# [Android 原创] x红书水印去除Frida脚本和Xposed模块

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2044329
- 抓取时间：2026-11-01T15:43:19.814Z

平时都用x红书下载头像、壁纸，会有水印，尝试`hook`一下加水印的操作，看看能不能获得无水印的图。

## Frida脚本

### 定位hook点

调研了一些常用的加水印方式，直接跑一下`frida`，看看哪些API用到了。

（ps: 在此之前是x红书的`frida`检测，这里就不细说了，是常用的`libmsaoaidsec.so`库。b站、豆瓣、爱奇艺都是用的这个，过的手段都一样，`hook dlopen`修改`pthread_creat`的返回值）

#### 一些常用的API

1. `Bitmap.compress`
2. `FileOutputStream.write`
3. `MediaStore.insertImage`
4. `Canvas.drawText`
5. `Canvas.drawBitmap`

跑一遍可以发现定位到了几个点。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250630222358476.png?raw=true)

##### `Bitmap.compress`

✅ 作用：将`Bitmap`压缩并输出到文件或流中。

通常用于保存图片到磁盘（例如PNG/JPEG格式），会被调用来将内存中的`Bitmap`转成可写入文件的数据。

##### `Canvas.drawText`

✅ 作用：在`Canvas`（画布）上绘制文字。

通常用于：

- 添加文字水印
- 显示图像上的文字说明
- 合成图像时叠加文字

那这个大概就是添加水印文字（x红书号）的API。

##### `Canvas.drawBitmap`

✅ 作用：在`Canvas`上绘制`Bitmap`图像。

常用于：

- 把一个`Bitmap`绘制到另一个`Bitmap`（比如背景 + 水印图）
- 图像合成处理（Logo、滤镜）
- 自定义控件、相机拍照回显等场景

那这个大概就是添加水印图像（x红书log）的API。

### 替换操作

#### 去除文字水印

前面猜测是`Canvas.drawText` API，根据调用规则进行`hook`：

```java
drawText(String text, float x, float y, Paint paint)

// text 是文字内容，x 和 y 是文字的坐标。
```

通过打印`text`来验证是否为添加文字水印的地方，然后在返回值处将`text`替换为空：

```javascript
try {
    Canvas.drawText.overload(
        "java.lang.String", "float", "float", "android.graphics.Paint"
    ).implementation = function (text, x, y, paint) {
        console.log("Canvas.drawText() called with text: " + text);
        console.log("Canvas.drawText() called with x, y: " + x + "   " + y);
        if (text.toLowerCase().includes("小红书") || text.toLowerCase().includes("xhs")) {
            console.warn(">>>>> Suspected watermark text found!");
        }
        //将原有的 text（小红书号:xxxxx)替换为空
        return this.drawText('', x, y, paint);
    };
} catch (e) {
    console.error("[!] Error hooking Canvas.drawText:", e);
}
```

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250625212323446.png?raw=true)

成功去除了文字水印，这个API定位是对的。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250630224207298.png?raw=true)

#### 去除图片logo水印

```java
drawBitmap(bitmap, float left, float top, Paint paint)

//参数：bitmap：图片对象，left:偏移左边的位置，top： 偏移顶部的位置
```

这部分一共有两个思路。

##### 构造透明 `bitmap` 伪造水印

根据这块`bitmap`的大小，太大了不可能是水印。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250625214547303.png?raw=true)

一开始猜测的`drawBitmap`是将水印附加在原图上，实际操作发现，`hook`到的`drawBitmap`其实是第一步，将原图叠加到一个纯黑的背景上，之后才调用另一个API附加到这个上面（尝试过发现下载了个黑图）。

继续批量测试一些`bitmap`的重载，最后定位到有`Rect`参数的API上：

```java
// 重载1: drawBitmap(Bitmap, float, float, Paint)
// 重载2: drawBitmap(Bitmap, Rect, Rect, Paint)
等
```

这里的大小就合适了，测试发现确实是它。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250625215627556.png?raw=true)

直接将`bitmap`转换为PNG下载下来可以发现，这两个`rect`尺寸的是水印图。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250630224841145.png?raw=true)

```javascript
const FileOutputStream = Java.use("java.io.FileOutputStream");
const File = Java.use("java.io.File");
const CompressFormat = Java.use("android.graphics.Bitmap$CompressFormat");
const Bitmap = Java.use("android.graphics.Bitmap");

function dumpBitmap(bitmap, name) {
    try {
        const file = File.$new("/sdcard/" + name + ".png");
        const out = FileOutputStream.$new(file);
        bitmap.compress(CompressFormat.PNG.value, 100, out);
        out.close();
        console.log(`Bitmap dumped: ${file.getAbsolutePath()}`);
    } catch (e) {
        console.error("Bitmap dump failed:", e);
    }
}
```

最后就获得了没有任何水印的图片。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250630225018753.png?raw=true)

##### hook原图并保存在指定位置

另一种方法就是，将图片

```javascript
Canvas.drawBitmap.overload(
"android.graphics.Bitmap", "float", "float", "android.graphics.Paint"
).implementation = function (bitmap, left, top, paint) {
    const width = bitmap.getWidth();
    const height = bitmap.getHeight();
    dumpBitmap(bitmap, "hook1")
    console.log("drawBitmap called: w=" + width + ", h=" + height + ", top=" + top);
```

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250630225208099.png?raw=true)

获得无水印原图。

![](https://github.com/SimpsonGet/SimpsonGet.github.io/blob/main/images/posts/xhsUnwatermark/image-20250630225220961.png?raw=true)

（脚本详见GitHub）

[//github.com/SimpsonGet/XhsWatermarkRemove](https://github.com/SimpsonGet/XhsWatermarkRemove:https)

## Xposed模块实现

在`frida`实现之后就想着形成`xposed`模块安装在手机上，固化驻留。

一开始是直接改写了`frida`脚本，使用`XposedHelpers.findAndHookMethod`方法来`hook` `drawText`、`drawBitmap`这两个方法，但是时机运行过程中无法hit到这两个方法，查阅资料后：

首先，`xposed`模块只能`hook`目标app自身类加载器加载的类。使用：

```java
public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
    // Hook 代码写这里
}
```

去`hook`方法时，能访问的是App的默认类加载器`PthClassLoader`。而

```java
XposedHelpers.findClass("android.graphics.Canvas", lpparam.classLoader);
```

访问的系统类是由`BootClassLoader`加载的。因此改为采用更加底层的`XposedBridge.hookAllMethods()`进行`hook`，遍历所有方法并判断目标重载方法，实现`hook`：

```java
#文字水印
private void hookCanvasDrawText() {
    for (Method method : Canvas.class.getDeclaredMethods()) {
        if (method.getName().equals("drawText")
            && method.getParameterTypes().length == 4
            && method.getParameterTypes()[0] == String.class
            && method.getParameterTypes()[1] == float.class
            && method.getParameterTypes()[2] == float.class
            && method.getParameterTypes()[3] == Paint.class) {

            XposedBridge.hookMethod(method, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    String text = (String) param.args[0];
                    float x = (float) param.args[1];
                    float y = (float) param.args[2];
                    XposedBridge.log("-  drawText called with text: " + text + " at (" + x + ", " + y + ")");
                    if (text != null && (text.toLowerCase().contains("小红书") || text.toLowerCase().contains("xhs"))) {
                        param.args[0] = "";
                        XposedBridge.log(">>> Watermark text cleared.");
                    }
                }
            });
        }
    }
}

#logo水印
private void hookCanvasDrawBitmap() {
    for (Method method : Canvas.class.getDeclaredMethods()) {
        if (method.getName().equals("drawBitmap")
            && method.getParameterTypes().length == 4
            && method.getParameterTypes()[0].getName().equals("android.graphics.Bitmap")
            && method.getParameterTypes()[1].getName().equals("android.graphics.Rect")
            && method.getParameterTypes()[2].getName().equals("android.graphics.Rect")
            && method.getParameterTypes()[3].getName().equals("android.graphics.Paint")) {

            XposedBridge.hookMethod(method, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    Bitmap bmp = (Bitmap) param.args[0];
                    int w = bmp.getWidth();
                    int h = bmp.getHeight();
                    XposedBridge.log("-  drawBitmap size: " + w + "x" + h);

                    if (w >> Replaced suspected watermark bitmap");
                    }
                }
            });
        }
    }
}
```

目前这两种方式判断`bitmap`是否为水印的方式是根据`size`来的，（`width`，可以达到要求。

在已经按照`LSPosed`的手机上，安装地址中的`xposed`模块即可。

[//github.com/SimpsonGet/XhsWatermarkRemove/releases/download/release-v1.0.0/xhsXposed.apk](https://github.com/SimpsonGet/XhsWatermarkRemove/releases/download/release-v1.0.0/xhsXposed.apk:https)

![](https://github.com/SimpsonGet/XhsWatermarkRemove/blob/main/ReadmePicture/xposedOpen.png?raw=true)

附上一张`xhs`下载的无水印猫图。

![](https://github.com/SimpsonGet/XhsWatermarkRemove/blob/main/ReadmePicture/1040g3k031j8o2p4mii505q0ggghji0q77fg497o.jpg?raw=true)
