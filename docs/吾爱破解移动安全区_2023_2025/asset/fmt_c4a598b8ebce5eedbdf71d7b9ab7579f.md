
```
[0x82ee4]: b'-wal\x00\x00\x00\x00'
[0x4dc4c]: b'generic_x86/sdk_x86/generic_x86'
[0x82fac]: b'bugle_db'
[0x4dc5c]: b'Andy\x00\x00\x00\x00'
[0x83b2c]: b'.xml\x00\x00\x00\x00'
[0x4dc6c]: b'ttVM_Hdragon'
[0x83b98]: b'.\x00\x00\x00\x00\x00\x00\x00'
[0x4dc7c]: b'generic_x86_64'
[0x83ca0]: b'.db\x00\x00\x00\x00\x00'
[0x4dc8c]: b'generic/google_sdk/generic'
[0x83d40]: b'.\x00\x00\x00\x00\x00\x00\x00'
[0x4dc9c]: b'vbox86p\x00'
[0x85f54]: b'libc.so\x00'
[0x4dcac]: b'generic/vbox86p/vbox86p'
[0x85f6c]: b'open\x00\x00\x00\x00'
[0x4dcbc]: b'google/sdk_gphone64'
[0x85f88]: b'close\x00\x00\x00'
[0x4dccc]: b'emulator'
[0x85fa4]: b'read\x00\x00\x00\x00'
[0x4dcdc]: b'android/os/Build'
[0x85fc0]: b'write\x00\x00\x00'
[0x4dcec]: b'FINGERPRINT'
[0x85fdc]: b'munmap\x00\x00'
[0xbc38]: b'FeatCheckEmulator'
[0x860ac]: b'pread64\x00'
[0x860c8]: b'pwrite64'
[0x860e4]: b'mmap64\x00\x00'
```

## 检测分析

### FeatureMemoryChecker

初始化 `inotify` 监控（自身进程）：

- 分配内存存储 `inotify` 相关数据（文件描述符、监控事件掩码等）。
- 调用 `inotify_init()` 创建 `inotify` 实例，用于监控文件系统事件。
- 监控自身进程的 `/proc/self/mem`（进程内存镜像）和 `/proc/self/pagemap`（进程页映射信息），监控事件掩码为 35（结合 `inotify` 事件定义，可能包含 `IN_ACCESS`、`IN_MODIFY` 等常用事件）。
- 创建线程 `TF_EmptyMemory_4F168`，传入监控相关数据。

`inotify` 事件的处理逻辑（`TF_EmptyMemory_4F168` 线程的核心处理函数），主要逻辑如下：

- 循环读取事件：通过 `read(a1, s, 0xAA0uLL)` 从 `inotify` 文件描述符（`a1`）循环读取事件（`a1` 为监控相关数据，包含 `inotify` 实例的文件描述符）。
- 事件解析与处理：
  对读取到的事件数据（存储在 `s` 中）进行循环解析，每次处理一个事件结构。
  检查事件类型（`v4 = ((_DWORD)s_1 + 1)`），若为 32、2、1 中的任意一种（对应 `inotify` 特定事件，如访问、修改、删除等），则：
  通过 `__android_log_print` 输出日志（标签 "XOX"，状态 507）。
  设置全局变量 `n551 = 507`。
  调用 `ExitProcess_1D638(507)` 强制退出程序。

但似乎我进行内存 dump 的时候并没有闪退？

核心功能：监控进程内存相关文件的访问 / 修改行为（可能是调试器或恶意程序的操作），一旦检测到则主动退出以防止被分析或篡改

### FeatLibcProtection

`DoLibcProtection_166D8`

调用 `dlsym` 拿 `libc` 常用函数的地址 存到全局变量中进行调用

### FeatAntiDump

`sub_350EC`

检测脱壳工具核心类

- `top/niunaijun/blackbox/core/VMCore`（BlackBox 脱壳工具核心类），触发状态码 531；
- `java/f8left/fupk3/core/Fupk`（FUpk3 脱壳工具核心类），触发状态码 533；
- `android/app/Aupk`（Aupk 脱壳工具核心类），触发状态码 532；
- `cn/youlor/Unpacker`（Unpacker 脱壳工具核心类），触发状态码 534。

检测脱壳工具相关文件

