
```
case "java/util/zip/GZIPOutputStream->write([B)V": {
    OutputStream outputStream = (OutputStream) dvmObject.getValue();
    ByteArray array = varArg.getObjectArg(0);
    Inspector.inspect(array.getValue(), "java/util/zip/GZIPOutputStream->write outputStream");
    try {
        outputStream.write(array.getValue());
    } catch (IOException e) {
        throw new IllegalStateException(e);
    }
    return;
}
```
看看输出；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251204181707857.png)

勉强能看出一些东西，实际上他们大多数是来自于我们打开的`/dev/properties`这个文件里，我们没补但是为什么会有数据呢？

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251205100951787.png)

这是因为`unidbg`有内置的，做算法分析的话就用他这个就好，后续开始算法分析；

## 3.2 固定随机数

首先还是把`memcpy` hook好，这是我的习惯，代码不给出了，前面很多次了；

其次，整个运行结果还是随机的，这对于算法还原是极不友好的，一般来说，随机的来源有以下几种：

`JNI`、库函数、系统调用、文件访问等；

这里我只简单提一下，如果有兴趣我可以出一篇文章，另外，此理论来源于龙哥；

看一下输出日志：

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251205150332655.png)

如果你足够细心你是可以发现这个调用的，之所以我们没补是因为`AbstractJni`里已经为我们内置好了；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251205150440722.png)

这里是随机点1，我改成一个固定字符串了；

```
case "java/util/UUID->toString()Ljava/lang/String;": {

    UUID uuid = (UUID) dvmObject.getValue();

    // return new StringObject(vm, uuid.toString());

    // add

    return new StringObject(vm, "32DAB5DB-A036-4B83-8884-1E95A552C4B2");

}
```
这里提一个小技巧，我们在修改这种源码性质的代码时，最好在旁边加个标记，如我这里的 `// add`，后期想要改回来直接全局搜索就好，非常容易定位到；

再一个比较容易出现的点就是时间戳；`UnixSyscallHandler`这个文件下有一些系统调用的实现，准确说应该是库函数；

我们改 `gettimeofday64` 这个函数；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251205151022468.png)

```
long currentTimeMillis = 1724570121338L;
```
再次运行结果就会固定了；

```
第一次：
2afh0N9kGamECtqV8zoLP+OT0XPMBL0QemNaiXKm1tJrViD1j34DN9P0Gr9vZ1BZKU3v2qNnoT0qC6eDhRy1Giyz6aRhbJ3MwdwlV2UhsS9MiQU05Fz78OPf0FPQSL8b4QA3C3+S6WDE3se+lBAoOKK2QG4/N56/VQ+ovb1sgOPrcAuroeXlcic4GoeRxtOBjxWYeHWxF7qRp4hVQbYFMnwMuD/TfAM1rjz3kP/7kcB3RiWKh6DIbQUTxXcMt6ib0o08SWt45VVCHluWIRRNcKXRFsz/E7k936mA944GxTmmYePEWBvtfdaR3pjUPJMi4v7mG8mFWM+wC3ZC8NXCDa2OTBfKReV3ECtDIk5HkDJziHoibt83DKuKgqnr0jvDFSG7DGyAPsZMV7rpGKh/rnl5A==

第二次：
2afh0N9kGamECtqV8zoLP+OT0XPMBL0QemNaiXKm1tJrViD1j34DN9P0Gr9vZ1BZKU3v2qNnoT0qC6eDhRy1Giyz6aRhbJ3MwdwlV2UhsS9MiQU05Fz78OPf0FPQSL8b4QA3C3+S6WDE3se+lBAoOKK2QG4/N56/VQ+ovb1sgOPrcAuroeXlcic4GoeRxtOBjxWYeHWxF7qRp4hVQbYFMnwMuD/TfAM1rjz3kP/7kcB3RiWKh6DIbQUTxXcMt6ib0o08SWt45VVCHluWIRRNcKXRFsz/E7k936mA944GxTmmYePEWBvtfdaR3pjUPJMi4v7mG8mFWM+wC3ZC8NXCDa2OTBfKReV3ECtDIk5HkDJziHoibt83DKuKgqnr0jvDFSG7DGyAPsZMV7rpGKh/rnl5A==
```
这样数据就被固定住了，接下来开始算法还原；

## 3.3 算法分析

到这里，我们开始首次打开`ida`，事实上由于工作繁忙，从开始写到现在已经三天了···

去看目标函数，偏移是`0x104e8`；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251205151727846.png)

有轻微的混淆很正常，毕竟都是大厂了，实际上这个样本混淆没构成什么分析阻碍；

先分析一下密文，前面说了，打开`memcpy`的hook；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251205155424659.png)

得到一个浅显的信息，`2af`是独立的，其实这个多次抓包也可以发现，后续分析只看后面的部分就好；

然后再看看其他部分；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206103545707.png)

这个需要读者有一些魔数相关的储备，并且前面补环境就已经聊到`gzip`压缩，不往这方面想实在是有点不应该；我们去搜一下`gzip`的魔数；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206103701118.png)

这一部分就应当是一个压缩过后的数据，那我们当然可以看看压缩前的数据；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206103932391.png)

如果你对这组数据没印象了，请往前看，他就是我们补环境的时候`gzip`写入的那部分数据，前面说过了，实际上是指纹相关的数据，这组数据的来源我暂时没有分析，只看算法，有后续我也会写；

