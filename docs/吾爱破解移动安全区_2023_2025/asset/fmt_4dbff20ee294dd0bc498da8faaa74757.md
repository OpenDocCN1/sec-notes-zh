
文件存在性检查函数首先尝试访问 `/etc/mumu-configs/device-prop-configs/mumu.config` 文件，若该文件存在，则直接返回 1（判定为 MuMu 模拟器）。若该文件不存在，会依次检查上述 `/data/local/cfg-*` 系列文件和共享文件夹路径，只要其中任意一个文件 / 路径存在，就返回 1；若所有文件 / 路径都不存在，则返回 0。

## `sub_4DE00`
准备模拟器专属系统属性列表函数中涉及的系统属性包括：
- `microvirt.dlhost`（Microvirt 系列模拟器的属性，如 Memu 模拟器）
- `microvirt.memu_version`（Memu 模拟器的版本属性）
- `persist.nox.model`（夜神模拟器的设备型号属性）

系统属性存在性检查函数通过 `_system_property_get_E1DC0` 函数依次检查上述系统属性是否存在。只要其中任意一个系统属性能被成功获取（返回值大于等于 1），则返回 1（判定为目标模拟器）；若所有属性都不存在，则返回 0。

## 一、Magisk 相关核心文件
| 路径 | 说明 |
| --- | --- |
| `/data/magisk/resetprop` | Magisk 用于修改系统属性的工具文件 |
| `/dev/.magisk.unblock` | Magisk 相关的设备文件，用于权限控制 |
| `/sbin/.magisk/` | Magisk 核心组件的挂载路径 |
| `/sbin/.core/mirror` | Magisk 镜像挂载路径，用于系统分区修改 |
| `/sbin/.core/img` | Magisk 模块存储路径 |
| `/sbin/.core/db-0/magisk.db` | Magisk 数据库文件，记录模块和配置信息 |
| `/sbin/magiskinit` | Magisk 初始化程序 |
| `/sbin/magisk` | Magisk 主程序 |
| `/sbin/.magisk` | Magisk 核心目录的隐藏路径 |
| `/data/adb/magisk.img` | Magisk 虚拟文件系统镜像 |
| `/data/adb/magisk.db` | Magisk 数据库（存储于 `/data/adb` 时的路径） |
| `/data/adb/.boot_count` | Magisk 记录启动次数的文件 |
| `/data/adb/magisk_simple` | Magisk 简易模式标识文件 |
| `/data/adb/magisk` | Magisk 数据存储目录（`/data/adb` 下） |

## 二、通用 Root 工具文件
| 路径 | 说明 |
| --- | --- |
| `/data/local/su` | 常见的 `su` 权限工具路径（本地存储） |
| `/data/local/bin/su` | `su` 工具在 bin 目录的路径 |
| `/data/local/xbin/su` | `su` 工具在 xbin 目录的路径 |
| `/sbin/su` | 系统 sbin 目录下的 `su` 工具 |
| `/system/bin/su` | 系统 bin 目录下的 `su` 工具 |
| `/system/bin/.ext/su` | 系统 bin 扩展目录下的 `su` 工具 |
| `/system/bin/failsafe/su` | 系统 failsafe 目录下的 `su` 工具 |
| `/system/sd/xbin/su` | 系统 sd 分区 xbin 目录下的 `su` 工具 |
| `/su/xbin/su` | `su` 工具在 `/su` 目录的路径 |
| `/su/bin/su` | `su` 工具在 `/su` 目录 bin 下的路径 |
| `/magisk/.core/bin/su` | Magisk 环境下的 `su` 工具路径 |
| `/system/usr/we-need-root/su` | 第三方 Root 工具的特殊路径 |
| `/system/xbin/su` | 系统 xbin 目录下的 `su` 工具 |

## 三、挂载与系统配置文件
| 路径 | 说明 |
| --- | --- |
| `/cache/.disable_magisk` | 禁用 Magisk 的标识文件 |
| `/cache/magisk.log` | Magisk 日志文件 |
| `/init.magisk.rc` | Magisk 初始化脚本文件 |