- 检查 `/data/local/tmp/FUpk3.txt` 或 `/data/local/tmp/libFUpk3.so`（FUpk3 工具的临时文件 / 库文件），触发状态码 533；
- 检查 `/data/local/tmp/unpacker.config`（Unpacker 工具的配置文件），触发状态码 534；
- 检查 `/data/local/tmp/fext.config`（Fext 相关脱壳工具的配置文件），触发状态码 536。

检测风险方法注册

- 检查 `dalvik/system/DexFile` 类中是否存在 `dumpMethodCode` 方法（用于 Dump Dex 方法代码的风险方法），触发状态码 535；
- 检查 `dalvik/system/DexFile` 类中是否存在 `fartextMethodCode` 方法（类似的代码提取风险方法），触发状态码 536。

当 `n2 == 2` 时：注册原生方法并校验环境

- 尝试查找 `com/sagittarius/v6/D` 类，并为其注册原生方法（`RegisterNatives`）。
- 若类查找失败或方法注册失败，判定为环境异常（可能被脱壳工具篡改），触发状态码 537 并终止应用。

### AntiFrida

`sub_2E2E0`

检测 Frida 相关线程与内存特征

- 遍历 `/proc/self/task` 目录，读取每个线程的 `status` 文件，检查是否包含 `gum-js-loop` 或 `gmain`（Frida 的线程特征）。
- 读取 `/proc/self/maps` 文件，分析内存映射信息，查找 `path` 前缀 `/data/local/` 查找是否包含 `FRIDA_PEER_SETUP_ACTIVE`（Frida 的内存特征字符串）。
- 检测 `linjector` 注入工具遍历 `/proc/self/fd` 目录，分析文件描述符的链接信息，检查是否包含 `linjector`（动态注入工具的特征）。

结果处理

- 若检测到上述任一工具的特征，通过 `__android_log_print` 打印日志（标签 “XOX”，级别 7 为 ERROR），设置全局变量 `n551` 为对应状态码（如 543、544、545），并调用 `ExitProcess_1D638` 强制终止应用进程。
- 若未检测到特征，返回常规检测结果。

`sub_2E8DC`

初始化与前置检查

- 检查全局标记 `byte_E23C0[0]`（完整性校验初始化标记），若已初始化则跳过初始化流程，直接进入校验阶段。
- 若未初始化且未标记为 “校验失败”，则定位 `libart.so` 库文件，检查该库是否正常加载。

定位关键方法与内存保护调整

- 查找 `libart.so` 中的 `_ZN3art9ArtMethod12PrettyMethodEb` 方法（`ArtMethod.PrettyMethod`，用于获取方法的字符串描述，是 `libart.so` 的核心方法）。
- 通过内存地址范围匹配，确定该方法对应的内存页，调整内存保护属性（`mprotect` 设置为可读写执行权限），为后续校验做准备。

关键方法完整性校验

- 再次获取 `_ZN3art9ArtMethod12PrettyMethodEb` 方法的内存地址，记录其原始内存数据（`qword_E23B8`）和方法地址（`qword_E23C8`），并标记初始化完成（`byte_E23C0[0] = 1`）。
- 后续调用时，校验当前 `_ZN3art9ArtMethod12PrettyMethodEb` 方法的内存数据与原始记录是否一致：
  - 若不一致（说明方法被篡改，可能是 Hook 或注入导致），则打印日志（标签 “XOX”，级别 7 为 ERROR），设置全局变量 `n551` 为状态码 `547`，调用 `ExitProcess_1D638` 强制终止应用进程。
  - 若一致，则说明 `libart.so` 核心方法未被篡改，正常执行后续逻辑。

### LogProtection

`sub_34F88`

- 先通过 `sub_2DB60` 函数加载并定位 `libandroid_runtime.so` 库（Android 运行时核心库，包含日志相关系统函数）。
- 若定位成功，进一步通过 `sub_2023C` 解密得到目标函数名 `__android_log_buf_write`（Android 系统底层日志写入函数，所有应用日志最终都会通过该函数输出）。

执行 Hook 操作

- 调用 `HOOK_2D820` 函数对 `__android_log_buf_write` 进行 Hook，将其原始执行逻辑替换为自定义函数 `sub_3504C` 为空函数
- 保存 Hook 后的函数地址到全局变量 `off_E2540`，用于后续可能的恢复或二次操作。

