
## 控制流跟踪

### 657 号控制流

这里直接单步跟一下 657 号控制流，第 1、2 步 `new` 了一个方法。

![](https://s2.loli.net/2022/08/15/FnocLEsAjRDPK2J.png)

![](https://s2.loli.net/2022/08/15/N8mSnblxsgD7Bco.png)

这里就要注意了，容易跟丢，先进入 `_$bH` 方法打上断点，然后下一个断点就走到里面了，接着在单步调试，会进到另一个小的控制流里面，如下图所示：

![](https://s2.loli.net/2022/08/15/gsQzxrR7GitHWEK.png)

![](https://s2.loli.net/2022/08/15/f1YMyuHbcSD89pt.png)

### 96 号小控制流

开始单步跟第 96 号小控制流，第 1 步定义了一个变量。

![](https://s2.loli.net/2022/08/15/O1HmajzAhyPIGt8.png)

96 号小控制流，第 2 步将 `_$PI` 的值赋值给了 `_$fT`，而 `_$PI` 的值其实是 `window.localStorage.$_YWTU`，`window.localStorage` 里面有很多值，这个东西我们文章最后再讲，其中一些值与浏览器指纹相关，这里先知道他是取值就行了。

![](https://s2.loli.net/2022/08/15/Iph7a9mAZ3OBXjb.png)

96 号小控制流，第 3 步，进入第 94 号小控制流，最终生成的是一个 8 位数组，这个其实就是前面我们想要的 `_$Yb` 的值了。

![](https://s2.loli.net/2022/08/15/rkLh8yueKizCJZp.png)

后面没有什么特别的，中间几步我就省略了，照着扣代码就行了，然后 96 号小控制流，第 4 步，就将 `_$EJ` 的值赋值给 `_$Yb` 了。

![](https://s2.loli.net/2022/08/15/NByMKbzeo8RXOxf.png)

到这里先别急着结束，后面还有关键的几步，96 号小控制流，第 5 步，又遇到了和前面类似的写法。

![](https://s2.loli.net/2022/08/15/gfN4o5KMZVFRi1l.png)

同样的，先进 `_$pu` 打断点，再单步跟。

![](https://s2.loli.net/2022/08/15/tM5YGIW9ZbXLdpr.png)

来到另一个小控制流，如下图所示：

![](https://s2.loli.net/2022/08/15/VhBvPgETkH6YLzW.png)

10 号小控制流第 1 步，取 `window.localStorage.$_cDro` 的值，转为 `int` 类型，赋值给 `_$5s`，这个 `_$5s` 后续也会加到 128 位大数组里面。

![](https://s2.loli.net/2022/08/15/5AkWGMghjqdi7es.png)

10 号小控制流后续还有几步，没啥用可以省略，最后一步返回 96 号小控制流。

![](https://s2.loli.net/2022/08/15/qzsKyoUYemx1aih.png)

然后 96 号小控制流后续也没啥了，返回 657 号控制流。

![](https://s2.loli.net/2022/08/15/p87rMvnTC2196Ba.png)

### 279 号控制流

此时我们已经拿到 `_$Yb` 了，777 号控制流就先不管了，后续还有些代码先不管不用扣，等用到的时候再说，返回 279 号控制流，接着前面的步骤，来到第 17 步，变量 `_$5s` 经过 264 号控制流后，生成了一个值并添加到 128 位大数组里面，而 `_$5s` 的值正是前面我们跟 `_$Yb` 时，通过 777 号控制流拿到的，实际上也就是取 `window.localStorage.$_cDro` 的值，转为了 `int` 类型。

![](https://s2.loli.net/2022/08/15/UQa9IVN13nFRsLz.png)

279 号控制流，第 18、19、20 步，往 128 位数组里面添加了两个定值、一个 8 位数组。

![](https://s2.loli.net/2022/08/17/hdP7nJiBoN2KCx6.png)

![](https://s2.loli.net/2022/08/17/U6jfTOLD9IphSrR.png)

![](https://s2.loli.net/2022/08/17/8XbmoP2aLukGDsg.png)

279 号控制流，第 21 步，往 128 位数组里面添加了一个 `undefined` 占位，后续会有操作将其填充值。

![](https://s2.loli.net/2022/08/17/1VD6UkKZsNFc4W2.png)

![](https://s2.loli.net/2022/08/17/2ltvx1AFDXVKT7Y.png)

279 号控制流，第 22 步，进入 58 号控制流，58 号控制流与 `window.localStorage.$_fb` 的值有关，如果有这个值，就会生成 20 位数组，如果没有就是 `undefined`。58 号控制流就只有一步，返回一个变量，本文中是 `_$0g`。

![](https://s2.loli.net/2022/08/17/xlVNbtQTW3Khv8g.png)

![](https://s2.loli.net/2022/08/17/Rl1IvByMxr74p3H.png)

这个 `_$0g` 是咋来的呢？同样的直接搜索，下断点，发现是通过 112 号控制流得来的，往前跟栈，同样是先经过了 777 号控制流，和之前的情况类似，中间的过程就不看了，直接看这个 112 号控制流。

![](https://s2.loli.net/2022/08/17/Oylr58LvxAGsZfV.png)

本文中，112 号控制流传的参是 `_$bd[279]` 即 `$_fb`，112 号控制流第 1 步，进入 247 号控制流。

![](https://s2.loli.net/2022/08/17/rPj5y8LQEVwbTx3.png)

247 号控制流就 3 步，先将 `window.localStorage` 赋值给一个变量，然后取其中 `$_fb` 的值再返回。

![](https://s2.loli.net/2022/08/17/5Vg1lkYB2wXo43h.png)

![](https://s2.loli.net/2022/08/17/JkjdIHxDp6Bnyio.png)

![](https://s2.loli.net/2022/08/17/A9vyw6zKDpj3mcf.png)

112 号控制流第 2、3 步，一个 `try-catch` 语句，取 `window.localStorage.$_fb` 计算得到 25 位数组，然后取前 20 位并返回，这就是前面我们需要的 `_$0g` 的值了。

![](https://s2.loli.net/2022/08/17/Nh1MWBGqVdjFlm8.png)

![](https://s2.loli.net/2022/08/17/QW1zyJ4gipaSN5L.png)

279 号控制流，第 23 步，将前面 `window.localStorage.$_fb` 计算得到的 20 位数组添加到 128 位大数组里面，注意这一步如果没有 `window.localStorage.$_fb` 值的话，是不会添加的。

![](https://s2.loli.net/2022/08/17/MDfSNKuQyBFRU2g.png)

279 号控制流，第 24 步，对一个变量进行位运算，然后取 `window.localStorage.$_f0` 进行运算，如果 `$_f0` 为空的话是不会往 128 位大数组里添加值的。

![](https://s2.loli.net/2022/08/17/1fg4AwsE9NkdVPp.png)

![](https://s2.loli.net/2022/08/17/DElJv7akiz1WGS2.png)

![](https://s2.loli.net/2022/08/17/Gm63aq8ASe7urVR.png)

279 号控制流，第 25 步，对一个变量进行位运算，然后取 `window.localStorage.$_fh0` 进行运算，如果 `$_fh0` 为空的话是不会往 128 位大数组里添加值的。

![](https://s2.loli.net/2022/08/17/SFM7jJ4mf3e2Zia.png)

![](https://s2.loli.net/2022/08/17/ERl7dj31pzrhS2g.png)

![](https://s2.loli.net/2022/08/17/nvsCJNxkEQpHLiB.png)

279 号控制流，第 26 步，对一个变量进行位运算，然后取 `window.localStorage.$_f1` 进行运算，如果 `$_f1` 为空的话是不会往 128 位大数组里添加值的。

![](https://s2.loli.net/2022/08/17/XgDfCiPSYw9sub4.png)

![](https://s2.loli.net/2022/08/17/LhmnHXBiFrslAfv.png)

![](https://s2.loli.net/2022/08/17/KPInFkbt13eOy7Y.png)

279 号控制流，第 27 步，进入 611 号控制流，611 号控制流主要是检测 `window.navigator.connection.type`，即 `NetworkInformation` 网络相关信息，里面判断了 `type` 是不是 `bluetooth`、`cellular`、`ethernet`、`wifi`、`wimax`，正常的话应该返回 0。

![](https://s2.loli.net/2022/08/17/hTIfp8USEBbQyPF.png)

![](https://s2.loli.net/2022/08/17/R7Z6EybWAegmkSD.png)

![](https://s2.loli.net/2022/08/17/M2gxiDzysldZGPf.png)

279 号控制流，接下来几步都是类似的，这里就直接统称第 28 步了，首先对一个变量进行位运算，然后分别取 `window.localStorage.$_fr`、`window.localStorage.$_fpn1`、`window.localStorage.$_vvCI`、`window.localStorage.$_JQnh` 进行运算，同样如果这些变量为空的话，也是不会往 128 位大数组里添加值的。

![](https://s2.loli.net/2022/08/17/Bz6EQ9l4ynVsTM8.png)

![](https://s2.loli.net/2022/08/17/pS9nNeuwgjc56Md.png)

![](https://s2.loli.net/2022/08/17/GNgXSzvHpRnDIct.png)

![](https://s2.loli.net/2022/08/17/MHvrq53aKCFdjPD.png)

279 号控制流，第 29 步，往 128 位大数组里添加了一个定值 4，本文中该变量名是 `_$kW`。

![](https://s2.loli.net/2022/08/17/NoMWwKTFtCul7GL.png)

`_$kW` 这个变量是咋来的，和前面的套路类似，直接搜索下断，同样是经过开头的 777 号控制流得来的，如下图所示：

![](https://s2.loli.net/2022/08/17/aNVjv4xS8nr5HwO.png)

继续 279 号控制流，中间有一些变量位运算之类的就省略了，第 30、31 步，取了一个 `https:443` 的长度进行计算，先后往 128 位大数组里添加了一个定值和一个 9 位数组。

![](https://s2.loli.net/2022/08/17/hpeNDyMRFJzdS7l.png)

![](https://s2.loli.net/2022/08/17/VZxqsyLA3u6BErD.png)

279 号控制流，接下来几步都是在取值，都差不多，就统称为第 32 步了。

![](https://s2.loli.net/2022/08/17/OtuagsylWn53ENr.png)

![](https://s2.loli.net/2022/08/17/XaJBMrhI5LjDNK3.png)

![](https://s2.loli.net/2022/08/18/bimwZOAXyBNu7VH.png)

![](https://s2.loli.net/2022/08/18/l2Nv8dOj9ZKAIgL.png)

![](https://s2.loli.net/2022/08/18/rFjCHN6Ofnpg2E1.png)

![](https://s2.loli.net/2022/08/18/oIOTMwuaH5pvfkn.png)

279 号控制流，第 33 步，之前 128 位大数组第 12 位是个 `undefined`，这里就将第 12 位填充上了一个 4 位数组，其中有个变量 `_$8L`，前面我们跟步骤的时候就有一个变量一直在做位运算，此处的 `_$8L` 就是这么来的。

![](https://s2.loli.net/2022/08/18/2axiRUA5o8Euqfh.png)

279 号控制流，最后两步，原来的 128 位大数组，只取有值的前 21 位，一共有多少位与 `window.localStorage` 的某些值有关，有值的话就长一些，没有就短一些，然后再将数组的每个元素合并成最终的一个大数组并返回，279 号控制流就结束了。

![](https://s2.loli.net/2022/08/18/bh8Cxv6OeB5mgsM.png)

![](https://s2.loli.net/2022/08/18/Df9PHdheR4jSkpA.png)

### 742 号控制流

返回到文章开头的逻辑，279 号控制流结束，返回到 742 号控制流，第 2 步，定义了一个变量并生成了一个 32 位数组。

![](https://s2.loli.net/2022/08/18/upyNGbMvhiITA9d.png)

![](https://s2.loli.net/2022/08/18/NhQsV3RpOv8CKU1.png)

742 号控制流，第 3 步，取 `$_ts` 里面的某个值并赋值给一个变量。

![](https://s2.loli.net/2022/08/18/jN3Id7LhoOeSVvn.png)

742 号控制流，第 4 步，将前面 279 号控制流得到的大数组与上一步 `$_ts` 里面的某个值进行合并，合并后计算得到一个值。

![](https://s2.loli.net/2022/08/18/okxEP64F5YyaJuN.png)

742 号控制流，第 4 步，将上一步得到的值进一步计算得到一个 4 位数组，再将其和大数组合并。

![](https://s2.loli.net/2022/08/18/GxaUVYiJ7o8Ed5f.png)

742 号控制流，接下来几步是对时间戳进行各种操作，这里统称为第 5 步。

![](https://s2.loli.net/2022/08/18/8QyRaAFWgY6tp5x.png)

![](https://s2.loli.net/2022/08/18/XLZcslaxjk7z8uq.png)

![](https://s2.loli.net/2022/08/18/Rc3k71HxCXlFQwh.png)

![](https://s2.loli.net/2022/08/18/gyvi9eaK7UOcJRz.png)

742 号控制流，第 6 步，将上一步得到的 4 个时间戳进行计算，得到一个 16 位数组。

![](https://s2.loli.net/2022/08/18/y3xE6WVXjS15TrK.png)

742 号控制流，第 7 步，将上一步得到的 16 位数组进行异或运算。

![](https://s2.loli.net/2022/08/18/lvHQreL61EFyUNa.png)

742 号控制流，第 8 步，将上一步的 16 位数组进行计算，得到一个字符串。

![](https://s2.loli.net/2022/08/18/HrOhCJUKamTxuvB.png)

742 号控制流，第 9 步，正式生成 `cookie` 值，其中 `_$bd[274]` 定值，一般视为版本号，将上一步得到的字符串、之前得到的大数组和一个 32 位数组进行计算、组合，得到最终结果。

![](https://s2.loli.net/2022/08/18/2SJlDtwW4scTiHZ.png)

742 号控制流结束，返回 772 号控制流，利用了一个方法，组装 `cookie`，然后赋值给 `document.cookie`，整个流程就结束了。

![](https://s2.loli.net/2022/08/18/379Spdsc2zXQtjU.png)

![](https://s2.loli.net/2022/08/18/41bOXEVjkN89Bwv.png)

![](https://s2.loli.net/2022/08/18/NDZTlUsG5gy26f4.png)

代码中用到的 `$_ts` 的值需要我们自己去匹配出来，动态替换，这些步骤和 4 代是类似的，本文就不再重复叙述，可以参考 K 哥 4 代的那篇文章进行处理即可。

![](https://s2.loli.net/2022/08/18/97NGBhkOCvM4xug.png)

## 后缀生成

本例中，请求头中有个 `sign` 参数，`Query String Parameters` 有两个后缀参数，这两个后缀和 4 代类似，都是瑞数生成的。

![](https://s2.loli.net/2022/08/18/1FYZumyLOQg8Xbq.png)

![](https://s2.loli.net/2022/08/18/1niZhzkVqfjP6oC.png)
