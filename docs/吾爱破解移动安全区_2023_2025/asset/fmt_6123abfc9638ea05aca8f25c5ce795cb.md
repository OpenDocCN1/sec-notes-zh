
```
{
  "mcpServers": {
    "ida-pro-mcp": {
      "url": "http://127.0.0.1:13337/sse  "
    },
    "jeb": {
      "command": "C:\\Users\\11602\\.local\\bin\\uv.exe",
      "args": [
        "--directory",
        "C:\\Users\\11602\\jebmcp\\jeb-mcp\\src\\jeb_mcp",
        "run",
        "server.py"
      ]
    },
  }
}
```

# APK样本介绍 Small Detector - 签名校验强度自测工具

`Small Detector`是一款专为检验Android应用签名校验机制强度设计的安全检测工具。

该APK通过模拟真实攻击场景，对目标应用的签名检测逻辑进行压力测试，帮助开发者评估其防护方案的有效性。

样本链接：https://bbs.kanxue.com/thread-288235.htm

## 核心检测维度

- `RootCheck`：检测设备Root状态及常见Root工具
- `HookCheck`：识别`Xposed`、`Substrate`等Hook框架
- `SignatureCheck`：验证APK签名完整性及一致性

本报告将针对以上三种检测机制进行深度技术分析，并提供相应的`Frida`绕过方案。

## 界面示例

随便打包查看界面如下：

