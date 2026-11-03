
`setRootDir()` //设置虚拟机的根目录，可以实现 IO 重定向，例如：app 读取 `/data/data/123.txt`，`rootDir` 设置为 `E:/unidbg`，那么真正的目录是 `E:/unidbg/data/data/123.txt`，

## 1. emulator 常用 Api

| 方法名 | 返回类型 | 描述 |
|---|---|---|
| `getMemory()` | `Memory` | 获取内存操作接口。 |
| `getPid()` | `int` | 获取进程的 PID。 |
| `createDalvikVM()` | `VM` | 创建虚拟机。 |
| `createDalvikVM(File apkFile)` | `VM` | 创建虚拟机并指定 APK 文件路径。 |
| `getDalvikVM()` | `VM` | 获取已创建的虚拟机。 |
| `showRegs()` | `void` | 显示当前寄存器状态，可指定寄存器。 |
| `getBackend()` | `Backend` | 获取后端 CPU。 |
| `getProcessName()` | `String` | 获取进程名。 |
| `getContext()` | `RegisterContext` | 获取寄存器上下文。 |
| `traceRead(long begin, long end)` | `void` | Trace 读内存操作。 |
| `traceWrite(long begin, long end)` | `void` | Trace 写内存操作。 |
| `traceCode(long begin, long end)` | `void` | Trace 汇编指令执行。 |
| `isRunning()` | `boolean` | 判断当前 `Emulator` 是否正在运行。 |

```java
LibraryResolver resolver = new AndroidResolver(23);
memory.setLibraryResolver(resolver);
```

`23` 和 `19` 分别对应于 `sdk23`（Android 6.0） 和 `sdk19`（Android 4.4）的运行库环境，处理 64 位 SO 时只能选择 SDK23。

