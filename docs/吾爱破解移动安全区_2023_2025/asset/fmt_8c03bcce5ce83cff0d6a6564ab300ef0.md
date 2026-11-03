
```
v24 = v74;

if ( !v74 )
```

```
Interceptor.attach(libanon.base.add(0x27398), {     //fail
    onEnter: function () {
        this.lr = this.context.lr.sub(libanon.base)
        console.log("onEnter  0x27398 ", this.lr)
        Interceptor.detachAll()
    },
    onLeave: function (retval) {
        console.log("[0x27398] ", retval.toInt32(), "lr", this.lr)
    }
})
```

我们在这里执行全部卸载，发现还是崩溃了，说明在这里之前就已经对整个内存进行检测了，那么我们要找一个没有检测的点，来缩小排查范围。

找这个点的原则是，这个函数只调用了一次，并且卸载后能保证app顺利运行。

- `4F254`，可以
- `593FC`，可以
- `27398`，不行

那么就可以确定，问题出在 `593FC`~`27398` 之间。那么我们又从这个中间的函数开始进行排查，一步一步缩小位置：

- `3EA0C`，可以
- `15128`，不行

那么现在进一步缩小到了 `3EA0C`～`15128` 之间，分析的范围大大减小了：

![](https://attach.52pojie.cn/forum/202511/20/220622vr5uiewjheniwdib.png)

进一步分析，`sub_14F7C`，没有看出什么问题，暂时排除。

目前这些函数都没有分析出什么有效的信息。

# 16. 入口校验绕过（复制 `text` → 替换参数基址）

在没有什么有效信息的情况下，当一次侥幸哥，进行暴力拆解

## 16.1 发现对入口函数进行了检测

上面分析出来的 `sha256` 的 `sub_304A8`，来 hook 一下：

```
Interceptor.attach(libanon.base.add(0x304A8), {
    onEnter: function (args) {
        console.log("onEnter  0x304A8 ", args[0], args[1], args[2], "base ", rangeDetails.base)
    }
})
```

结果是：

![](https://attach.52pojie.cn/forum/202511/20/220624z99czt11ftst9tn4.png)

通过分析结果，发现 `304A8`，对入口的函数也做了检测

## 16.2 处理对入口函数检测

针对 `sub_304A8` 的入口校验：

先把匿名段 `text` 拷贝一份 `origin`；

若发现其校验针对当前段基址，则把参数中的基址替换为 `origin`（干净副本）以规避校验。

```
var origin
var size = 0x7a2a0

origin = Memory.alloc(size)
origin.writeByteArray(libanon.base.readByteArray(size))

Interceptor.attach(libanon.base.add(0x304A8), {
    onEnter: function (args) {
        if (args[1].toString() === libanon.base.toString()) {
            var rangeDetails = Process.findRangeByAddress(args[0]);
            console.log("onEnter  0x304A8 ", args[0], args[1], args[2], "base ", rangeDetails.base)
            args[1] = origin
            console.log("[0x304A8] 替换成功")
        }
    }
})
```

此时已经过掉了检测，并进入了首页。但是输出看到，不停在刷，`maps` 检测明显开了线程。

![](https://attach.52pojie.cn/forum/202511/20/220626trmvr7ztmntlmbn2.png)

# 17. 线程面收敛：定位 `pthread_create` 入口并“即刻返回”

虽然已经完成了过掉 frida 检测，为了完美一点，我们再来找到线程并处理掉它。

## 17.1 定位 `pthread_create`

根据 `/proc/self/maps` 调用的反向引用，追到 `pthread_create`

我们使用 `[0x40814]` `/proc/self/maps` `lr` => `0x3b66c` 的 `0x3b66c`，在 ids 中按 `x`，一步一步向上查找。最终发现了 `sub_7A230`，这个看起来很像是 `pthread_create` 函数：

![](https://attach.52pojie.cn/forum/202511/20/220628y4jrji76ivflvfe1.png)

`pthread_create` 函数原型：

```
#include

int pthread_create(
    pthread_t *thread,                  // [out] 新线程ID
    const pthread_attr_t *attr,         // [in]  线程属性(可为NULL)
    void *(*start_routine)(void *),     // [in]  线程入口函数
    void *arg                           // [in]  传给入口函数的参数
);
```

## 17.2 查找所有使用 `pthread_create` 的地方，并处理掉

找到被创建的线程入口地址，在运行态用 `Arm64Writer` 写入 `RET`，实现就地空返回。

在 ida 中，我们通过 `x` 键，查找 `sub_7A230` 的引用，然后去把线程入口函数给处理掉。

![](https://attach.52pojie.cn/forum/202511/20/220630w220sahlmliiyinn.png)

```
function retFunc(parg2) {
    // 修改内存保护，使其可写
    Memory.protect(parg2, 4, 'rwx');
    // 使用 Arm64Writer 写入 'ret' 指令
    var writer = new Arm64Writer(parg2);
    writer.putRet();
    writer.flush();
    writer.dispose();
    console.log("ret " + parg2 + " success");
}
retFunc(libanon.base.add(0x5A708))
retFunc(libanon.base.add(0x5BE28))
retFunc(libanon.base.add(0x5D1A4))
retFunc(libanon.base.add(0x5D710))
```

最终结果，没有那么多线程检测一直刷了，看着比较舒服。

![](https://attach.52pojie.cn/forum/202511/20/220749z326v2n2o99n7q88.png)

![](https://attach.52pojie.cn/forum/202511/20/220747ecttt8ot98hac9bd.png)

# 结论回顾（方法 > 结论）

- 入口选择决定成败：相对直接 Hook `System.loadLibrary`，从 `__loader_android_dlopen_ext` 切入能更早获得“真实装载面”的证据；
- 匿名映射段是关键战场：`JNI_OnLoad` → 函数指针 → 匿名段 这一跳，要求在内存态完成 dump 与“仅 `text` 段”的最小可用反汇编；
- 校验链路是主线：识别 `xxHash`/`SHA256`/`HMAC` 的组合与落点，用等式化替换与调用点定位做最小侵入的试探；
- 系统性缩小问题空间：从可卸载点开始做二分排除，定位到刷 `maps` 的线程入口，补丁而非大面积禁用，以减小误伤面与回归压力。

# 限制与风险

- 结论对 ROM/内核/ART 版本敏感：不同 SoC 与 API Level 下，`maps` 命名、权限组合、`linker` 细节均可能影响可见性与时序。
- 某些对抗属于“只对当下样本有效”：比如栈上/堆上指针偏移、匿名段大小、`HMAC` 初始材料位置等，需在发布后持续校验与回归。
- 工具链差异：`Zygisk Frida Gadget` 与纯 `frida-server` 的可见性与时序差异
