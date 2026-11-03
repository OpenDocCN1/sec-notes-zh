
## 补充环境与系统调用

```
    System.out.println("call_init result-->>" + str);
}
```

- 这里开始需要补环境了，报错如下：

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114152821803.png)

- 这是一个系统调用相关的错误，`clock_gettime` 这个系统调用，不熟悉的读者可以留言我后面可以单独写一篇文章；我们可以从第一行报错点进去；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114153005460.png)

- 可以发现 `unidbg` 已经实现了一部分，去看看是哪一部分；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114153038181.png)

- 很好，没有 `id` 为 2 的情况，所以需要我们自己去补，正常补的话是一段苦人的差事，我选择偷懒；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114153541925.png)

- 非常非常不建议这么补，我这里只是偷懒，分析完我还要恢复的，但是这样执行就没问题了；另外，对于系统调用相关的报错可以多多结合 `ai`，它比我们见多识广；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114153652758.png)

## 固定随机数

- 前面分析了，有随机数 `rand` 的参与，结果一直在变，给它固定住；

```
public void hook_rand() {
    IHookZz hookZz = HookZz.getInstance(emulator);
    hookZz.wrap(module.findSymbolByName("lrand48"), new WrapCallback() {
        @Override
        public void preCall(Emulator emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
        }

        @Override
        public void postCall(Emulator emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
            ctx.setR0(1);
        }
    });

    hookZz.wrap(module.findSymbolByName("srand48"), new WrapCallback() {
        @Override
        public void preCall(Emulator emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
        }

        @Override
        public void postCall(Emulator emulator, HookZzArm32RegisterContext ctx, HookEntryInfo info) {
            ctx.setR0(1);
        }
    });
}
```

- 这部分 `hook` 代码可以留着，后续需要直接参考就可以，现在结果固定住了；回到我们刚进入函数的时候；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114160230673.png)

- `hook` 一下 `rand` 函数，看看固定后的结果是什么，这也是为了分辨后续哪一部分值是随机出来的；

```
emulator.attach().addBreakPoint(module.base + 0x3C6C);
```

- 断住后先在控制台敲 `bl` 再按 `c` 查看返回值 `R0`；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114160419963.png)

- 这里不理解的我也可以单独说一下，随机值需要记住；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114160446294.png)

## 分析 MD5 与参数

- 这里 `rand` 的结果，再看 `md5` 函数；

```
emulator.attach().addBreakPoint(module.base + 0x1C18);
```

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114160552505.png)

- 很长的数据，先看是不是标准的；这里有一些地方需要注意，参数1的长度有点长，尽量读的时候多读一些；最后加密对比是这样的：

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114161156915.png)

- 首先证明了他是一个标准的 `md5`，其次、参数还是整个 apk 的签名，首先看日志可以看出来这一点；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114161255791.png)

- 这两个值是一样的，其次，如果去分析 `md5_1C18(*(dword_A0D8 + 20))` 的话，也是可以找到证据的；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114161342265.png)

- 在 `sub_DD0` 函数就可以发现，从偏移 `20` 处开始取的，应用签名存的偏移就是这么多，那自然是对得上的；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114161452622.png)

- 所以这个 `md5` 的结果就是固定的，继续看 `sub_1EF0` 的参数和返回值；

## 分析 sub_1EF0 参数

```
emulator.attach().addBreakPoint(module.base + 0x1EF0);
```

- 参数有好几个，我们写成持久化的风格；