结果返回

- 若 Hook 成功，返回替换后的函数地址；若定位库失败或 Hook 失败，返回对应错误标识（0 相关结果）。

### FeatCheckEmulator

核心函数 `sub_4D3D0` 的作用：

- 准备模拟器特征字符串列表：通过一系列解密函数（`sub_20BDC` 等）生成一组字符串，这些字符串是常见的模拟器标识，包括：`sdk`、`google_sdk`、`Andy`、`ttVM_Hdragon`（天天龙模拟器）、`Droid4X`（海马玩模拟器）、`nox`（夜神模拟器）、`sdk_x86`、`sdk_google`、`vbox86p`（VirtualBox 模拟器）、`emu64x`（64 位模拟器）。
- 获取系统 `PRODUCT` 属性值：调用 `sub_1F1D0` 函数，获取 Android 系统中 `android.os.Build.PRODUCT` 属性的字符串值（该属性通常标识设备的产品名称，模拟器会有特定值）。
- 匹配模拟器特征：检查获取到的 `PRODUCT` 属性值中是否包含上述列表中的任何一个模拟器标识。
  - 若包含，则返回 1（判定为模拟器）；
  - 若遍历完所有标识都不匹配，则返回 0（判定为非模拟器）

`sub_4D550`

- 准备模拟器制造商特征列表通过一系列解密函数（`sub_2165C` 等）生成一组字符串，这些字符串是常见模拟器的制造商标识，包括：`unknown`、`Genymotion`（知名模拟器）、`Andy`（安迪模拟器）、`MIT`（可能关联特定模拟器）、`nox`（夜神模拟器）、`TiantianVM`（天天模拟器）、`iToolsAVM`（iTools 模拟器）、`Netease`（网易模拟器）。
- 获取系统 `MANUFACTURER` 属性值调用辅助函数 `sub_1F1D0`（之前分析过的 JNI 工具函数），获取 Android 系统中 `android.os.Build.MANUFACTURER` 属性的字符串值（该属性通常标识设备的制造商名称，模拟器会有特定的制造商标识）。
- 匹配模拟器特征检查获取到的 `MANUFACTURER` 属性值中是否包含上述列表中的任何一个模拟器制造商标识：
  - 若包含，则返回 1（判定为模拟器）；
  - 若遍历完所有标识都不匹配，则返回 0（判定为非模拟器）。

`sub_4D6B0`

- 准备模拟器特征标识列表通过解密函数（`sub_21F1C` 等）生成一组模拟器相关特征字符串，包括：`generic`、`generic_x86`（通用 x86 架构标识，常见于模拟器）、`Andy`（安迪模拟器）、`TTVM`（天天模拟器相关）。
- 获取系统 `BRAND` 属性值调用辅助函数 `sub_1F1D0`（JNI 工具函数），获取 Android 系统 `android.os.Build.BRAND` 属性的字符串值。该属性标识设备品牌，模拟器通常会使用通用品牌名或专属品牌标识。
- 特征匹配判断检查获取到的 `BRAND` 属性值是否包含上述特征列表中的任意一个标识：
  - 包含则返回 1，判定为模拟器；
  - 遍历所有标识均不匹配则返回 0，判定为非模拟器。

`sub_4D7D0`

- 生成模拟器特征标识列表通过一系列解密函数（`sub_2245C` 等）生成 11 个模拟器相关特征字符串，包括：`generic`、`generic_x86`、`generic_x86_64`（通用架构标识，模拟器常用）、`Andy`（安迪模拟器）、`ttVM_Hdragon`（天天模拟器）、`Droid4X`（海马玩模拟器）、`nox`（夜神模拟器）、`vbox86p`（VirtualBox 模拟器）、`iToolsAVM`（iTools 模拟器）、`emu64x`（64 位模拟器）、`emulator`（直接模拟器标识）。
- 获取系统 `DEVICE` 属性值调用辅助函数 `sub_1F1D0`（JNI 工具函数），获取 Android 系统 `android.os.Build.DEVICE` 属性的字符串值。该属性标识设备的具体型号或设备代号，模拟器会带有专属标识。
- 特征匹配与结果返回检查获取到的 `DEVICE` 属性值是否包含上述特征列表中的任意一个标识：
  - 若包含，返回 1，判定为模拟器；
  - 遍历所有标识均不匹配，返回 0，判定为非模拟器。

