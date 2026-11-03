
# [Android 原创] 《安卓逆向这档事》十四、是时候学习一下Frida一把梭了(中)

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1838539
- 抓取时间: 2026-11-01T21:39:18

![](https://pic.rmb.bdstatic.com/bjh/13975df155153ae785b86e901db44d921765.png)

## 一、课程目标

1. 配置`objection`环境
2. 了解`objection`常用Api
3. 了解`frida_java_trace`工具分析控制流混淆

## 二、工具

1. 教程Demo(更新)
2. `jadx-gui`
3. `VS Code`
4. `IDLE`

## 三、课程内容

### 1.objection

#### 1.什么是objection

`objection`是基于`frida`的命令行hook集合工具, 可以让你不写代码, 敲几句命令就可以对java函数的高颗粒度hook, 还支持RPC调用。可以实现诸如内存搜索、类和模块搜索、方法hook打印参数返回值调用栈等常用功能，是一个非常方便的，逆向必备、内存漫游神器。

[`https://github.com/sensepost/objection`](https://github.com/sensepost/objection)

#### 2.objection环境配置

已不更新，要和`frida`的版本匹配。

`python`使用的版本建议大于3.8，不然可能会报错，或者你调低`frida`以及`objection`的版本。

```
pip install objection==1.11.0
pip install frida-tools==9.2.4
```

`frida` 版本：`14.2.18`

#### 3.objection快速上手

1. `help` 命令注释

执行 `objection --help`（`help` 命令）：

```
Checking for a newer version of objection...

Usage: objection [OPTIONS] COMMAND [ARGS]...

       _   _         _   _

   ___| |_|_|___ ___| |_|_|___ ___

  | . | . | | -_|  _|  _| | . |   |

  |___|___| |___|___|_| |_|___|_|_|

        |___|(object)inject(ion)

       Runtime Mobile Exploration

          by: @leonjza from @sensepost

  默认情况下，通信将通过USB进行，除非提供了`--network`选项。

选项:

  -N, --network            使用网络连接而不是USB连接。

  -h, --host TEXT          [默认: 127.0.0.1]

  -p, --port INTEGER       [默认: 27042]

  -ah, --api-host TEXT     [默认: 127.0.0.1]

  -ap, --api-port INTEGER  [默认: 8888]

  -g, --gadget TEXT        要连接的Frida Gadget/进程的名称。 [默认: Gadget]

  -S, --serial TEXT        要连接的设备序列号。

  -d, --debug              启用带有详细输出的调试模式。(在堆栈跟踪中包括代{过}{滤}理源图)

  --help                   显示此消息并退出。

命令:

  api          以无头模式启动objection API服务器。

  device-type  获取关于已连接设备的信息。

  explore      启动objection探索REPL。

  patchapk     使用frida-gadget.so补丁一个APK。

  patchipa     使用FridaGadget dylib补丁一个IPA。

  run          运行单个objection命令。

  signapk      使用objection密钥对APK进行Zipalign和签名。

  version      打印当前版本并退出。
```

2. 注入命令

```
objection -g 包名 explore
```

- `help`：不知道当前命令的效果是什么，在当前命令前加 `help`，比如 `help env`，回车之后会出现当前命令的解释信息。
- 按空格：不知道输入什么就按空格，会有提示出来。
- `jobs`：可以进行多项hook。
- 日志：`objection`的日志文件生成在 `C:\Users\Administrator\.objection`。

启动前就hook：

```
objection -g 进程名 explore --startup-command "android hooking watch class 路径.类名"
```

#### 4.objection基础api

- `memory list modules` - 查看内存中加载的库

```
memory list modules
Save the output by adding `--json modules.json` to this command
Name                                                              Base          Size                 Path
----------------------------------------------------------------  ------------  -------------------  ------------------------------------------------------------------------------
app_process64                                                     0x57867c9000  40960 (40.0 KiB)     /system/bin/app_process64
linker64                                                          0x72e326a000  229376 (224.0 KiB)   /system/bin/linker64
libandroid_runtime.so                                             0x72e164e000  2113536 (2.0 MiB)    /system/lib64/libandroid_runtime.so
libbase.so                                                        0x72dfa67000  81920 (80.0 KiB)     /system/lib64/libbase.so
libbinder.so                                                      0x72dec1c000  643072 (628.0 KiB)   /system/lib64/libbinder.so
libcutils.so                                                      0x72de269000  86016 (84.0 KiB)     /system/lib64/libcutils.so
libhidlbase.so                                                    0x72df4cc000  692224 (676.0 KiB)   /system/lib64/libhidlbase.so
liblog.so                                                         0x72e0be1000  98304 (96.0 KiB)     /system/lib64/liblog
```

- `memory list exports so名称` - 查看库的导出函数

```
memory list exports liblog.so

Save the output by adding `--json exports.json` to this command

Type      Name                                  Address

--------  ------------------------------------  ------------

function  android_log_write_int32               0x72e0be77c8

function  android_log_write_list_begin          0x72e0be76f0

function  __android_log_bswrite                 0x72e0be9bd8

function  __android_log_security                0x72e0bf2144

function  __android_log_bwrite                  0x72e0be9a18

function  android_log_reset                     0x72e0be75ec

function  android_log_write_string8             0x72e0be7a38

function  android_logger_list_free              0x72e0be8c04

function  __android_log_print                   0x72e0be9728

function  __android_logger_property_get_bool    0x72e0bf2248

function  android_logger_get_id                 0x72e0be8270

function  android_logger_set_prune_list         0x72e0be8948
```

- `android hooking list activities` - 查看内存中加载的activity
- `android hooking list services` - 查看内存中加载的services

![](https://pic.rmb.bdstatic.com/bjh/149aa38a7e5ef7ea0a4f1e2cfa0815a66608.png)

- `android intent launch_activity 类名` - 启动activity或service（可以用于一些没有验证的activity，在一些简单的ctf中有时候可以出奇效）
- 关闭ssl校验 `android sslpinning disable`
- 关闭root检测 `android root disable`

#### 5.objection内存漫游

- 内存搜刮类实例

执行 `android heap search instances 类名`：

```
android heap search instances 类名(命令)
Class instance enumeration complete for com.zj.wuaipojie.Demo  
 Hashcode  Class                  toString()
---------  ---------------------  -----------------------------
215120583  com.zj.wuaipojie.Demo  com.zj.wuaipojie.Demo@cd27ac7
```

- 调用实例的方法

调用 `android heap execute getPublicInt(实例的hashcode+方法名)`。

如果是带参数的方法，则需要进入编辑器环境：

```
android heap evaluate
console.log(clazz.a("吾爱破解"));
```

按住 `esc + enter` 触发。

- `android hooking list classes` - 列出内存中所有的类（结果比静态分析的更准确）

```
android hooking list classes
tw.idv.palatis.xappdebug.MainApplication
tw.idv.palatis.xappdebug.xposed.HookMain
tw.idv.palatis.xappdebug.xposed.HookMain$a
tw.idv.palatis.xappdebug.xposed.HookMain$b
tw.idv.palatis.xappdebug.xposed.HookMain$c
tw.idv.palatis.xappdebug.xposed.HookMain$d
tw.idv.palatis.xappdebug.xposed.HookSelf
u
v
void
w
xposed.dummy.XResourcesSuperClass
xposed.dummy.XTypedArraySuperClass

Found 10798 classes
```

- `android hooking search classes 关键类名` - 在内存中所有已加载的类中搜索包含特定关键词的类

```
android hooking search classes wuaipojie
Note that Java classes are only loaded when they are used, so if the expected class has not been found, it might not have been loaded yet.

com.zj.wuaipojie.Demo
com.zj.wuaipojie.Demo$Animal
com.zj.wuaipojie.Demo$Companion
com.zj.wuaipojie.Demo$InnerClass
com.zj.wuaipojie.Demo$test$1
com.zj.wuaipojie.MainApplication
com.zj.wuaipojie.databinding.ActivityMainBinding
... 

Found 38 classes
```

- `android hooking search methods 关键方法名` - 在内存中所有已加载的类的方法中搜索包含特定关键词的方法（一般不建议使用，特别耗时，还可能崩溃）
- `android hooking list class_methods 类名` - 内存漫游类中的所有方法
