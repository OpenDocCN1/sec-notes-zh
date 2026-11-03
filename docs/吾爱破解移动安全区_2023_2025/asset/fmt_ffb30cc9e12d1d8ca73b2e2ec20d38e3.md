
# [Android 脱壳] 记录一次加固逆向分析以及加固步骤详解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2008061
- 抓取时间: 2026-11-01T16:06:33.057Z

## 前言

最近想要重点学习一下类抽取这种类型的加固是如何实现的，故在网上搜寻。最终看到了luoyesiqiu大佬的dpt-shell这个项目。对这个项目研究后发现这一款开源加固已经可以说很成熟了。故先对其逆向分析后再从代码层面研究如何实现的。

项目地址：[//github.com/luoyesiqiu/dpt-shell](https://github.com/luoyesiqiu/dpt-shell:https)

分析版本：V1.12.2

## 准备阶段

欸嘿，是时候请出来之前的老朋友了（在之前分析某加固时用到的demo），然后采用dptshell进行加固

非常的方便，只需要现在编译好了的 `dpt.jar`，然后在命令框念出如下咒语：

（吾爱破解传不了大附件，又不太像搞网盘，附件就放一个脚本吧）

```
java -jar dpt.jar -f /path/to/apk
```

等待程序吟唱：

![](https://attach.52pojie.cn/forum/202511/04/120110cmqp8nj8sjm1ojir.png)

吟唱结束我们就得到了：

![](https://attach.52pojie.cn/forum/202511/04/120115mvbjjetwlzjtj8m4.png)

（wow，太贴心了，还帮我们进行了签名）

另外如果不想签名可以看如下帮助：

```
usage: java -jar dpt.jar [option] -f 

 -c,--disable-acf      Disable app component factory(just use for debug).

 -d,--dump-code        Dump the code item of DEX and save it to .json

                       files.

 -D,--debug            Make apk debuggable.

 -f,--apk-file    Need to protect apk file.

 -l,--noisy-log        Open noisy log.

 -x,--no-sign          Do not sign apk.
```

## 3> 逆向分析

### 4> 壳处理逻辑初步分析

![](https://attach.52pojie.cn/forum/202511/04/120117tynulgbbzzzbkuyc.png)

这里可以发现类都已经被抽取了。

首先我们可以看到工厂类已经被替换成了壳的代理工厂类

![](https://attach.52pojie.cn/forum/202511/04/120119t2qovxsccx755xnd.png)

那么这里其实就涉及到了一个知识点：

这个 `androidx.core.app.AppComponentFactory` 是用来动态控制组件示例话的，允许在 `Activity`、`Service`、`BroadcastReceiver`、`ContentProvider` 等组件被系统创建时拦截并替换其实例。dptshell 在此处进行壳 `so` 的加载，以及对一些系统函数的 Hook。

详细的我们可以继续往下面看

`ActivityThread.handleBindApplication()` 是按照什么顺序加载 APK 并加载应用程序组件的呢，我们可以看下图：

![](https://attach.52pojie.cn/forum/202511/04/120121cvlrag6f8li83id8.png)

这里我们着重要看的就是 `instantiateClassLoader` 这个方法了。

![](https://attach.52pojie.cn/forum/202511/04/120123r0tyd7qo6owzaqov.png)

这里主要载入了壳 `so`，然后到达代理 `Application`，完成了源 dex 的 `Application` 的替换了生命周期函数的调用，开始运行源 dex 的程序代码，并且为了其能够被正常加载做处理，后续会在源码分析中，详细来分析。

### 4> 壳 `so` 解密分析

那么对于此类抽取壳的分析，当然是要从 Native 层入手了。

![](https://attach.52pojie.cn/forum/202511/04/120125bc8oxrkkbncc0fkb.png)

这里我们选择分析 arm64 架构下的 `dpt.so`

这里我们直接 IDA 打开会发现 ELF 文件中 `bitcode` 段都被加密了

![](https://attach.52pojie.cn/forum/202511/04/120128r3yyo1aaa1vy6vvy.png)

#### 5> 静态解密 `bitcode` 段

一般出现这种情况我们就需要在从 `initArray` 段入手分析了，应为 `initArray` 段执行的是构造函数，在 `loadlibray` 之后就会立马被 `linker` 所执行。

正好我们在 `initArray` 段的 `sub_C67C` 函数中发现了如下函数：

![](https://attach.52pojie.cn/forum/202511/04/120130o90cujjjdnqznclj.png)

函数直接就以 `bitCode` 作为参数了，实属可疑

![](https://attach.52pojie.cn/forum/202511/04/120132z2jgjl1bznjb2vig.png)

`sectionName` 被传入了 `sub_FD4C`，这里就是在寻找 `bitcode` 段的地址，好对其进行后续的处理

![](https://attach.52pojie.cn/forum/202511/04/120134d561c3pve1h1ded8.png)

在 `sub_EB7C` 对内存页的权限进行修改，我们可以在 `segment` 窗口中看到 `bitcode` 段的权限是不可写的，所以要通过 `mprotect` 来修改内存页的权限，方便对代码进行动态解密。

![](https://attach.52pojie.cn/forum/202511/04/120136n30i4n2b4lqyqbqw.png)

![](https://attach.52pojie.cn/forum/202511/04/120138e88smljmqy7lyj9m.png)

那么上面分析完了，内存页权限修改完了，接下来要做的就是对内存中被加密的字节进行解密了，

![](https://attach.52pojie.cn/forum/202511/04/120140www8tqof4j9t7kmm.png)

流程上来看肯定就是这两个了。

![](https://attach.52pojie.cn/forum/202511/04/120143ptn5fsayaj7az75a.png)

![](https://attach.52pojie.cn/forum/202511/04/120145yfa21116b9292mzf.png)

相信大家都能一眼看出来这个是一个 RC4 吧。

根据下面的参数可以找到 `key`

![](https://attach.52pojie.cn/forum/202511/04/120147jniyy8tf4qe631et.png)

这里需要注意的是，`key` 的长度被限定到了 16，我们在写代码的时候不能用 `lenkey`，因为 `key` 后面很多 0。

使用 `idapython` 解密 `bitcode` 段：

```
import idc
import ida_segment
import idautils

def rc4_decrypt(key, data):
    """RC4解密实现"""
    S = list(range(256))
    j = 0
    out = []

    # KSA初始化
    for i in range(256):
        j = (j + S[i] + key[i % 16]) % 256
        S[i], S[j] = S[j], S[i]

    # PRGA生成密钥流并解密
    i = j = 0
    for byte in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        k = S[(S[i] + S[j]) % 256]
        out.append(byte ^ k)
    return bytes(out)

def decrypt_bitcode():
    # 配置目标段名（根据步骤1结果修改）
    target_segment = ".bitcode"  # 修改为你的段名

    # 获取段对象
    seg = ida_segment.get_segm_by_name(target_segment)
    if not seg:
        print(f"[!] 错误：未找到段 '{target_segment}'")
        return

    start_ea = seg.start_ea
    end_ea = seg.end_ea
    print(f" 找到段 {target_segment}: 0x{start_ea:X}-0x{end_ea:X}")

    # 读取加密数据
    encrypted_data = idc.get_bytes(start_ea, end_ea - start_ea)
    if not encrypted_data:
        print("[!] 错误：无法读取段数据")
        return

    # 定义RC4密钥
    rc4_key = bytes([
        0xE5, 0x5E, 0x5A, 0x20, 0x2C, 0x25, 0xD9, 0x1C, 0x72, 0x74,
        0x2E, 0x36, 0x99, 0x02, 0x80, 0x06, 0x00, 0x00, 0x00, 0x00,
        0x00, 0x00, 0x00
    ])

    # 执行解密
    decrypted_data = rc4_decrypt(rc4_key, encrypted_data)
    print("[+] 解密完成，正在写回IDA数据库...")

    # 临时修改段权限为可写
    original_perms = idc.get_segm_attr(start_ea, idc.SEGATTR_PERM)
    idc.set_segm_attr(start_ea, idc.SEGATTR_PERM, 0x7)  # RWX

    # 逐字节修补数据
    for offset, byte in enumerate(decrypted_data):
        idc.patch_byte(start_ea + offset, byte)

    # 恢复段权限
    idc.set_segm_attr(start_ea, idc.SEGATTR_PERM, original_perms)

    print("[+] 解密数据已成功写入，建议重新分析代码区域！")
    print("    操作完成！")

# 执行解密函数
decrypt_bitcode()
```

执行完后保存再重载文件会看到 `bitcode` 段代码被成功识别了：

![](https://attach.52pojie.cn/forum/202511/04/120149wqatjgatzmamtkwg.png)

#### 5> 内存中 `dump` 解密了的 `bitcode`

Dump SO 的话，不管你用 GDA 也好，还是什么小工具都可以，我这里展示 frida 的。

frida 的话首先还是得 hook `dlopen` 找到 `dlopen` 打开我们需要 dump 的 `so` 的时机，然后就可以开始获取 `so` 的 `Base` 和 `Size` 了，具体实现如下：

```
function my_hook_dlopen(soName,index) {
    //mapsRedirect();
    //hook_memcmp_addr();
    Interceptor.attach(Module.findExportByName(null, "android_dlopen_ext"),
        {
            onEnter: function (args) {
                var pathptr = args[0];
                if (pathptr !== undefined && pathptr != null) {
                    var path = ptr(pathptr).readCString();
                    if (path.indexOf(soName) >= 0) {
                        this.is_can_hook = true;
                    }
                }
            },
            onLeave: function (retval) {
                if (this.is_can_hook) {
                    if (index == 1) {
                        NativeFunc();
                        dump_so(soName);
                    } else {
                        dump_so2(soName);
                    }
                }
            }
        }
    );
}

function dump_so2(so_name) {
    var libso = Process.getModuleByName(so_name);
    console.log("[name]:", libso.name);
    console.log("[base]:", libso.base);
    console.log("[size]:", ptr(libso.size));
    console.log("[path]:", libso.path);
    var file_path = "/sdcard/Download/" + libso.name + "_" + libso.base + "_" + ptr(libso.size) + ".so";
    var file_handle = new File(file_path, "wb");
    if (file_handle && file_handle != null) {
        Memory.protect(ptr(libso.base), libso.size, 'rwx');
        var libso_buffer = ptr(libso.base).readByteArray(libso.size);
        file_handle.write(libso_buffer);
        file_handle.flush();
        file_handle.close();
        console.log("[dump]:", file_path);
    }
}

function dump_so(so_name) {
    var libso = Process.getModuleByName(so_name);
    console.log("[name]:", libso.name);
    console.log("[base]:", libso.base);
    console.log("[size]:", ptr(libso.size));
    console.log("[path]:", libso.path);
    var file_path = "/data/data/cn.pbcdci.fincryptography.appetizer/" + libso.name + "_" + libso.base + "_" + ptr(libso.size) + ".so";
    var file_handle = new File(file_path, "wb");
    if (file_handle && file_handle != null) {
        Memory.protect(ptr(libso.base), libso.size, 'rwx');
        var libso_buffer = ptr(libso.base).readByteArray(libso.size);
        file_handle.write(libso_buffer);
        file_handle.flush();
        file_handle.close();
        console.log("[dump]:", file_path);
    }
}

setImmediate(my_hook_dlopen("libdpt.so",2));
```

这里 `dump_so` 1 和 2 的区别在于一个是 `dump` 到私有目录，另一个是 `sdcard`，到 `sdcard` 是为了方便我们 `pull`，所以我默认使用 2。

启动！

![](https://attach.52pojie.cn/forum/202511/04/120151mtu0u626uggrgg2h.png)
