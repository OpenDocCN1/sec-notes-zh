
# BYD 白盒 AES 分析

## 定位写入与打印结果

```
emulator.traceWrite(0x40568000, 0x40568000 + 2);
```

- 去看看打印结果；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111144332993.png)

- 位置对了，这里去看 `pc` 处的偏移 `0x1a150`；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111144614087.png)

- 发现它在一个 `base64_encode` 函数，那就是最后做了一次 base64 编码，那我们手工解码回去；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111144729671.png)

- 记住解码的结果，在分析的时候可以截个小图留在置顶方便分析；这里去看看 base64 传进来的是不是我们手工解码的数据，如果一直就证明它是标准的；

## 定位 AES 数据来源

```
emulator.attach().addBreakPoint(module.base + 0x1A0EC);
```

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111145014104.png)

- 确实是标准的，所以现在的目标变成了找 `1b bc ef` 这一段怎么来的，继续 `traceWrite`，地址就是 `0x4055ac00`；

```
emulator.traceWrite(0x4055ac00, 0x4055ac00 + 2);
```

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111145826237.png)

- 数据并不匹配，但是可以去看看 `0x98ec`；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111145910650.png)

- 之所以要来看就是因为有符号提示你，本身猜测就是对称加密，这里就大概可以心里有点底了；

- 那 `1b bc ef` 可能就是 aes 加密的结果；

- `WBACRAES_EncryptOneBlock` 这个函数有些问题，找一下上图函数的交叉引用，它也叫 `WBACRAES_EncryptOneBlock`；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111150104275.png)

- 这里有关键性的提示，到这里我们先不着急，汇总一下已知的情况；

- `aes128`、`cbc`、白盒；

- 正常来说我们需要找 `key` 和 `iv`，白盒的话相对来说就不是正常的方法去找；

- 继续接着上面往下说，要确定走没走，`hook` 一下就知道了；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111151806354.png)

## 分析 WBACRAES_EncryptOneBlock

```
emulator.attach().addBreakPoint(module.base + 0x98B4);
```

- 这里肯定是找明文嘛，`x1` 寄存器就是明文；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111151902888.png)

- `hook` 到了证明确实走了，那就继续往下跟，去看里面的 `CWAESCipher::WBACRAES_EncryptOneBlock`，它的偏移是 `9894`，它也是走了的，但是函数实现比较可怜；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111152007388.png)

- 这种情况伪代码是不方便看了，那就去看汇编；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111152032643.png)

- 代码就几条，就不翻译了，`BLR` 就是跳转，那我们就去 inline hook 一下，看看 `x4` 的值是多少，就是对应的函数调用；

```
emulator.attach().addBreakPoint(module.base + 0x98A8);
```

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111152214439.png)

- 对应的地址是 `0x7f68`，跳过去看看；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111152247139.png)

- 这就是它真正调用的位置了，读者可以 `hook` 验证，是没问题的；

- 接下来分析这个函数就行，最终的目的是 `dfa` 攻击，先看 `PrepareAESMatrix` 函数；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111153822226.png)

- 这个函数应该是一个矩阵 `state` 化的函数，应该说是一个矩阵准备函数，我们来验证一下；

- 这里的 `a3` 就是最后的矩阵，`hook` 的时候去看 `x2` 寄存器；

```
emulator.attach().addBreakPoint(module.base + 0x70E4);
```

- 这里需要注意，`x2` 直接看是得不出什么结论的，函数执行结束的时候再看，按 `blr` 下断点到 `lr` 寄存器，然后按 `c` 执行完成之后通过 `x2` 的值去读内容；这里切记是读地址而不是 `mx2`；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111154149792.png)

- 直接看可能看着有点模糊，我稍微修剪一下；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111154405779.png)

- 这样就直观多了，`0xbfffeb10` 就是 `state` 的地址；这个地址后续会和 `key` 进行初始轮密钥加，但是在开始之前它应该要喝 `iv` 进行异或，所以这里的值应该是异或之后的结果才对；并且我们知道，0 和其他数据异或，依旧是本身，这是一个特性，我们暂且猜测是这样；

## 定位 PrepareAESMatrix 与 state

- `dfa` 需要这样几个因素：找轮数、找 `state`、找时机；

- 轮数和时机可以看作一体，目标是最后两次列混淆之间进行就行，看下图就明白了；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111155109646.png)

- 也就是说：第 8 次的轮密钥加和第 9 次的字节代换和第 9 次的循环左移这三个时机均可；

- 再就是找 `state`，这个前面刚刚说过；再看整体的轮次；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111155313174.png)

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111155338168.png)