剩下的日志大概就是明文相关的了，可以用在后续指纹的获取分析上，这里我们开始分析这个算法；

首先看一下返回结果的位置，就是下面这个`NewStringUTF`结尾的地址；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206104729357.png)

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206104756158.png)

去`ida`看看这个地址；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206104817952.png)

它在`sub_11378`函数里，代码量不多，并且有字符串解密的存在；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206105115843.png)

这种我建议dump一下内存里的`so`，有些会呈现解密状态或更容易被我们分析，当然你也可以不管；我这里是分析的dump后的`so`，具体的步骤也不提了；后续我会给到文件链接；

先看看这个函数出参入参吧，注意，后续所有图片均是dump下来的`so`分析的，参数情况是这样的：

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206110727639.png)

```
emulator.attach().addBreakPoint(module.base + 0x11378);
```
`a1`不用看是`jnienv`，其余的参数是这样的：

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206110824595.png)

这是参数1，它是压缩后的数据，所以它前面应该还有个压缩函数；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206110854281.png)

参数2是压缩数据的长度，`0x12b`也及就是`299`，对得上；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206111125261.png)

参数3看起来是前缀，现在看返回值；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206111212890.png)

这是最开始看`NewStringUTF`定位的时候的位置，也就是说返回值应该看`x0`，除非`ida`有特别提示你返回值在哪个寄存器，否则就是`x0`，这属于调用约定；

在控制台下`blr`断点，这么久了这个应该不用多说了；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206111343779.png)

断下来之后发现它并不是一个地址，不信的可以去读一下；到这里属于陷入了一点僵局；

其实这种情况在前面某篇文章也遇到了，我不记得是哪篇了，总之`unidbg`在处理`java`对象的时候，如`map`等就会这样，返回的这个数值是类似于`hashcode`的东西，这是我的好友`XiaoFeng`去阅读源码后的结论，总之他需要使用`unidbg`的`api`去阅读，请看以下代码；

```
/**
* 打印Java对象内容
*/
protected void printJavaObject(long jobjectOffset, String label) {
    DvmObject javaObject = vm.getObject((int) jobjectOffset);
    if (javaObject != null) {
        System.out.println("[" + label + "] " + javaObject.getValue());
    } else {
        System.out.println("[" + label + "] null");
    }
}
```
那这里就不能再控制台玩了，写成持久化的形式；

```
emulator.attach().addBreakPoint(module.base + 0x11378, new BreakPointCallback() {
    @Override
    public boolean onHit(Emulator emulator, long address) {
        RegisterContext context = emulator.getContext();
        UnidbgPointer pointer0 = context.getPointerArg(1);
        UnidbgPointer pointer3 = context.getPointerArg(3);
        Inspector.inspect(pointer0.getByteArray(0, 299), "0x11378 入参1");
        Inspector.inspect(pointer3.getByteArray(0, 100), "0x11378 入参2");
        emulator.attach().addBreakPoint(context.getLRPointer().peer, new BreakPointCallback() {
            @Override
            public boolean onHit(Emulator emulator, long address) {
                UnidbgPointer pointer0 = context.getPointerArg(0);
                printJavaObject(pointer0.peer, "0x11378");
                return true;
            }
        });
        return true;
    }
});
```
参数这里不看了，直接看目标；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206112350788.png)

可以发现，正确的打印出了目标，说明这确实是目标函数没有任何问题，这个方法大家可以记住，后续又出现类似情况可以往这方面想想；

到现在，入参出参明确，继续分析；`sub_11378`这个函数代码里极少，非常容易找到关键位置；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206112719385.png)

`sub_53A08`的参数2 `v21`实际上就是传进来的明文，不用多讲了，再去`hook`；

```
emulator.attach().addBreakPoint(module.base + 0x53A08);
```
看看参数情况，`x0`是一串字符串，有`aes`的字样，可能是`key`；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206112913495.png)

`x1`则是压缩后的数据，可以看作是明文；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206113021051.png)

`x2`是长度，`x3`则是缓冲区；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206113040336.png)

那就下断点读返回值了；依旧是`blr`再按`c`，读`x3`的地址的值，切记不要去按`mx3`；此时`x3`早就不知道是什么东西了；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206113108924.png)

这串数据是我们没见过的，再看看伪代码；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206113549364.png)

还有一个`sub_53FE8`函数，经过了它计算可能就是最终的结果，进去看看；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206140823335.png)

重点可能也就在这一大段字符串解密上，如果你仔细数一下一共是`64`位，有可能是`base64`的码表，并且从结果来看是有充足的理由怀疑是经过`base64`的；

我们尝试把密文解一下，或者把那段数据编码一下；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206141456892.png)

结果完美契合，证明它是一个标准的`base64`函数，这里引申一下，我为何要看dump下来的`so`，请看对比；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206141821019.png)

原`so`：

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206141837145.png)

`dump`的`so`是已经解密的状态，而原`so`你也可以自行解密测试，因为它是原地解密的形式；看这一条测试；

![](https://www.xiayutian.xyz/usr/assets/android/pdd/image-20251206142037064.png)

手动解密出来也是可以的，我们去看`dump`后的`so`；

选中`9D110-9D148`按`U`把他们转成未定义状态，随后这段数据会呈现为金色的状态；
