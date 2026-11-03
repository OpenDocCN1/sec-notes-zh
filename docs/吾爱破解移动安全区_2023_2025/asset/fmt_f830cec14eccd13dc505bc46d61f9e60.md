
# MD5 加密分析记录

## 1. Native MD5 调用示例

```java
public void call_android_native_md5(){
    ArrayList list_arg = new ArrayList<>(10);
    list_arg.add(vm.getJNIEnv());//Env
    list_arg.add(0);//jclass
    String str = "chenchenchen777";
    byte[] bytes = str.getBytes();
    ByteArray byteArray = new ByteArray(vm, bytes);
    list_arg.add(vm.addLocalObject(byteArray));//str
    list_arg.add(1);
    Number number = module.callFunction(emulator, 0x01A981, list_arg.toArray());
    ByteArray object = vm.getObject(number.intValue());
    String result = new String(object.getValue());
    System.out.println("md5 encode result: " + result);
}
```

这样是的定位也是通过动态注册找的

![](https://attach.52pojie.cn/forum/202501/19/165844yaj6jep5yyv65yvy.png)

按照同样的方法先去试试主动调用，看看我们的结果和标准有差别没，没差别就不用分析了

![](https://attach.52pojie.cn/forum/202501/19/165846xcnn11olskcav8aa.png)

有差别就要分析。

首先是在`208`的位置找到了`MD5`的位置

![](https://attach.52pojie.cn/forum/202501/19/165849oz3cf73pfgf63fgo.png)

这里去`HOOK`一下看看参数：

```java
public void inline_Hook_md5(){

        attach.addBreakPoint(module.base + 0x13E3C, new BreakPointCallback() {

            @Override

            public boolean onHit(Emulator emulator, long address) {

                RegisterContext context = emulator.getContext();

                Inspector.inspect("encrypt md5 plainText addr :", (int) context.getPointerArg(0).peer);

                Inspector.inspect("encrypt md5 plainText len :", (int) context.getPointerArg(1).peer);

                Inspector.inspect("encrypt digest encryptoText addr :", (int) context.getPointerArg(2).peer);

                attach.addBreakPoint(context.getLRPointer().peer,new BreakPointCallback() {

                    @Override

                    public boolean onHit(Emulator emulator, long address) {

                        Inspector.inspect("Already encrypted:",0x01);

                        return false;

                    }

                });

                return false;

            }

        });

    }
```

![](https://attach.52pojie.cn/forum/202501/19/165853nft4btfvtacbtxyk.png)

![](https://attach.52pojie.cn/forum/202501/19/165858cm3gmb1bgbabbbdg.png)

可以看到这里，在我们输入的明文的后面添加了新的字符串进去，经过多次的尝试，发现其实是一样。不是随机生成的

```
dJLdCJiVnDvM9JUpsom9
```

拿着这串新的数据去`MD5`，发现也不是正确的结果，那么就说明是有问题的。

![](https://attach.52pojie.cn/forum/202501/19/165903rr6zu0dvf0fgvddo.png)

## 2. doMD5sign 分析

在`239`行的位置发现了`doMD5sign`函数，有点像关键函数

![](https://attach.52pojie.cn/forum/202501/19/165907prjccvczprro3wij.png)

里面是调用了`MD5`的，还有一些像是`bytesToInt`的转换。这里调用的`MD5`，先看看是不是正常的`MD5`。这里的`md5`的第三个参数是返回值的地址

![](https://attach.52pojie.cn/forum/202501/19/165910l2qpzxxe22zt9awy.png)

![](https://attach.52pojie.cn/forum/202501/19/165912nykx6l5xbwefuc1v.png)

可以看到，这里是标准的`MD5`加密。然后就再去`HOOK`一下`doMD5sign`函数，看看这里的返回值是什么，（这里的第三个参数，是一个二级指针。存放的是返回值的地址。）

```java
public void inline_Hook_doMD5sign(){

        attach.addBreakPoint(module.base + 0x014D54, new BreakPointCallback() {

            @Override

            public boolean onHit(Emulator emulator, long address) {

                RegisterContext context = emulator.getContext();

                Inspector.inspect("encrypt md5 plainText addr :", (int) context.getPointerArg(0).peer);

                Inspector.inspect("encrypt md5 plainText len :", (int) context.getPointerArg(1).peer);

                Inspector.inspect("encrypt digest encryptoText addr :", (int) context.getPointerArg(2).peer);

                attach.addBreakPoint(context.getLRPointer().peer,new BreakPointCallback() {

                    @Override

                    public boolean onHit(Emulator emulator, long address) {

                        Inspector.inspect("Already encrypted:",0x01);

                        return false;

                    }

                });

                return false;

            }

        });

    }
```

在函数结束之后去查看`encrypt digest encryptoText addr`的地址

![](https://attach.52pojie.cn/forum/202501/19/165924jp5jvk0w4qdvyp2q.png)

这里是小端序存储的地址，可以看到这里的结果就是我们加密的结果

![](https://attach.52pojie.cn/forum/202501/19/165927j8byvfw8bwbybf3b.png)

那么说明在`MD5`之后进行了数据的再加密，那么还得往下看

![](https://attach.52pojie.cn/forum/202501/19/165930yat45ggt4r994044.png)

能够发现的是这里的结果其实全是`bytesToInt`之后的拼接的结果。

![](https://attach.52pojie.cn/forum/202501/19/165932i9edq00ej4heumu0.png)

通过`D-810`简单的去除一点混淆之后其实就可以看到最后一个位置的加密加密过程了，其实也就是位运算的结果。

## 3. Python 复现

用`python`复现一下：

```python
import binascii

def bytes_to_int(src, offset):
    """
    还原 C 语言中的 `bytesToInt` 函数, 将字节数组中从 offset 开始的 4 个字节转换为 uint32_t 整数.

    参数:
    - src: MD5 结果的字节数组.
    - offset: 从哪个位置开始读取字节.

    返回:
    - 返回转换后的 uint32_t 整数.
    """
    # 按照 C 代码中的顺序来组合字节
    value = (src[offset + 1] 
```

这里的两个抓包过程中的`q`以及`sign`就可以分析完成了，可以直接通过分析复现协议算法。

本文章中所有内容仅供学习交流使用，不用于其他任何目的，擅自使用本文讲解的技术而导致的任何意外，与作者不负责
