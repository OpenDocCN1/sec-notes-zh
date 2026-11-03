
```
            writer.flush();

        });

        patchedOffsets.add(offset);

        console.log("[nopFunctionEntry] patched 0x" + offset.toString(16));

    } catch (e) {

        console.log("[nopFunctionEntry] patch failed 0x" + offset.toString(16) + " -> " + e);

    }

}

function patch_target_addr() {

    const module = Process.findModuleByName("目标so文件.so");

    if (!module) return;

    nopFunctionEntry(module, 0x1B8D4);

    nopFunctionEntry(module, 0x19E0C);        //对应的三个线程执行的函数

    nopFunctionEntry(module, 0x1C544);

}

function hook_linker_call_constructors() {

    const linker64_base_addr = Module.getBaseAddress('linker64');

    if (!linker64_base_addr) {

        console.log("linker64 not found");

        return;

    }

    const offset = 0x52110;

    const call_constructors = linker64_base_addr.add(offset);

    Interceptor.attach(call_constructors, {

        onEnter: function (args) {

            const secmodule = Process.findModuleByName("目标so文件.so");

            if (secmodule) {

                console.log('hook_linker_call_constructors onEnter 目标so文件');

                patch_target_addr();

            }

        }

    });

}

hook_linker_call_constructors();
```

![](https://attach.52pojie.cn/forum/202511/29/164555ffje6cjt769c99t0.jpg)

所以还是有点问题，我们继续分析，发现`sub_1B8D4`和`sub_19E0C`两个函数最终都会调用`sub_234E0`这个函数来实现闪退，那么我们直接patch掉这个函数不就等同于对这两个线程函数检测的bypass了嘛。

![](https://attach.52pojie.cn/forum/202511/29/164558zz4q612x22ztoox4.jpg)

然后发现在线程函数`sub_1C544`中对应执行类似的持续扫描闪退的操作的函数应该是`sub_26334`函数。

![](https://attach.52pojie.cn/forum/202511/29/164600xp8ud0ksuuixgs9g.jpg)

![](https://attach.52pojie.cn/forum/202511/29/164602uv3ia3pfk8vdkyy0.jpg)

有此重大发现之后可以将bypass的脚本更新为如下，如此可以最小程度上的对程序进行修改的同时实现bypass：

```
const patchedOffsets = new Set();
function nopFunctionEntry(module, offset) {
    const addr = module.base.add(offset);
    if (patchedOffsets.has(offset)) {
        console.log("[nopFunctionEntry] already patched 0x" + offset.toString(16));
        return;
    }
    try {
        Memory.patchCode(addr, 4, code => {
            const writer = new Arm64Writer(code, { pc: addr });
            writer.putRet();
            writer.flush();
        });
        patchedOffsets.add(offset);
        console.log("[nopFunctionEntry] patched 0x" + offset.toString(16));
    } catch (e) {
        console.log("[nopFunctionEntry] patch failed 0x" + offset.toString(16) + " -> " + e);
    }
}

function patch_target_addr() {
    const module = Process.findModuleByName("目标so文件.so");
    if (!module) return;
    nopFunctionEntry(module, 0x234E0);//直接造成检测闪退的地址
    nopFunctionEntry(module, 0x26334);//直接造成检测闪退的地址
}

function hook_linker_call_constructors() {
    const linker64_base_addr = Module.getBaseAddress('linker64');
    if (!linker64_base_addr) {
        console.log("linker64 not found");
        return;
    }
    const offset = 0x52110;
    const call_constructors = linker64_base_addr.add(offset);
    Interceptor.attach(call_constructors, {
        onEnter: function (args) {
            const secmodule = Process.findModuleByName("目标so文件.so");
            if (secmodule) {
                console.log('hook_linker_call_constructors onEnter 目标so文件');
                patch_target_addr();
            }
        }
    });
}

hook_linker_call_constructors();
```

可以成功绕过检测：

![](https://attach.52pojie.cn/forum/202511/29/164605ocy0u0cugcgt1r1g.jpg)

### 3> 0x04 总结

此文分析仅代表个人见解，希望师傅们多多指正捏~

参考文章：[参考文章1](https://xiaoeeyu.github.io/2024/08/09/%E7%BB%95%E8%BF%87%E7%88%B1%E5%A5%93%E8%89%BAlibmsaoaidsec-so%E7%9A%84Frida%E6%A3%80%E6%B5%8B/index.html)