## FeatCheckRoot
### `sub_487F4`
- 打开文件 `/proc/self/attr/prev`（进程自身属性历史文件）；
- 读取该文件内容；
- 在读取的内容中搜索字符串 `:zygote:`；
- 若找到则返回 1，否则返回 0（无论文件打开是否成功，未找到时均返回 0）。

其核心目的是通过检查进程属性历史，判断当前进程是否与 zygote 进程存在关联。

### `sub_48674`
- 第一步：检查进程与 zygote 的关联，调用 `chckRoot1_487F4()` 函数，若返回 1（即进程属性历史中存在 `:zygote:`），则直接返回 1（判定为 Root 相关状态）。
- 第二步：检查 Magisk 工具文件是否存在
  - 首先检查 `/data/magisk/resetprop`（Magisk 用于修改系统属性的工具文件），若存在则返回 1（判定为存在 Magisk）。
  - 若上述文件不存在，继续检查一组与 Magisk 相关的设备文件（如 `/dev/.magisk.unblock` 等，由 `v2 = &dev__magisk_unblock` 指向的列表），只要其中一个存在，就返回 1。
- 第三步：检查挂载信息中的 Magisk 痕迹。若前两步未检测到痕迹，则读取 `/proc/self/mounts`（进程挂载信息文件），并在内容中搜索一组与 Magisk 相关的路径字符串（如 `/sbin/.magisk/`、`/sbin/.core/mirror` 等，由初始化函数定义）。
  - 若搜索到任何一个目标字符串，返回 1（判定为存在 Magisk 挂载痕迹）。
  - 若未搜索到或文件操作失败，返回 0（未检测到 Root 相关痕迹）。

### `sub_48908`
JNI 检测是否存在特定类 `android/bde/BdeEngineManager`。

### `sub_48230`
`popen(which,su)` 检测返回结果。

### `sub_483DC`
检测 `su` 文件，`access` 遍历可能的 `su` 文件。

### `sub_47AC4`
#### 一、核心检测的系统属性与目标值
| 系统属性名 | 检测目标值 | 属性含义 |
| --- | --- | --- |
| `ro_debuggable` | `"1"` | 系统调试开关，设为 1 表示开启系统级调试（默认 0，开启后易被篡改） |
| `service_adb_root` | `"1"` | ADB root 权限开关，设为 1 表示允许通过 ADB 获取 root 权限 |
| `ro_secure` | `"0"` | 系统安全等级开关，设为 0 表示降低安全等级（默认 1，关闭后系统限制减少） |
| `sys_initd` | `"1"` | init.d 脚本支持开关，设为 1 表示启用 `init.d`（常用于 Root 后自启脚本，存在安全风险） |

#### 二、分版本检测逻辑（以 Android SDK 34 为分界）
SDK 版本按优先级依次检测，满足任一条件即返回 1（风险状态）：
- 条件 1：`ro_debuggable` 属性存在且值为 `"1"`；
- 条件 2：若条件 1 不满足，检测 `service_adb_root` 属性存在且值为 `"1"`；
- 条件 3：若条件 2 不满足，检测 `ro_secure` 属性存在且值为 `"0"`；
- 条件 4：若条件 3 不满足，检测 `sys_initd` 属性存在且值为 `"1"`。

SDK 版本 ≥ 34（高版本 Android 系统）

检测逻辑与低版本一致，但去掉了优先级顺序，直接并列检测 4 个条件，满足任一即返回 1（风险状态）：
- `ro_debuggable = "1"`；
- `service_adb_root = "1"`；
- `ro_secure = "0"`；
- `sys_initd = "1"`。

#### 三、关键补充说明
所有属性检测均通过 `_system_property_get_E1DC0` 函数获取属性值，若获取失败（返回值

### `sub_47DF0`
- 执行系统命令获取属性信息
  - 调用 `popen` 执行 `getprop` 命令（Android 系统获取所有系统属性的命令），以只读（r）模式读取输出结果。
  - `getprop` 命令会返回设备所有系统属性的键值对，函数通过 `fgets` 逐行读取这些内容到缓冲区 `s`。
- 检测关键不安全特征：逐行匹配缓冲区内容，若包含以下任一特征字符串，立即记录状态码、打印日志并终止进程：

