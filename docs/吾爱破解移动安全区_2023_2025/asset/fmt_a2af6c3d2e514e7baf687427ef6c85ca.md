
```
                        new NativeCallback(() => console.log(`Interceptor.replace: 0x${offset.toString(16)}`), "void", [])
                    );
                });
            }
        }
    });
}
```

![](https://attach.52pojie.cn/forum/202504/07/125312tzw2yj5b721xykg5.png)

这里绕过了 frida 检测，调用了`call_function`函数，打印了一串字符串。

### 总结

这里只是对于在模拟器上如何实现跨架构进行 so 加载的探究流程，由于时间有点短，其中很多的细节没有细致的研究，只是对于整个模拟器 so 加载流程做了一个小小的判断。

我类比于 hook `dlopen`的方法想去 HOOK `NativeBridgeLoadLibrary`，想去得到对应的 frida 检测的 so 文件，所以去还原了`NativeBridgeItf`这个结构体，也是最终去得到了`loadLibraryExt`函数地址，进行了 HOOK `loadLibraryExt`，也是得到了和之前手机端一样的结果了。

之后也是复现了自己在手机端的绕过 frida 检测，也是成功绕过了。

但是确实有了一些对于 android 虚拟机的理解，比如这种跨架构的过程，好比逆向过程中把每一个指令进行拦截然后用对应的架构语言去解释，然后实现在自己的架构里面进行虚拟加载，确实对于安卓虚拟机有了一些认识。