![](https://attach.52pojie.cn/forum/202512/08/212215psqknkwinkqbsq8z.jpg)

![](https://attach.52pojie.cn/forum/202512/08/212218fvyp7fp9hdabpkni.jpg)

使用ai编写的脚本hook后界面如下

![](https://attach.52pojie.cn/forum/202512/08/212212gfm8fd4qu8rqfdid.jpg)

# APK安全检测分析报告

## RootCheck、HookCheck、SignatureCheck检测机制分析与绕过方案

### SignatureCheck 类分析报告

#### 1. 方法基本信息

##### 1.1 `Java_io_calvin_det_work_SignatureCheck_checkFeature`
- 地址: `0x0000000000084CF4`
- 大小: `0x64c` (1612字节)
- 功能推测: 检查应用的特征签名，可能是验证APK完整性或校验应用是否被篡改

##### 1.2 `Java_io_calvin_det_work_SignatureCheck_checkGroup`
- 地址: `0x00000000000821D8`
- 大小: `0x2d0` (720字节)
- 功能推测: 检查应用所属的组信息，可能用于验证应用来源或权限组

##### 1.3 `Java_io_calvin_det_work_SignatureCheck_checkIORedirect`
- 地址: `0x00000000000A3FD8`
- 大小: `0xfac` (4012字节)
- 功能推测: 检查IO重定向，可能用于检测应用是否被调试器附加或IO操作被劫持

##### 1.4 `Java_io_calvin_det_work_SignatureCheck_checkPermission`
- 地址: `0x00000000000A0074`
- 大小: `0x6e4` (1764字节)
- 功能推测: 检查应用权限，验证应用是否拥有预期的权限设置

##### 1.5 `Java_io_calvin_det_work_SignatureCheck_editPermission`
- 地址: `0x0000000000081A00`
- 大小: `0x7d8` (2008字节)
- 功能推测: 编辑或修改权限设置，可能用于动态调整应用权限以应对检测

##### 1.6 `Java_io_calvin_det_work_SignatureCheck_getConfusepathByteArray`
- 地址: `0x0000000000099160`
- 大小: `0x238` (568字节)
- 功能推测: 获取混淆路径的字节数组，可能用于获取经过混淆处理的路径信息

##### 1.7 `Java_io_calvin_det_work_SignatureCheck_getSystemSignatureByteArray`
- 地址: `0x000000000008878C`
- 大小: `0x398` (920字节)
- 功能推测: 获取系统签名的字节数组，用于对比当前应用签名与系统预期签名

##### 1.8 `Java_io_calvin_det_work_SignatureCheck_getUidByPath`
- 地址: `0x00000000000A1CDC`
- 大小: `0x22c` (556字节)
- 功能推测: 根据路径获取用户ID，用于验证文件或目录的所有者身份

##### 1.9 `Java_io_calvin_det_work_SignatureCheck_openat`
- 地址: `0x00000000000842D8`
- 大小: `0x1a0` (416字节)
- 功能推测: 打开指定路径的文件，可能用于访问特定的系统文件或配置文件

##### 1.10 `Java_io_calvin_det_work_SignatureCheck_readlinkat`
- 地址: `0x0000000000082BDC`
- 大小: `0x97c` (2428字节)
- 功能推测: 读取链接目标路径，用于解析符号链接的真实路径

##### 1.11 `Java_io_calvin_det_work_SignatureCheck_svcHookAnti`
- 地址: `0x0000000000067A90`
- 大小: `0x4c0` (1216字节)
- 功能推测: 通过系统调用实现反Hook检测机制

##### 1.12 `Java_io_calvin_det_work_SignatureCheck_verify_1fd_1apk_1path`
- 地址: `0x00000000000A0884`
- 大小: `0x7c0` (1984字节)
- 功能推测: 验证文件描述符对应的APK路径，确认APK来源合法性

#### 2. 关键Frida Hook信息

##### 2.1 Hook地址
- `checkFeature`: `0x84cf4`
- `checkGroup`: `0x821d8`
- `checkIORedirect`: `0xa3fd8`
- `checkPermission`: `0xa0074`
- `editPermission`: `0x81a00`
- `getConfusepathByteArray`: `0x99160`
- `getSystemSignatureByteArray`: `0x8878c`
- `getUidByPath`: `0xa1cdc`
- `openat`: `0x842d8`
- `readlinkat`: `0x82bdc`
- `svcHookAnti`: `0x67a90`
- `verify_1fd_1apk_1path`: `0xa0884`

##### 2.2 调用方式
这些方法都是通过`JNI`接口从Java层调用到Native层的函数，通常用于执行敏感操作或反调试检测。

##### 2.3 绕过方法
- 对于返回布尔值的方法，可以通过Hook修改其返回值为期望值（通常是false或0）来绕过检测
- 对于返回字符串或字节数组的方法，可以返回空值或合法的默认值
- 对于复杂的验证逻辑，可以在进入方法时记录参数，在离开时修改返回值

#### 3. 分析总结
`SignatureCheck`类实现了多种签名校验和安全检测机制，主要包括：
- 应用完整性校验（`checkFeature`, `getSystemSignatureByteArray`）
- 权限验证（`checkPermission`, `editPermission`, `checkGroup`）
- 路径和文件验证（`getUidByPath`, `openat`, `readlinkat`, `verify_1fd_1apk_1path`）
- IO重定向检测（`checkIORedirect`）
- 反Hook机制（`svcHookAnti`）
这些检测机制相互配合，构成了一个多层次的安全防护体系。要完全绕过这些检测，需要对每个方法进行针对性的Hook处理。

### HookCheck 类分析报告

#### 1. 方法基本信息

##### 1.1 `Java_io_calvin_det_work_HookCheck_hookCheck`
- 地址: `0x00000000000C25B8`
- 大小: `0xa30` (2608字节)
- 功能推测: 检测设备是否被Hook框架劫持，如`Xposed`、`Substrate`等
- 反编译代码分析:
- 函数初始化了大量的局部变量和字符串资源
- 调用了多个系统级API进行检测
- 最终返回检测结果

##### 1.2 `Java_io_calvin_det_work_HookCheck_svcHookCheck`
- 地址: `0x00000000000C8384`
- 大小: `0x64` (100字节)
- 功能推测: 通过系统调用(SVC)方式检测Hook框架
- 反编译代码分析:
- 直接调用底层函数进行检测
- 返回值与1进行按位与操作，返回布尔值

#### 2. 关键Frida Hook信息

##### 2.1 Hook地址
- `hookCheck`方法: `0xc25b8`
- `svcHookCheck`方法: `0xc8384`

##### 2.2 调用方式
- `hookCheck`: Java层通过`JNI`调用Native函数`Java_io_calvin_det_work_HookCheck_hookCheck`
- `svcHookCheck`: Java层通过`JNI`调用Native函数`Java_io_calvin_det_work_HookCheck_svcHookCheck`

##### 2.3 绕过方法
- 对于`hookCheck`方法，可以通过拦截其返回值并始终返回0或空字符串来绕过
- 对于`svcHookCheck`方法，可以通过拦截其返回值并始终返回false(0)来绕过

#### 3. 分析总结
`HookCheck`类主要用于检测设备上是否存在Hook框架，它采用了两种不同的检测方式：
- 基于常规API调用的检测(`hookCheck`)
- 基于系统调用的检测(`svcHookCheck`)
这两种检测机制都需要在Native层进行Hook才能有效绕过。

### SignatureCheck 类分析报告

#### 1. 方法基本信息

##### 1.1 `Java_io_calvin_det_work_SignatureCheck_checkFeature`
- 地址: `0x0000000000084CF4`
- 大小: `0x64c` (1612字节)
- 功能推测: 检查应用的特征签名，可能是验证APK完整性或校验应用是否被篡改

##### 1.2 `Java_io_calvin_det_work_SignatureCheck_checkGroup`
- 地址: `0x00000000000821D8`
- 大小: `0x2d0` (720字节)
- 功能推测: 检查应用所属的组信息，可能用于验证应用来源或权限组

##### 1.3 `Java_io_calvin_det_work_SignatureCheck_checkIORedirect`
- 地址: `0x00000000000A3FD8`
- 大小: `0xfac` (4012字节)
- 功能推测: 检查IO重定向，可能用于检测应用是否被调试器附加或IO操作被劫持

##### 1.4 `Java_io_calvin_det_work_SignatureCheck_checkPermission`
- 地址: `0x00000000000A0074`
- 大小: `0x6e4` (1764字节)
- 功能推测: 检查应用权限，验证应用是否拥有预期的权限设置

##### 1.5 `Java_io_calvin_det_work_SignatureCheck_editPermission`
- 地址: `0x0000000000081A00`
- 大小: `0x7d8` (2008字节)
- 功能推测: 编辑或修改权限设置，可能用于动态调整应用权限以应对检测

##### 1.6 `Java_io_calvin_det_work_SignatureCheck_getConfusepathByteArray`
- 地址: `0x0000000000099160`
- 大小: `0x238` (568字节)
- 功能推测: 获取混淆路径的字节数组，可能用于获取经过混淆处理的路径信息

##### 1.7 `Java_io_calvin_det_work_SignatureCheck_getSystemSignatureByteArray`
- 地址: `0x000000000008878C`
- 大小: `0x398` (920字节)
- 功能推测: 获取系统签名的字节数组，用于对比当前应用签名与系统预期签名

##### 1.8 `Java_io_calvin_det_work_SignatureCheck_getUidByPath`
- 地址: `0x00000000000A1CDC`
- 大小: `0x22c` (556字节)
- 功能推测: 根据路径获取用户ID，用于验证文件或目录的所有者身份

##### 1.9 `Java_io_calvin_det_work_SignatureCheck_openat`
- 地址: `0x00000000000842D8`
- 大小: `0x1a0` (416字节)
- 功能推测: 打开指定路径的文件，可能用于访问特定的系统文件或配置文件

##### 1.10 `Java_io_calvin_det_work_SignatureCheck_readlinkat`
- 地址: `0x0000000000082BDC`
- 大小: `0x97c` (2428字节)
- 功能推测: 读取链接目标路径，用于解析符号链接的真实路径

##### 1.11 `Java_io_calvin_det_work_SignatureCheck_svcHookAnti`
- 地址: `0x0000000000067A90`
- 大小: `0x4c0` (1216字节)
- 功能推测: 通过系统调用实现反Hook检测机制

##### 1.12 `Java_io_calvin_det_work_SignatureCheck_verify_1fd_1apk_1path`
- 地址: `0x00000000000A0884`
- 大小: `0x7c0` (1984字节)
- 功能推测: 验证文件描述符对应的APK路径，确认APK来源合法性

#### 2. 关键Frida Hook信息

##### 2.1 Hook地址
- `checkFeature`: `0x84cf4`
- `checkGroup`: `0x821d8`
- `checkIORedirect`: `0xa3fd8`
- `checkPermission`: `0xa0074`
- `editPermission`: `0x81a00`
- `getConfusepathByteArray`: `0x99160`
- `getSystemSignatureByteArray`: `0x8878c`
- `getUidByPath`: `0xa1cdc`
- `openat`: `0x842d8`
- `readlinkat`: `0x82bdc`
- `svcHookAnti`: `0x67a90`
- `verify_1fd_1apk_1path`: `0xa0884`

##### 2.2 调用方式
这些方法都是通过`JNI`接口从Java层调用到Native层的函数，通常用于执行敏感操作或反调试检测。

##### 2.3 绕过方法
- 对于返回布尔值的方法，可以通过Hook修改其返回值为期望值（通常是false或0）来绕过检测
- 对于返回字符串或字节数组的方法，可以返回空值或合法的默认值
- 对于复杂的验证逻辑，可以在进入方法时记录参数，在离开时修改返回值

#### 3. 分析总结
`SignatureCheck`类实现了多种签名校验和安全检测机制，主要包括：
- 应用完整性校验（`checkFeature`, `getSystemSignatureByteArray`）
- 权限验证（`checkPermission`, `editPermission`, `checkGroup`）
- 路径和文件验证（`getUidByPath`, `openat`, `readlinkat`, `verify_1fd_1apk_1path`）
- IO重定向检测（`checkIORedirect`）
- 反Hook机制（`svcHookAnti`）
这些检测机制相互配合，构成了一个多层次的安全防护体系。要完全绕过这些检测，需要对每个方法进行针对性的Hook处理。

AI编写的脚本只是修改了返回值；本文并没有通过AI深入研究所有方法的具体实现以及通用的绕过思路；分享的内容只是抛砖引玉，难免有所不足。

参考文章：
https://bbs.kanxue.com/thread-286813.htm
https://bbs.kanxue.com/thread-289245.htm
https://bbs.kanxue.com/thread-288235.htm