| 检测的特征字符串 | 含义说明 | 触发的状态码 |
| --- | --- | --- |
| `[Android/aosp` | 设备系统为 Android 原生 AOSP 版本（无厂商安全加固，易被篡改） | 551 |
| `[userdebug]` | 系统为调试版本（userdebug 编译类型，默认开启调试权限，安全限制宽松） | 548 |
| `[unlocked]` | 设备引导加载器（Bootloader）已解锁（解锁后可刷入第三方 ROM、获取 Root 权限） | 549 |
| `.flash.locked]: [0]` | 设备闪存锁定状态为 0（即未锁定，允许修改系统分区） | 549 |
| `[ro.debuggable]: [1]` | 系统调试开关已开启（`ro.debuggable=1`，允许系统级调试，存在被篡改风险） | 550 |

### `sub_4837C`
`access` 检测 `/system/app/Superuser.apk`。

### `sub_47714`
- 读取挂载信息文件打开 `/proc/mounts` 文件（系统挂载信息的核心文件，记录所有分区的挂载路径、类型、权限等），以只读模式逐行读取内容。
- 解析挂载项并检测可写权限：对每一行挂载信息，通过 `sscanf` 解析出挂载路径、挂载点、文件系统类型和挂载权限等字段，重点检查挂载权限中是否包含 `rw`（可写标识）：
  - 函数中通过 `qword_E2938` 数组关联关键系统目录（如 `/system`、`/system/bin`、`/system/xbin`、`/vendor/bin`、`/sbin`、`/etc` 等），这些目录默认应为只读挂载，若被修改为可写则存在安全风险。
  - 若解析到某系统目录的挂载权限包含 `rw`，则设置标记 `v6` 为 1，表示检测到可写挂载。
- 结果返回：函数最终返回 `v6 & 1`，即 1 表示存在系统分区可写挂载（风险状态），0 表示所有检测的系统分区均为只读挂载（正常状态）。

## FeatCheckHook

### `sub_4E3A4`
#### 一、Xposed 框架检测
- 检查 Xposed 核心类：尝试查找 `de.robv.android.xposed.XposedBridge` 类，若该类存在或触发异常（说明环境中存在 Xposed 相关组件），则初步判定存在 Xposed 框架。
- 异常栈追踪验证：若检测到异常，通过获取异常栈信息（`java.lang.Throwable.getStackTrace`），并解析栈元素的字符串表示，进一步确认是否存在 Xposed 相关的调用痕迹。

#### 二、虚拟环境检测
- VirtualApp 标识检测：分析栈信息中是否包含 `com.lody.virtual`（VirtualApp 的包名特征），若存在则判定为运行在 VirtualApp 虚拟环境中。
- XposedHelpers 类补充验证：尝试查找 `de.robv.android.xposed.XposedHelpers` 类，进一步确认 Xposed 框架的存在。
- 进程与路径特征检测
  - 检测进程中是否包含 `io.va.exposed64`（Exposed 框架的进程特征）；
  - 检查系统路径中是否存在 `/io.va.exposed64/virtual/`（Exposed 虚拟环境的路径特征）。

#### 三、结果返回
若上述任一检测条件满足（存在 Xposed 框架、VirtualApp 或 Exposed 环境），函数返回 `true`；否则返回 `false`。

### `sub_48458`
`access` 检测以下文件：