```
emulator.attach().addBreakPoint(module.base + 0x1EF0, new BreakPointCallback() {
    @Override
    public boolean onHit(Emulator emulator, long address) {
        RegisterContext context = emulator.getContext();
        UnidbgPointer pointer1 = context.getPointerArg(0);
        Inspector.inspect(pointer1.getByteArray(0, 0xe), "参数0");
        UnidbgPointer pointer2 = context.getPointerArg(1);
        Inspector.inspect(pointer2.getByteArray(0, 0x5), "参数1");
        UnidbgPointer pointer3 = context.getPointerArg(2);
        Inspector.inspect(pointer3.getByteArray(0, 0xa), "参数2 随机数");
        UnidbgPointer pointer4 = context.getPointerArg(3);
        Inspector.inspect(pointer4.getByteArray(0, 0x20), "参数3 md5签名");
        // 3. 获取栈指针
        UnidbgPointer sp = context.getStackPointer();
        int arg5 = sp.getInt(0); // 栈顶第一个32位值是第5个参数
        UnidbgPointer arg555 = UnidbgPointer.pointer(emulator, arg5);
        Inspector.inspect(arg555.getByteArray(0, 0x22), "参数4");
        //                int arg6 = sp.getInt(4); // 下一个32位值是第6个参数
        //                int arg7 = sp.getInt(8); // 再下一个32位值是第7个参数
        return true;
    }
});
```

- 这里引申出一个知识点，在 `arm32` 下，参数一般是放在 `R0-R3` 中，如果参数超过则逐个放在栈中，以小端序存储，并且一般在一个函数进来之后会把所有的参数都放在栈中保存起来，返回值通常都是通过 `R0` 返回；

- 看看结果；

```
>-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

- 参数1好像不太认识，它是来自 `byte_A0DC`，按下 `X` 没有交叉引用，但是他实际上是 `A0D8` 偏移 `4` 位存储的数据，但是我进去看发现不太对得上；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114165018337.png)

- 对应的位置是在获取 `ro.build.version.sdk`，`hook` 发现确实是；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114165304196.png)

- 那这里就并不是我们分析的那样，换一个 `ida` 版本看看；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114165725685.png)

- 解决问题，是有交叉引用的，在 `sub_DD0` 函数里；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114165800806.png)

## 分析 sub_DD0 与 sub_2010

- 传进来的参数就是 `byte_A0DC`，这个函数就是真正赋值的地方；从 `aFda893hflEsi` 这个内容里取数据，一共循环 `5` 次，我们可以打印一下 `sub_2010()` 函数的返回值；

```
emulator.attach().addBreakPoint(module.base + 0x2010);
```

- 返回值确实是这个，这里不贴图了，我们去看分析一下它的生成；

- 重点肯定就在于 `v3` 的值，这里反编译很差，所以去看汇编；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114174820988.png)

- 可以发现，`MUL.W R0, R5, R1` 是乘法指令，也就是说 `R1` 就是 `v3` 的值，我们去 `hook` 打印一下；

```
emulator.attach().addBreakPoint(module.base + 0x5868, new BreakPointCallback() {
    @Override
    public boolean onHit(Emulator emulator, long address) {
        RegisterContext context1 = emulator.getContext();
        int pointer2 = context1.getIntArg(1);
        System.out.println("0x5868 参数1:" + pointer2);
        emulator.attach().addBreakPoint(context1.getLRPointer().peer, new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator emulator, long address) {
                RegisterContext context = emulator.getContext();
                int pointer1 = context.getIntArg(1);
                System.out.println("0x5868 返回值:" + pointer1);
                return true;
            }
        });
        return true;
    }
});
```

- 输出如下：

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251114174957615.png)

- 返回值分别是：0、0、2、0、5；根据这段算法还原一下；

```
aFda893hflEsi = "fda&8^%$#)93hfl_esi.*"
def sub_2010():
    a1 = [0, 0, 0, 0, 0]
    v3 = [0, 0, 2, 0, 5]
    for i in range(0, 5):
        # v3 = sub_5868(0xC, (i + 3))
        a1[i] = aFda893hflEsi[i * v3[i] - i + 4]
    return a1

print(sub_2010())
```

- 生成的结果就是：`8&%d*`，和 `hook` 的结果是对得上的；这里的 `v3` 跟进去实际上也不是一个多复杂的算法，感兴趣可以去还原一下，实际上没有进行随机化，每个设备都可能是这个值，就说到这里；

## 分析 sub_3688

- 回到正题，接下来看看 `sub_3688` 函数；参数1是这样的：

![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115144431598.png)

```
emulator.attach().addBreakPoint(module.base + 0x3688);
```

- 其实就是前面的函数拼接起来，两个参数没有问题，但是返回值有些不对劲，请看：