![](https://pic.rmb.bdstatic.com/bjh/241120/6928ce9343c8d818db850a9a6f510f9a774.png)

## 2. memory 常用 Api

| 方法名 | 返回类型 | 描述 |
|---|---|---|
| `setLibraryResolver(AndroidResolver resolver)` | `void` | 设置 Android SDK 版本解析器，目前支持 19 和 23 两个版本。 |
| `getStackPoint()` | `long` | 获取当前栈指针的值。 |
| `pointer(long address)` | `UnidbgPointer` | 获取指针，指向指定内存地址，可通过指针操作内存。 |
| `getMemoryMap()` | `Collection` | 获取当前内存的映射情况。 |
| `findModule(String moduleName)` | `Module` | 根据模块名获取指定模块。 |
| `findModuleByAddress(long address)` | `Module` | 根据地址获取指定模块。 |
| `loadLibrary(File file, boolean forceLoad)` | `ElfModule` | 加载 SO 文件，会调用 `Linker.do_dlopen()` 方法完成加载。 |
| `allocatestack(int size)` | `UnidbgPointer` | 在栈上分配指定大小的内存空间。 |
| `writestackstring(String value)` | `UnidbgPointer` | 将字符串写入栈内存中。 |
| `writestackBytes(byte[] value)` | `UnidbgPointer` | 将字节数组写入栈内存中。 |
| `malloc(int size, boolean runtime)` | `UnidbgPointer` | 分配指定大小的内存空间，返回指向该内存的指针。 |

## 3. VM 常用 Api

```java
// 创建 Dalvik 虚拟机实例
vm = emulator.createDalvikVM();
// 设置是否输出详细的 JNI 调用日志
vm.setVerbose(false);
// 加载指定路径的 SO 库文件，不自动调用 JNI_OnLoad 函数
DalvikModule dm = vm.loadLibrary(new File("unidbg-android/src/test/resources/example_binaries/armeabi-v7a/libnative-lib.so"), false);
// 手动调用 JNI_OnLoad 方法
dm.callJNI_OnLoad(emulator);
```

| 方法名 | 返回类型 | 描述 |
|---|---|---|
| `createDalvikVM(File apkFile)` | `VM` | 创建虚拟机，指定 APK 文件，file 可为空 |
| `setVerbose(boolean verbose)` | `void` | 设置是否输出 JNI 运行日志。 |
| `loadLibrary(File soFile, boolean callInit)` | `DalvikModule` | 加载 SO 模块，参数二设置是否自动调用 init 函数。 |
| `setJni(Jni jni)` | `void` | 设置 JNI 交互接口，推荐实现 `AbstractJni`。 |
| `getJNIEnv()` | `Pointer` | 获取 `JNIEnv` 指针，可作为参数传递。 |
| `getJavaVM()` | `Pointer` | 获取 `JavaVM` 指针，可作为参数传递。 |
| `callJNI_OnLoad(Emulator emulator, Module module)` | `void` | 调用 `JNI_OnLoad` 函数。 |
| `addGlobalObject(DvmObject obj)` | `int` | 向 VM 添加全局对象，返回该对象的 hash 值。 |
| `addLocalObject(DvmObject obj)` | `int` | 向 VM 添加局部对象，返回该对象的 hash 值。 |
| `getObject(int hash)` | `DvmObject` | 根据 hash 值获取虚拟机中的对象。 |
| `resolveClass(String className)` | `DvmClass` | 解析指定类名，构建并返回一个 `DvmClass` 对象。 |
| `getPackageName()` | `String` | 获取 APK 包名。 |
| `getVersionName()` | `String` | 获取 APK 版本名称。 |
| `getVersionCode()` | `String` | 获取 APK 版本号。 |
| `openAsset(String assetName)` | `InputStream` | 打开 APK 中的指定资源文件。 |
| `getManifestXml()` | `String` | 获取 `AndroidManifest.xml` 文件的文本内容。 |
| `getSignatures()` | `CertificateMeta[]` | 获取 APK 签名信息。 |
| `findClass(String className)` | `DvmClass` | 通过类名获取已经加载的类（`DvmClass` 对象）。 |
| `getEmulator()` | `Emulator` | 获取模拟器对象 `emulator`。 |

1. `VM dalvikVM = emulator.createDalvikVM(new File("apk file path"))` - 创建虚拟机并指定 APK 文件，加载指定 APK 文件，`unidbg` 可以帮我们完成一些小操作，例如：解析 Apk 基本信息，Apk 的版本名、版本号、包名、Apk 签名等信息，减少补环境操作；解析和管理 Apk 资源文件，加载 Apk 后可以通过 `openAsset` 获取 APK assets 目录下的文件。

2. `loadLibrary` 三个重载方法

```java
/**
 * 加载指定名称的库文件。
 * @Param libname 库文件的名称，不包括前缀 "lib" 和后缀 ".so"（例如 "example" 对应 "libexample.so"）。
 * @param forceCallInit 是否强制调用库的初始化函数（如 JNI_OnLoad）。
 * @Return 加载后的 DalvikModule 对象，封装了加载的库模块。
 */
DalvikModule loadLibrary(String libname, boolean forceCallInit);

/**
 * 从原始字节数组中加载指定的库文件。
 * @param libname 库文件的名称，仅用于标识该库，与文件路径无关。
 * @param 传入 buffer 方便解析 elf
 */
DalvikModule loadLibrary(String libname, byte[] raw, boolean forceCallInit);

/**
 * 从指定路径的加载 ELF。
 * @param elfFile 表示库的 ELF 文件，必须是有效的 ELF 格式文件。例如:new File("unidbg-android/src/test/resources/example_binaries/armeabi-v7a/libnative-lib.so")
 * @param forceCallInit 是否强制调用库的初始化函数（如 JNI_OnLoad）。
 */
DalvikModule loadLibrary(File elfFile, boolean forceCallInit);
```

## 4. JNI 函数的调用

![](https://pic.rmb.bdstatic.com/bjh/241127/42f64fbf7955fee4e576103830d0b9362530.png)

```java
// 第一个参数传入模拟器实例
// 第二个参数传入要调用的函数在 java 层方法签名信息
// 第三个参数传入可变参数列表，这里的参数在旧版本有很多需要自己封装，新版则帮我们封装好了，详细看 callJniMethod 方法
boolean success = obj.callJniMethodBoolean(emulator, "jnitest(Ljava/lang/String;)Z", str);
```

```java
/**
 * 调用 JNI 方法的辅助函数。
 *
 * @param emulator   模拟器实例。
 * @param vm         Dalvik 虚拟机实例。
 * @param objectType 调用方法的类（DvmClass）。
 * @param thisObj    方法调用的对象实例（DvmObject）。
 * @param method     要调用的方法签名（例如 "methodName(参数类型)返回类型"）。
 * @param args       方法的可变参数列表。
 * @return           方法执行后的返回值（Number 类型），可能是整数或浮点数。
 */
protected static Number callJniMethod(Emulator emulator, VM vm, DvmClass objectType, DvmObject thisObj, String method, Object... args) {
    // 查找对应的本地函数指针（Native 方法）
    UnidbgPointer fnPtr = objectType.findNativeFunction(emulator, method);
    // 将当前对象添加到本地引用表，防止被垃圾回收
    vm.addLocalObject(thisObj);
    // 创建用于存放函数参数的列表，初始容量为10
    List list = new ArrayList<>(10);
    // 添加 JNI 环境指针（JNIEnv*）
    list.add(vm.getJNIEnv());
    // 添加 this 对象的引用（jobject）
    list.add(thisObj.hashCode());
    // 处理传入的参数列表
    if (args != null) {
        for (Object arg : args) {
            if (arg instanceof Boolean) {
                // 如果参数是布尔值，转换为 JNI_TRUE 或 JNI_FALSE
                list.add((Boolean) arg ? VM.JNI_TRUE : VM.JNI_FALSE);
                continue;
            } else if (arg instanceof Hashable) {
                // 如果参数实现了 Hashable 接口，表示是 DvmObject 或其子类
                list.add(arg.hashCode()); // 添加对象引用（jobject）
                if (arg instanceof DvmObject) {
                    // 将 DvmObject 对象添加到本地引用表
                    vm.addLocalObject((DvmObject) arg);
                }
                continue;
            } else if (arg instanceof DvmAwareObject ||
                    arg instanceof String ||
                    arg instanceof byte[] ||
                    arg instanceof short[] ||
                    arg instanceof int[] ||
                    arg instanceof float[] ||
                    arg instanceof double[] ||
                    arg instanceof Enum) {
                // 如果参数是 DvmAwareObject、字符串、数组或枚举等类型
                // 创建一个代理 DvmObject 对象
                DvmObject obj = ProxyDvmObject.createObject(vm, arg);
                // 添加对象引用（jobject）
                list.add(obj.hashCode());
                // 将对象添加到本地引用表
                vm.addLocalObject(obj);
                continue;
            }

            // 对于其他类型的参数，直接添加到参数列表
            list.add(arg);
        }
    }
    // 调用本地函数，传入参数列表，并返回结果
    return Module.emulateFunction(emulator, fnPtr.peer, list.toArray());
}
```

基本类型直接传递，`int`、`long`、`boolean`、`double` 等。

下面几种对象类型 `unidbg` 也帮我们封装好了

- `String`
- `byte` 数组
- `short` 数组
- `int` 数组
- `float` 数组
- `double` 数组
- `Enum` 枚举类型

## 5. 特殊参数构造

对于其他数据类型需要借助 `resolveClass` 构造，例如 `Context`

```java
DvmObject context = vm.resolveClass("android/content/Context").newObject(null);
```

## 6. 符号调用与偏移调用

### 符号调用