| 路径 | 说明 |
| --- | --- |
| `/system/lib/libxposed_art.so` | Xposed 框架核心库（32 位） |
| `/system/lib64/libxposed_art.so` | Xposed 框架核心库（64 位） |
| `/system/xposed.prop` | Xposed 框架属性文件 |
| `/cache/recovery/xposed.zip` | Xposed 框架刷机包路径 |
| `/system/framework/XposedBridge.jar` | Xposed 框架桥接文件 |
| `/system/bin/app_process64_xposed` | Xposed 注入的进程文件（64 位） |
| `/system/bin/app_process32_xposed` | Xposed 注入的进程文件（32 位） |
| `/magisk/xposed/system/lib/libsigchain.so` | Magisk 版 Xposed 组件库 |
| `/magisk/xposed/system/lib/libart.so` | Magisk 版 Xposed 替换的运行时库 |
| `/magisk/xposed/system/lib/libart-disassembler.so` | Magisk 版 Xposed 反汇编库 |
| `/magisk/xposed/system/lib/libart-compiler.so` | Magisk 版 Xposed 编译器库 |
| `/system/bin/app_process32_orig` | Xposed 替换前的原始进程文件（32 位） |
| `/system/bin/app_process64_orig` | Xposed 替换前的原始进程文件（64 位） |
| `/system/lib/libmemtrack_real.so` | Xposed 相关的内存追踪库（32 位） |
| `/system/lib64/libmemtrack_real.so` | Xposed 相关的内存追踪库（64 位） |
| `/system/lib/libriru_edxp.so` | Riru 版 EdXposed 核心库（32 位） |
| `/system/lib64/libriru_edxp.so` | Riru 版 EdXposed 核心库（64 位） |
| `/system/lib/libwhale.edxp.so` | Whale 版 EdXposed 核心库（32 位） |
| `/system/lib64/libwhale.edxp.so` | Whale 版 EdXposed 核心库（64 位） |
| `/system/framework/edxp.jar` | EdXposed 框架桥接文件 |

### `sub_484D4`
- 读取进程内存映射文件
  - 生成当前进程的 `/proc/%d/maps` 路径（`%d` 为当前进程 PID），通过系统调用打开该文件。
  - 逐段读取文件内容到缓冲区 `v20`，分析进程的内存映射信息。
- 检测 Xposed/EdXposed 特征：检查内存映射内容中是否包含以下任一特征：
  - Xposed 核心库特征字符串（由 `v11` 指向，对应 Xposed 框架的关键库标识）；
  - `XposedBridge.jar`（Xposed 框架的桥接文件标识）；
  - `edxp.jar`（EdXposed 框架的桥接文件标识）。
- 结果返回：若检测到上述任一特征，返回 1（判定存在 Xposed/EdXposed 框架）；若读取完毕未检测到特征，返回 0（判定无风险）。

## FeatCheckPmsHook

### `sub_4ED18`
- 定位关键类与字段
  - 查找 `android.app.ActivityThread` 类，该类是 Android 应用进程的核心控制类。
  - 获取其静态字段 `sPackageManager`（该字段指向 `IPackageManager` 接口实例，负责应用包管理相关操作）。
- 检测动态代理特征
  - 查找 `java.lang.reflect.Proxy` 类（Java 动态代理的核心类）。
  - 检查 `ActivityThread.sPackageManager` 是否是 `Proxy` 的实例：
    - 若 `IPackageManager` 被动态代理（即 `sPackageManager` 是 `Proxy` 实例），说明包管理接口可能被篡改，函数会打印日志（标签 “XOX”，级别 7 为 `ERROR`），设置全局变量 `n551` 为状态码 546，并调用 `ExitProcess_1D638` 强制终止应用进程。
    - 若未被代理，返回检测结果（0 表示正常）。

该函数通过检测 `IPackageManager` 是否被动态代理，判断应用的包管理功能是否被恶意篡改（常见于 Hook 工具对应用安装、卸载等行为的拦截场景）。一旦检测到风险，立即终止应用，防止恶意操作（如静默安装恶意应用、篡改包信息等），保障应用的包管理安全性。

## FeatureAntiMthook
检测 MT 管理器。

### `sub_50DAC`
- `env->FindClass` `bin/mt/apksignaturekillerplus/HookApplication`
- `Dlopen` `libmthook.so`

### `sub_50E7C`
遍历所有 `FD`，用 `open` 打开并读取 `FD` 对应的目标路径（或通过 `readlink` 获取链接指向）。检测是否打开了指定前缀路径的文件。

## FeatureJavaAntiDebug

### `509C8`
JNI 调用 `android.os.Debug.isDebuggerConnected()` 静态方法，该方法返回 `boolean` 类型，`true` 表示进程已被调试器连接，`false` 表示未被调试。

## FeatureNativeAntiDebug

### `4F8A0`