`sub_4D960`

- 生成模拟器特征标识列表通过解密函数（`sub_22FBC` 等）生成 10 个模拟器相关特征字符串，覆盖主流模拟器及 SDK 测试设备标识：`sdk`、`google_sdk`、`Droid4X`（海马玩模拟器）、`TiantianVM`（天天模拟器）、`Andy`（安迪模拟器）、`Android SDK built for x86_64`（x86_64 架构 SDK 模拟器）、`Android SDK built for x86`（x86 架构 SDK 模拟器）、`iToolsAVM`（iTools 模拟器）、`MuMu`（沐沐模拟器）、`sdk_gphone64`（64 位 SDK 测试设备，常见于模拟器）。
- 获取系统 `MODEL` 属性值调用辅助函数 `sub_1F1D0`（JNI 工具函数），获取 Android 系统 `android.os.Build.MODEL` 属性的字符串值。该属性标识设备的型号名称，模拟器通常会显示 SDK 相关名称或专属型号标识。
- 特征匹配与结果返回检查获取到的 `MODEL` 属性值是否包含上述特征列表中的任意一个标识：
  - 若包含，返回 1，判定为模拟器；
  - 遍历所有标识均不匹配，返回 0，判定为非模拟器。

`sub_4DAE0`

- 生成模拟器硬件特征列表通过解密函数（`sub_23A3C` 等）生成 4 个模拟器专属的硬件特征字符串：`goldfish`（Android 官方模拟器的硬件标识）、`vbox86`（VirtualBox 模拟器相关硬件标识）、`nox`（夜神模拟器）、`ttVM_x86`（天天模拟器 x86 架构版本）。
- 获取系统 `HARDWARE` 属性值调用辅助函数 `sub_1F1D0`（JNI 工具函数），获取 Android 系统 `android.os.Build.HARDWARE` 属性的字符串值。该属性标识设备的硬件型号，模拟器的硬件标识通常是固定的专属名称，与真实设备差异明显。
- 特征匹配与结果返回检查获取到的 `HARDWARE` 属性值是否包含上述特征列表中的任意一个标识：
  - 若包含，返回 1，判定为模拟器；
  - 遍历所有标识均不匹配，返回 0，判定为非模拟器。

`sub_4DC00`

生成模拟器指纹特征列表通过解密函数（`sub_23F7C` 等）生成 10 个模拟器相关的 `FINGERPRINT` 特征字符串，涵盖通用模拟器、架构专属模拟器及知名第三方模拟器标识：

- 通用类：`generic/sdk/generic`、`generic/google_sdk/generic`、`emulator`
- 架构专属类：`generic_x86/sdk_x86/generic_x86`、`generic_x86_64`
- 第三方模拟器类：`Andy`（安迪模拟器）、`ttVM_Hdragon`（天天模拟器）、`vbox86p`（VirtualBox 模拟器）
- SDK 测试设备类：`google/sdk_gphone64`、`generic/vbox86p/vbox86p`

获取系统 `FINGERPRINT` 属性值调用辅助函数 `sub_1F1D0`（JNI 工具函数），获取 Android 系统 `android.os.Build.FINGERPRINT` 属性的字符串值。该属性是设备的唯一指纹标识，由厂商、设备型号、系统版本等信息组合而成，模拟器的指纹标识具有明显的专属特征，且不易篡改。

特征匹配与结果返回检查获取到的 `FINGERPRINT` 属性值是否包含上述特征列表中的任意一个标识：

- 若包含，返回 1，判定为模拟器；
- 遍历所有标识均不匹配，返回 0，判定为非模拟器。

`sub_4DD80`

准备 MuMu 模拟器专属文件列表函数中涉及的文件包括：

- `/etc/mumu-configs/device-prop-configs/mumu.config`（MuMu 模拟器的设备属性配置文件）
- `/data/local/cfg-gdaeg`、`/data/local/cfg-yzi`、`/data/local/cfg-hzg`、`/data/local/cfg-znz`、`/data/local/cfg-ndi`、`/data/local/cfg-zhz`（MuMu 模拟器本地配置文件系列）
- `/mnt/windows/BstSharedFolder` 等（MuMu 模拟器与 Windows 共享文件夹的挂载路径）
