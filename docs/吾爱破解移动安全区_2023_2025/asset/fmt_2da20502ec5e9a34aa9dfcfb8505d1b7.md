
```java
            if (xc_MethodHook$MethodHookParam.args[0].toString().contains("accessibility")) {
                xc_MethodHook$MethodHookParam.setResult(null);
            }
        }
        else {
            if("getStringForUser".equals(name)) {
                if ("android.provider.Settings$NameValueCache".equals(v1)) {
                    //RLog.m("Test---get--othier:enter android.provider.Settings$NameValueCache222");
                    this.setAccessible(xc_MethodHook$MethodHookParam.thisObject, "mValues");
                    v0_1 = XposedHelpers.getObjectField(xc_MethodHook$MethodHookParam.thisObject, "mValues");
                    if ("android_id".equals(xc_MethodHook$MethodHookParam.args[1])) {
                        ((HashMap) v0_1).put("android_id", this.deviceInfoModel.getAndroidId());
                        xc_MethodHook$MethodHookParam.setResult(this.deviceInfoModel.getAndroidId());
                    }
                    if ("accessibility_enabled".equals(xc_MethodHook$MethodHookParam.args[1])) {
                        ((HashMap) v0_1).put("accessibility_enabled", "0");
                        name = "0";
                        xc_MethodHook$MethodHookParam.setResult(name);
                    }
                    if ("enabled_accessibility_services".equals(xc_MethodHook$MethodHookParam.args[1])) {
                        ((HashMap) v0_1).put("enabled_accessibility_services", null);
                        xc_MethodHook$MethodHookParam.setResult(null);
                    }
                    super.afterHookedMethod(xc_MethodHook$MethodHookParam);
                }
                if ("android_id".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    xc_MethodHook$MethodHookParam.setResult((Object)this.deviceInfoModel.getAndroidId());
                }
                if ("adb_enabled".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    xc_MethodHook$MethodHookParam.setResult((Object) 0);
                }
                if ("data_roaming".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    xc_MethodHook$MethodHookParam.setResult((Object) 0);
                }
                if ("bluetooth_address".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    if (TextUtils.isEmpty(this.deviceInfoModel.getBtAddress())) {
                        super.afterHookedMethod(xc_MethodHook$MethodHookParam);
                        xc_MethodHook$MethodHookParam.setResult(this.deviceInfoModel.getBtAddress());
                    }
                }
                if ("bluetooth_name".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    if (TextUtils.isEmpty(this.deviceInfoModel.getBtName())) {
                        super.afterHookedMethod(xc_MethodHook$MethodHookParam);
                        xc_MethodHook$MethodHookParam.setResult(this.deviceInfoModel.getBtName());
                    }
                }
                if ("accessibility_enabled".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    xc_MethodHook$MethodHookParam.setResult(0);
                }
                if ("enabled_accessibility_services".equals(xc_MethodHook$MethodHookParam.args[1])) {
                    xc_MethodHook$MethodHookParam.setResult(null);
                    super.afterHookedMethod(xc_MethodHook$MethodHookParam);
                }
                if (xc_MethodHook$MethodHookParam.args[1].toString().contains("accessibility")) {
                    xc_MethodHook$MethodHookParam.setResult(null);
                }
            }
            if("set".equals(name)) {
                Log.i("SysPropertyHook", "SystemProperties set : " + xc_MethodHook$MethodHookParam.args[0] + " , " + xc_MethodHook$MethodHookParam.args[1]);
            }
            if(!"getInt".equals(name)) {
            }
            if("ro.build.version.sdk".equals(xc_MethodHook$MethodHookParam.args[0])) {
                xc_MethodHook$MethodHookParam.setResult(this.deviceInfoModel.getBuildSdk());
            }
        }
    }

    @Override
   protected void beforeHookedMethod(XC_MethodHook.MethodHookParam paramMethodHookParam) throws Throwable {
        super.beforeHookedMethod(paramMethodHookParam);
        String str1 = paramMethodHookParam.method.getName();
        Class localClass = paramMethodHookParam.method.getDeclaringClass();
        if ("putStringForUser".equals(str1)) {
            Class[] arrayOfClass = new Class[3];
            arrayOfClass[0] = ContentResolver.class;
            arrayOfClass[1] = String.class;
            arrayOfClass[2] = Integer.TYPE;
            Method localMethod = localClass.getDeclaredMethod("getStringForUser", arrayOfClass);
            Object[] arrayOfObject = new Object[3];
            arrayOfObject[0] = paramMethodHookParam.args[0];
            arrayOfObject[1] = paramMethodHookParam.args[1];
            arrayOfObject[2] = paramMethodHookParam.args[3];
            String str2 = (String) localMethod.invoke(localClass, arrayOfObject);
            String str3 = localClass.getSimpleName() + "," + paramMethodHookParam.args[1] + "," + paramMethodHookParam.args[2] + "," + str2;
            Log.i("SysPropertyHook"," Settings putStringForUser : "+str3);
        }else if("get".equals(str1)){
            setBuilds();
        }
    }
    }
```

## 3. `RobuildHook.java`

这里会通过`Build`类的`getString()`获取到一些系统属性的值。

[Java] 纯文本查看 复制代码
