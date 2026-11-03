
## `Context.checkSelfPermission`

```java
@Override
public int callIntMethod(BaseVM vm, DvmObject dvmObject, String signature, VarArg varArg) {
    switch (signature) {
        case "android/content/Context->checkSelfPermission(Ljava/lang/String;)I": {
            System.out.println("[+]Context.checkSelfPermission arg0-->>" + varArg.getObjectArg(0));
            return 1;
        }
    }
    return super.callIntMethod(vm, dvmObject, signature, varArg);
}
```

- 看下一个错误；

```text
java.lang.UnsupportedOperationException: android/content/Context->getSystemService(Ljava/lang/String;)Ljava/lang/Object;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:869)
```

## `Context.getSystemService`

- 这个api是获取系统服务相关，应用程序通过 `Context` 对象的 `getSystemService` 方法，传入一个表示所需服务的字符串常量；系统返回对应服务的代理对象，应用可以通过这个对象与系统服务交互。

```java
@Override
public DvmObject callObjectMethod(BaseVM vm, DvmObject dvmObject, String signature, VarArg varArg) {
    switch (signature) {
        case "android/content/Context->getSystemService(Ljava/lang/String;)Ljava/lang/Object;": {
            String serviceName = varArg.getObjectArg(0).getValue().toString();
            System.out.println("getSystemService->" + serviceName);
        }
    }
    return super.callObjectMethod(vm, dvmObject, signature, varArg);
}
```

- 他会告诉你需要哪个服务；

`getSystemService->phone`

- 此时同时跟随着报错；

```text
java.lang.UnsupportedOperationException: android/content/Context->getSystemService(Ljava/lang/String;)Ljava/lang/Object;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:179)
```

- 这里需要结合起来看，他要返回的就是`phone`相关的服务，后续再对这个服务做一些操作；

```java
case "android/content/Context->getSystemService(Ljava/lang/String;)Ljava/lang/Object;": {
    // 获取服务名称
    String serviceName = varArg.getObjectArg(0).getValue().toString();
    System.out.println("getSystemService->" + serviceName);
    switch (serviceName) {
        // 根据服务名称返回不同的对象
        case "phone": {
            // 返回TelephonyManager对象
            return vm.resolveClass("android/telephony/TelephonyManager").newObject(null);
        }
    }
}
```

## `TelephonyManager` 相关方法

- 继续看错误；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getSimState()I
        at com.github.unidbg.linux.android.dvm.AbstractJni.callIntMethod(AbstractJni.java:967)
        at com.Samples.pdd.anti_token_64.callIntMethod(anti_token_64.java:136)
```

- 这个方法是为了获取当前sim卡的状态的，返回5吧，表示就绪状态，注意位置不要补错了；

```java
case "android/telephony/TelephonyManager->getSimState()I": {
    return 5;
}
```

- 继续跑；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getSimOperatorName()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:186)
```

- 这个方法是获取运营商名称的，随意即可；

```java
case "android/telephony/TelephonyManager->getSimOperatorName()Ljava/lang/String;": {
    return new StringObject(vm, "中国移动");
}
```

- 这里给空字符串也行吧，不过和前面一致比较好，继续看；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getSimCountryIso()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:182)
```

- 这个是返回国家相关的简称，按照这样返回；

```java
case "android/telephony/TelephonyManager->getSimCountryIso()Ljava/lang/String;": {
    return new StringObject(vm, "cn");
}
```

- 继续跑；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getNetworkType()I
        at com.github.unidbg.linux.android.dvm.AbstractJni.callIntMethod(AbstractJni.java:967)
        at com.Samples.pdd.anti_token_64.callIntMethod(anti_token_64.java:136)
```

- 这部分几乎都是这种类型的，后续的我不再进行解释了，因为我遇到不会的也是google，但不要百度就好；

```java
case "android/telephony/TelephonyManager->getNetworkType()I": {
    return 2;
}
```

- 继续；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getNetworkOperator()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:183)
```

- 这个是获取运营商代号的，这里和前面对应上；

```java
case "android/telephony/TelephonyManager->getNetworkOperator()Ljava/lang/String;": {
    return new StringObject(vm, "46000");
}
```

- 继续执行；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getNetworkOperatorName()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:184)
```

- 这个也是类似的，只不过获取的是人类可读的名称；

```java
case "android/telephony/TelephonyManager->getNetworkOperatorName()Ljava/lang/String;": {
    return new StringObject(vm, "中国移动");
}
```

- 继续执行；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getNetworkCountryIso()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callObjectMethod(AbstractJni.java:935)
        at com.Samples.pdd.anti_token_64.callObjectMethod(anti_token_64.java:185)
```

- 这个一看就和城市有关；

```java
case "android/telephony/TelephonyManager->getNetworkCountryIso()Ljava/lang/String;": {
    return new StringObject(vm, "cn");
}
```

- 继续看下一个；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getDataState()I
        at com.github.unidbg.linux.android.dvm.AbstractJni.callIntMethod(AbstractJni.java:967)
        at com.Samples.pdd.anti_token_64.callIntMethod(anti_token_64.java:136)
```

- 这个方法是获取连接状态的，直接返回；

```java
case "android/telephony/TelephonyManager->getDataState()I": {
    return 0;
}
```

- 返回1应该也没事，继续看下一个；

```text
java.lang.UnsupportedOperationException: android/telephony/TelephonyManager->getDataActivity()I
        at com.github.unidbg.linux.android.dvm.AbstractJni.callIntMethod(AbstractJni.java:967)
        at com.Samples.pdd.anti_token_64.callIntMethod(anti_token_64.java:136)
```

- 这个api的含义是用于获取当前移动数据连接的活动状态，表示数据正在如何传输；都是一个系列的；

```java
case "android/telephony/TelephonyManager->getDataActivity()I": {
    return 0;
}
```

- 到这里前面这一系列基本上补完了，开始新的错误了；

## 后续补充方法

```text
java.lang.UnsupportedOperationException: com/xunmeng/pinduoduo/secure/EU->gad()Ljava/lang/String;
        at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticObjectMethodV(AbstractJni.java:506)
        at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticObjectMethodV(AbstractJni.java:440)
```

- 这个很明显是样本自己的方法，去hook看看返回值，这里触发时机随意找，最好先别登录账号；

```java
@Override
public DvmObject callStaticObjectMethodV(BaseVM vm, DvmClass dvmClass, String signature, VaList vaList) {
    switch (signature) {
        case "com/xunmeng/pinduoduo/secure/EU->gad()Ljava/lang/String;":
            return new StringObject(vm, "3512d332db67860a");
    }
    return super.callStaticObjectMethodV(vm, dvmClass, signature, vaList);
}
```

- 继续执行；

```text
java.lang.UnsupportedOperationException: android/os/Debug->isDebuggerConnected()Z
        at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticBooleanMethod(AbstractJni.java:181)
        at com.github.unidbg.linux.android.dvm.AbstractJni.callStaticBooleanMethod(AbstractJni.java:176)
```

- 这个是判断是否被debugger，肯定返回`false`嘛；

```java
@Override
public boolean callStaticBooleanMethod(BaseVM vm, DvmClass dvmClass, String signature, VarArg varArg) {
    switch (signature) {
        case "android/os/Debug->isDebuggerConnected()Z":
            return false;
    }
    return super.callStaticBooleanMethod(vm, dvmClass, signature, varArg);
}
```

- 继续执行就会遇到这次样本的一个小难点；
