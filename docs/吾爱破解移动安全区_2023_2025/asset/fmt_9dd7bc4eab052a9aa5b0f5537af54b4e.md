
```
if(!loadPackageParam.packageName.equals("com.zj.wuaipojie")){
    return ;
}
Class a = loadPackageParam.classLoader.loadClass("com.zj.wuaipojie.Demo");
XposedBridge.hookAllMethods(a,"repleaceFunc",new XC_MethodReplacement() {
    @Override
    protected Object replaceHookedMethod(MethodHookParam methodHookParam) throws Throwable {
        return "";
    }
});
    }
}
```
这里需要使用 `XC_MethodReplacement` 并实现 `replaceHookedMethod` 接口，直接给一个 `return ""` 即可。

![](https://xia0ji233.pro/2025/11/16/Android1/11.png)

可以看到原本应该输出的 这是替换函数 没有了。

## Hook 变量的技巧

在说这个之前，区分一下两种类型的变量，一种是静态的，它只有一份，一种是非静态的，它随具体实例。

比如人这个类，数量就是对于整个人类而言的，它只有一个，所以通常来说它的定义是静态的变量。

再比如身高对人类而言，每个人都有一个确定的身高，所以通常来说它只能定义为非静态变量。

### Hook 静态变量

使用 `setStaticObjectField` 方法即可。

以图中的静态方法为例

![](https://xia0ji233.pro/2025/11/16/Android1/12.png)

直接调用方法修改即可

```
package com.xiaoji.xposeddemo;
import android.util.Log;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class Hook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if(!loadPackageParam.packageName.equals("com.zj.wuaipojie")){
            return ;
        }

        final Class clazz = XposedHelpers.findClass("com.zj.wuaipojie.Demo", loadPackageParam.classLoader);
        XposedHelpers.setStaticObjectField(clazz, "staticField", "Hooked Static Field Value by xia0ji233");
    }
}
```

结果也是成功修改：

![](https://xia0ji233.pro/2025/11/16/Android1/13.png)

### Hook 非静态变量

直接看代码和结果吧，主要使用 `hookAllConstructors` 方法去做

```
package com.xiaoji.xposeddemo;

import android.util.Log;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class Hook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if(!loadPackageParam.packageName.equals("com.zj.wuaipojie")){
            return ;
        }

        final Class clazz = XposedHelpers.findClass("com.zj.wuaipojie.Demo", loadPackageParam.classLoader);
        XposedBridge.hookAllConstructors(clazz, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                super.afterHookedMethod(param);
                //param.thisObject获取当前所属的对象
                Object ob = param.thisObject;
                XposedHelpers.setIntField(ob,"privateInt",114514);
            }
        });
    }
}
```

这里 hook 该类所有的构造函数，并在构造函数结束之后（实现 `afterHookedMethod` 方法）对成员的属性进行修改，成功修改实例的成员值。

![](https://xia0ji233.pro/2025/11/16/Android1/14.png)

### Hook 构造函数

这里有参构造和无参构造一起 hook，用的是同一种方法，参数需要自己加才能 hook 到指定构造方法。

```
package com.xiaoji.xposeddemo;
import android.util.Log;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class Hook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if(!loadPackageParam.packageName.equals("com.zj.wuaipojie")){
            return ;
        }

        XposedHelpers.findAndHookConstructor("com.zj.wuaipojie.Demo", loadPackageParam.classLoader, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                super.beforeHookedMethod(param);
                Log.e("zj2595","Demo Constructor Hooked");
            }
        });
        XposedHelpers.findAndHookConstructor("com.zj.wuaipojie.Demo", loadPackageParam.classLoader, String.class, new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                super.beforeHookedMethod(param);
                Log.e("zj2595","Demo(String) Constructor Hooked, param: " + param.args[0]);
            }
        });
    }
}
```

因为无参构造调用了有参构造，所以无参的构造 hook 先输出，再输出有参构造，如果是调用结束后的 hook 输出顺序则会相反。

![](https://xia0ji233.pro/2025/11/16/Android1/16.png)

## 其它类型的 hook

### 主动调用

当需要主动调用某方法时，需要使用方法 `callMethod`，不过这里只能调用非静态方法，因此必须拿到实例才能调用，实例可以自己新建也可以用某些方法拿到。

如果是静态类，那就传类和静态方法名即可。

```
package com.xiaoji.xposeddemo;

import android.util.Log;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class Hook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if(!loadPackageParam.packageName.equals("com.zj.wuaipojie")){
            return ;
        }

        Class clazz = XposedHelpers.findClass("com.zj.wuaipojie.Demo",loadPackageParam.classLoader);
        XposedHelpers.callMethod(clazz.newInstance(),"refl");
    }
}
```

这里是非静态方法的主动调用，如果要调用静态方法，则换成

```
Class clazz = XposedHelpers.findClass("com.zj.wuaipojie.Demo",loadPackageParam.classLoader);
XposedHelpers.callMethod(clazz,"staticFunc");
```

这里展示一下调用非静态方法的结果：

![](https://xia0ji233.pro/2025/11/16/Android1/17.png)

### 反射调用

先介绍一下反射吧（实则我也不懂）

讲反射之前先讲一讲正射，正射是加载类最常见的一种方式。

```
Apple apple = new Apple();
apple.setPrice(4);
```

通过导入包 `apple` 并直接调用构造方法构造类的方法称为正射，正射需要在编译阶段就确定类。

反射则是通过运行时动态加载类。

```
Class clz = Class.forName("com.xiaoji.Apple");
Method method = clz.getMethod("setPrice", int.class);
Constructor constructor = clz.getConstructor();
Object object = constructor.newInstance();
method.invoke(object, 4);
```

反射就是在运行时才知道要操作的类是什么，并且可以在运行时获取类的完整构造，并调用对应的方法。

反射调用

```
package com.xiaoji.xposeddemo;

import android.util.Log;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XC_MethodReplacement;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;

public class Hook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
        if(!loadPackageParam.packageName.equals("com.zj.wuaipojie")){
            return ;
        }

        Class clazz = Class.forName("com.zj.wuaipojie.Demo",false,loadPackageParam.classLoader);
        Method refl = clazz.getDeclaredMethod("refl");
        refl.setAccessible(true);
        refl.invoke(clazz.newInstance());
    }
}
```

成功通过反射调用方法，调用私有方法时，需要使用 `setAccessible` 把权限设置一下。

![](https://xia0ji233.pro/2025/11/16/Android1/18.png)

### Hook 内部类

基本与 hook 普通方法一致，使用 `findAndHookMethod` 方法，在传类名的时候需要使用类似 `com.zj.wuaipojie.Demo$InnerClass` 的写法，用 `$` 区分内部类。

## 后记

对于 Android 而言，我算是迈出了一小步吧，最近的几个月，可能在对新知识的学习上懈怠了不少，Android 对我而言是很早就进规划，但是一直没有动力学习的知识。感谢吾爱破解的正己大神，写出如此完善简洁易懂的教材，让我在学习的路上能够节省大量时间，也能对学习的知识一直保持一定的兴趣。

希望接下来三个月内，可以把已有的知识点学完，等着过年抢吾爱的安卓红包了

![](https://xia0ji233.pro/2025/11/16/Android1/face.jpg)
