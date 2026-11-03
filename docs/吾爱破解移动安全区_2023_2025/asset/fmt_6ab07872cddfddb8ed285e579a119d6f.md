
# [Android 原创] 《安卓逆向这档事》第二十五课、Unidbg之补完环境我就睡(中)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2063957
- 抓取时间: 2026-11-01T13:58:06.422Z

![](https://pic.rmb.bdstatic.com/bjh/241224/a5a71662ce7a80f210e94738b801fcf09656.png)

## 一、课程目标

- 理解文件访问补环境的 `IOResolver` 机制，并掌握不同 `FileIO` 的应用场景
- 掌握 `/proc/self/maps`、`/proc/self/status` 等关键文件的核心处理策略
- 理解补齐库函数的原理，掌握使用 `Dobby` Hook `__system_property_get` 等底层函数以伪造系统属性的方法

## 二、工具

1. 教程 Demo
2. `IDEA`
3. `IDA`

## 三、课程内容

### 一. 补文件访问

#### 一、文件访问补环境的核心概念

文件访问 (`File Access`) 补环境是 Unidbg 应用中仅次于 JNI 补环境的重要环节。当 Unidbg 模拟执行的原生库（`.So` 文件）尝试通过文件 I/O 操作访问文件系统时——例如读取设备信息 (`/proc/cpuinfo`)、检测运行环境 (`/proc/self/maps`、`/xbin/su`) 或校验自身完整性 (`APK` 文件)——Unidbg 必须能够对这些访问请求作出响应。在一个纯净的 Unidbg 环境中，这些被访问的目标文件通常是不存在的。如果一个关键的文件访问失败或返回了不符合预期的内容，原生库可能会改变其执行逻辑、触发风控机制或直接中断执行。因此，文件访问补环境的目的就是通过 Unidbg 的 `IOResolver` 责任链机制拦截这些文件系统调用，并提供一个内容、权限都符合原生库逻辑预期的虚拟文件或目录。

#### 二、核心机制：责任链模式与 `IOResolver`

Unidbg 并未将所有文件处理逻辑写死，而是采用了一种灵活的责任链模式。当代码尝试访问一个文件时，请求会依次通过一个处理器链，直到被成功处理或最终失败。

这个处理链的顺序通常是：

- 用户自定义的 `IOResolver`：你添加的文件处理器，后添加的拥有更高优先级。
- `AndroidResolver`：Unidbg 内置的默认处理器。
- 虚拟文件系统（VFS）：处理映射到文件系统的真实文件。

这样的设计使得我们可以注入自定义逻辑来处理文件访问，而无需修改 Unidbg 源码，保证了项目的可移植性和环境的整洁性。

![](https://attach.52pojie.cn/forum/202510/03/204449pwmdt1t68t87ktef.png)

添加自定义文件处理器:

你可以通过 `emulator.getSyscallHandler().addIOResolver()` 来添加一个文件处理器。这个操作必须在加载 SO 库（`loadLibrary`）之前完成才能生效。

```
// BiliIOResolver.java - 独立的处理器类
public class BiliIOResolver implements IOResolver {
    @Override
    public FileResult resolve(Emulator emulator, String pathname, int oflags) {        
        // 打印所有文件访问请求，无论是否处理
        System.out.println("File open request: " + pathname);
        // 在这里添加具体的文件处理逻辑
        return null; // 返回null表示未处理，交由下一个处理器
    }
}
// Main.java - 在主类中添加处理器
public class Bili extends AbstractJni {
    public Bili() {
        // ...模拟器初始化代码...
        emulator.getSyscallHandler().addIOResolver(new BiliIOResolver());
        // ...加载SO和后续操作...
    }
}
```

### 三、文件处理的三种返回结果

在 `resolve` 方法中，你可以返回一个 `FileResult` 对象，它有三种状态：

- `FileResult.success(FileIO)`: 表示文件访问成功，返回一个 `FileIO` 对象。这是最常用的方式。
- `FileResult.failed(errno)`: 表示文件访问失败，并返回一个指定的错误码，如 `UnixEmulator.EACCES` (权限不足)。
- `FileResult.fallback()`: 表示回退，只有当其他所有处理器都无法处理时，才会由它来处理。

### 四、常规文件处理与 `FileIO` 对象

处理文件访问需要结合三方面的知识：Linux/Android 文件系统知识、对业务（风控/检测）的经验以及 Unidbg 的实现方法。

#### 1. 使用 `SimpleFileIO` 处理物理文件

当需要返回一个真实存在的文件时，`SimpleFileIO` 是最佳选择。你需要将文件从真机上导出，并放置在你的项目路径下。

案例：补 `boot_id` 和 `APK` 文件

`boot_id` 常用于生成设备指纹，而 `APK` 文件访问常用于签名校验或资源读取，两者都必须处理。

`boot_id` 文件在开机时生成，在设备关机前不会改变内容。我们将这个文件从真机上 `push` 出来

```
C:\Users\zhengji>adb shell

* daemon not running; starting now at tcp:5037

* daemon started successfully

vermeer:/ $ cat /proc/sys/kernel/random/boot_id

c7de54b2-f238-481d-b8e1-41c05413b2cd

vermeer:/ $ cp /proc/sys/kernel/random/boot_id /sdcard

vermeer:/ $ exit

C:\Users\zhengji>adb pull /sdcard/boot_id D:\unidbg-master\unidbg-android\src\test\resources

/sdcard/boot_id: 1 file pulled, 0 skipped. 0.0 MB/s (37 bytes in 0.012s)
```
```
@Override
public FileResult resolve(Emulator emulator, String pathname, int oflags) {
    switch (pathname) {
        // 注意：这里的路径需要和样本访问的完全一致
        case "/proc/sys/kernel/random/boot_id":{
                return FileResult.success(new SimpleFileIO(oflags, new File("unidbg-android/src/test/resources/cpu/boot_id"), pathname));
            }
    }
    return null;
}
```

#### 2. 使用 `ByteArrayFileIO` 动态生成文件内容

当文件内容需要动态生成或随机化时（例如躲避基于设备指纹的风控），`ByteArrayFileIO` 非常有用。它直接接收一个字节数组作为文件内容。

案例：随机化 `boot_id` 和补 CPU 频率

```
@Override
public FileResult resolve(Emulator emulator, String pathname, int oflags) {
    switch (pathname) {
        case "/proc/sys/kernel/random/boot_id": {
            // 动态生成UUID作为boot_id
            String randomBootId = UUID.randomUUID().toString();
            return FileResult.success(new ByteArrayFileIO(oflags, pathname, randomBootId.getBytes(StandardCharsets.UTF_8)));
        }
        case "/sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_max_freq": {
            // 直接返回一个字符串作为文件内容
            return FileResult.success(new ByteArrayFileIO(oflags, pathname, "1766400".getBytes()));
        }
    }
    return null;
}
```

注意：`ByteArrayFileIO` 不支持写入操作，如果样本需要写入文件，使用它会抛出 `UnsupportedOperationException`。

#### 3. 使用 `DirectoryFileIO` 和虚拟文件系统处理目录

- `DirectoryFileIO`: 当样本明确访问一个目录时，可以使用它，并传入一个本地文件夹。
- 虚拟文件系统（VFS）: 这是处理目录访问更好的选择。你可以在初始化模拟器时设置一个根目录，然后将需要的文件按真实路径结构放置其中。这对于样本需要遍历目录（如 `/system/fonts`）的场景非常高效。

#### 4. 处理失败的文件访问

多数情况下，对于环境检测类的文件（如 `su` 文件），我们什么都不做，让它自然失败即可。但在某些特殊场景，需要手动返回失败。

案例：模拟无 Root 权限

有些 Root 检测会尝试访问 `/data` 目录的权限。由于 Unidbg 的虚拟文件系统会自动创建 `/data` 目录导致访问成功，我们需要手动拦截并返回“权限不足”。

```
@Override
public FileResult resolve(Emulator emulator, String pathname, int oflags) {
if ("/data".equals(pathname)) {
return FileResult.failed(UnixEmulator.EACCES); // EACCES = 13, Permission denied
}
return null;
}
```

### 五、关键点

文件路径 (`Path`) | 
主要用途 (`Purpose`) | 
推荐处理策略 (`Recommended Strategy`) | 
注意事项 (`Key Points`) | 

`/proc/self/cmdline` | 
获取当前进程名 | 
必须补。使用 `ByteArrayFileIO` 动态生成。 | 
路径中的 pid 需用 `emulator.getPid()` 动态获取；内容末尾必须包含 `\0`。 | 

`/proc/self/status` | 
获取进程状态，检测 `TracerPid` | 
必须补。使用 `ByteArrayFileIO` 动态生成。 | 
`TracerPid` 必须为 0；`Pid` 和 `Tgid` 字段应动态替换为 `emulator.getPid()`。 | 

`/proc/self/maps` | 
获取内存映射，检测 `frida` 等 | 
策略性处理：  
1. 优先：用真实 `maps` 文件 (`SimpleFileIO`)。
2. 备选：若发生内存错误，则不处理，退回使用 Unidbg 默认的 `fakeMaps`。 | 
真实 `maps` 可能导致内存地址访问异常；`fakeMaps` 缺少 `APK`等映射信息。 | 

`/data/app/.../base.apk` | 
签名校验、资源读取 | 
必须补。使用 `SimpleFileIO` 提供真实的 `APK` 文件。 | 
路径在不同设备上可能不同，注意日志中的实际访问路径。 | 

`/proc/sys/kernel/random/boot_id` | 
生成设备指纹 | 
建议补。用 `SimpleFileIO` (固定) 或 `ByteArrayFileIO` (随机化)。 | 
有些风控会记录 `boot_id`，随机化可避免关联。 | 

`/sys/.../cpuinfo_max_freq` | 
读取设备硬件信息 | 
建议补。使用 `ByteArrayFileIO` 提供一个常见值。 | 
直接返回一个字符串即可，如 `"2867200"`。 | 

`/xbin/su`, `/system/bin/su` | 
Root 环境检测 | 
无需处理或显式失败。让其自然失败 (返回 `null`) 或通过 `stat` 返回 `ENOENT`。 | 
不要提供一个假文件，让文件访问失败是最真实的无 Root模拟。 | 

`/proc/net/tcp`, `/proc/net/udp` | 
检测代理、抓包工具 | 
强烈建议不补。 | 
高版本 Android 已禁止访问。不补能模拟高版本系统行为，且避免暴露主机网络信息。 | 

`/proc` 目录下的文件访问频率极高，且容易出错，需要特殊对待。

#### 1. `/proc/self/cmdline`

- 用途：获取当前进程名，常用于反调试或环境检测。
- 处理要点：
  - 必须补，否则可能导致关键逻辑失败。
  - 由于进程 ID (`pid`) 每次运行都可能变化，硬编码 `/proc/12345/cmdline` 会失效。因此，最佳实践是直接拦截符号链接路径 `/proc/self/cmdline`，这样可以无需关心动态的 `pid`。
  - 返回的进程名字符串末尾应包含一个 `\0` (`NULL`) 字符，这是 `cmdline` 的格式规范，不加的话在解析时存在出错的可能性。

```
case "/proc/self/cmdline":   
return FileResult.success(new ByteArrayFileIO(oflags, pathname, PACKAGE_NAME.getBytes()));
```

#### 2. `/proc/self/status`

- 用途：获取进程状态，尤其是 `TracerPid` 字段，用于检测是否被调试（`gdb`/`ida`）。
- 处理要点：
  - 最简单的处理是只返回 `TracerPid: 0`。
  - 更稳妥的方式是返回一个从真机 `status` 文件中复制的、内容完整模板。