- 接下来就可以进行 `dfa` 了，找到一个合适的时机；这个读者可以随意选择，只要是在第一张图的 9 轮里里面就可以，先看看循环次数；这里有个点需要注意，明文记得改一下，否则分组次数太多了，我这里改的是 `hello`；

```
   public void call_dfa(){
        Debugger debugger = emulator.attach();
        debugger.addBreakPoint(module.base+0x7ffc,new BreakPointCallback() {
            UnidbgPointer pointer;
            int num = 1;                    //m0xbfffeb10
            @Override
            public boolean onHit(Emulator emulator, long address) {
                pointer = UnidbgPointer.pointer(emulator, 0xbfffeb10L); // state
                System.out.println("num->"+num);

                num+=1;
                return true;
            }

        });

    }
```

- 没问题，次数是合理的；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111160021701.png)

- 这里需要注意，万事俱备，但并没有所有事情都准备好，比如我们要看的是 aes 的结果，所以还需要额外写上一个看 aes 结果的 `hook`，这个无所谓哪个方法，只要能 `hook` 到 aes 的输出就可以，也可以参照我的代码写；

- 先看一次注入的结果，这里先不给 `dfa` 的代码：

```
正常：E6F6549E 5DCCB842 9ED21690 32665B6B

dfa：E6F6419E 5D6DB842 14D21690 32665BAF
```

## DFA 注入与故障密文

- 可以发现，异常出现在四个字节，这就是一个良好的故障密文，这个具体可以去看 `dfa` 的原理；接下来就是大量获取故障密文；

```
public static void main(String[] args) {
    byd demo = new byd();
    //        demo.hookMemcpy();
    for (int i = 0; i >" + res);
}

public static int randint(int min, int max) {
    Random rand = new Random();
    return rand.nextInt((max - min) + 1) + min;
}

public void call_dfa() {
    Debugger debugger = emulator.attach();
    debugger.addBreakPoint(module.base + 0x7ffc, new BreakPointCallback() {
        UnidbgPointer pointer;
        int num = 1;                    //m0xbfffeb10

        @Override
        public boolean onHit(Emulator emulator, long address) {
            pointer = UnidbgPointer.pointer(emulator, 0xbfffeb10L); // state
            //                System.out.println("num->" + num);
            if (num % 9 == 0) {
                Random random = new Random();
                int randomLine = random.nextInt(4); // 生成 0 到 3 之间的随机数
                int randnum = randint(0, 255);
                pointer.setByte(randint(0, 15), (byte) randnum); //随机注入
            }
            num += 1;
            return true;
        }
    });
}

public void HookByConsoleDebugger() {
    Debugger debugger = emulator.attach();
    debugger.addBreakPoint(module.base + 0x98b4, new BreakPointCallback() {
        int num = 0;

        @Override
        public boolean onHit(Emulator emulator, long address) {
            RegisterContext context = emulator.getContext();
            num += 1;
            long x1 = emulator.getBackend().reg_read(Arm64Const.UC_ARM64_REG_X1).longValue();
            emulator.attach().addBreakPoint(context.getLRPointer().peer, new BreakPointCallback() {
                @Override
                public boolean onHit(Emulator emulator, long address) {
                    if (num == 1) {
                        Backend backend = emulator.getBackend();
                        byte[] bytes = backend.mem_read(x1, 0x10);
                        StringBuilder hexString = new StringBuilder();
                        for (byte b : bytes) {
                            hexString.append(String.format("%02X", b & 0xFF));
                        }
                        System.out.println(hexString);
                    }
                    return true;
                }
            });
            return true;
        }
    });
}
}
```

- 打印密文之前把输出都取消掉，方便后续复制；

![](https://www.xiayutian.xyz/usr/assets/android/byd%E7%8E%8B%E6%9C%9D%E7%99%BD%E7%9B%92AES_img/image-20251111163140229.png)

## 使用 phoenixAES 进行 DFA

- 我这里是跑了 30 次，拿去 `dfa`，这里要用到 `phoenixAES` 这个库，`pip` 即可；

```
import phoenixAES

with open('data','wb') as f:

    f.write("""

E6F6549E5DCCB8429ED2169032665B6B

E6F6549E5DCCB8429ED2169032665B6B

E6F6549E5DCCB8429ED2169032665B6B

E6F6549E5DCCB8429ED2169032665B6B

E6D8549E37CCB8429ED216893266086B

11F6549E5DCCB8479ED2759032245B6B

E6F6549E5DCCB8429ED2169032665B6B

E6F6549E5DCCB8429ED2169032665B6B
```
