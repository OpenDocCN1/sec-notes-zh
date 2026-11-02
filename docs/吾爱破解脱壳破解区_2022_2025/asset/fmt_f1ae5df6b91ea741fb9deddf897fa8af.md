
![](https://s2.loli.net/2022/06/30/VpSCie6b4D5LGkw.png)

通过前面我们的分析，我们也知道了有两次生成 Cookie 的地方，快速搜索 `5`，搜索结果第二个即为入口：

![](https://s2.loli.net/2022/06/30/egxvfLmXQohK3iT.png)

## 假 Cookie 生成逻辑

首先单步跟假 Cookie，虽然是假的，但是后续生成真 Cookie 中会用到，在跟的时候你会走到这个逻辑里面：

![](https://s2.loli.net/2022/06/30/9Mfd2hgTRApY7ct.png)

有一步会调用 `_$8e()` 方法，而 `_$8e` = `_$Q9`，`_$Q9` 又嵌套在 `_$d0` 里的，搜索一下哪里调用了 `_$d0`，发现是代码开头：

![](https://s2.loli.net/2022/06/30/PR3JOmejlXprqEU.png)

那么传入的参数 `_$Wn` 是啥呢？单步跟入，是一个方法，作用就是取 202 页面的 `content` 内容，那么我们在本地就直接删掉这个 `_$Wn` 方法，直接传入 `content` 的值即可，如下图所示：

![](https://s2.loli.net/2022/06/30/pMSh49Z5m7CoOjH.png)

另外，我们发现，代码有非常多的在数组里面按索引取值的情况，比如上图中的 `_$PV[68]` 的值，实际上就是字符串 `content`，很显然我们要把这个数组的来源找到，直接搜索 `_$PV` =，可以找到疑似定义和赋值的地方：

![](https://s2.loli.net/2022/06/30/M3x5LR6e8XdwWFm.png)

![](https://s2.loli.net/2022/06/30/qWjcGn45xrwLaiV.png)

所以我们得看看这个 `_$iL` 方法，传入了一个非常长的字符串，打断点进去看看，果然生成了 `_$PV`，是一个 725 位的数组：

![](https://s2.loli.net/2022/06/30/F9flpiCVXKcHOjJ.png)

接下来在扣代码的过程中，你会经常遇到一个变量，在本文中是 `_$sX`：

![](https://s2.loli.net/2022/06/30/9VpsxQRHyTkYPn6.png)

有没有很熟悉？这个值就是我们前面拿到的 `$_ts` 变量，在开头就可以看到是将 `window.$_ts` 赋值给了 `_$sX`：

![](https://s2.loli.net/2022/06/30/mf8tD9bWBQPJXpc.png)

继续走，会走到以下逻辑中：

![](https://s2.loli.net/2022/06/30/v5XogiWEnwmx31q.png)

这里会遇到六个数组，他们都已经有值了，所以我们得找到他们是咋来的，任意搜索其中一个数组名称，会找到定义和赋值的地方：

![](https://s2.loli.net/2022/06/30/maJAnI7eld5sT3w.png)

![](https://s2.loli.net/2022/06/30/pBXaZPHe85yDxiK.png)

赋值明显是调用了 `_$rv` 方法，再搜 `_$rv` 方法，发现是开头就调用了：

![](https://s2.loli.net/2022/06/30/7qAMQsSBGJ4K9mL.png)

后续没有什么特别的，一直单步，最后有个 `join('')` 操作，就生成了假 Cookie：

![](https://s2.loli.net/2022/06/30/tDb1kliY63CZXSH.png)

接下来是生成 Cookie 的名字 `FSSBBIl1UgzbN7N80T`，然后将 Cookie 赋值给 `document.cookie`，然后又向 `localStorage` 里面的 `$_ck` 赋了个值，`localStorage` 的内容可以直接复制下来，没有太大影响。

![](https://s2.loli.net/2022/06/30/ngkaizOL6MqKUXY.png)

## 真 Cookie 生成逻辑

单步跟真 Cookie，在本文中也就是 `_$ZN(768, 1);`，可以看到开始进入了无穷无尽的 if-else 控制流：

![](https://s2.loli.net/2022/06/30/Y9KruCtVHFoBx8l.png)

这里本地应该怎样处理呢？我的做法是以 `_$Hn` 和其值命名函数，`function _$Hn768(){}` 就表示所有走 768 号控制流的方法，继续跟，生成真 Cookie 的方法基本上在 747 号控制流，后续我们主要以 747 号控制流的各个步骤来看，747 号控制流扣出来的代码大致如下：

![](https://s2.loli.net/2022/06/30/fV1Hum7tgGFKPMb.png)

### 取假 Cookie

单步跟 747 号控制流，会有个进入第 709 号控制流的步骤，会取先前生成的假 Cookie，经过一系列操作之后返回一个数组：

![](https://s2.loli.net/2022/06/30/EFfQYHNDIKci7go.png)

![](https://s2.loli.net/2022/06/30/bOtLNQJHjgWmU5n.png)

至此我们在本地同步扣的代码，如果正常的话，返回的数组也应该是一样的（后续的数据就不一样了，有一些时间戳之类的参数参与运算）：

![](https://s2.loli.net/2022/06/30/SluzHAydLeaVW6C.png)

### 自动化工具检测

继续跟 747 号控制流，会进入 268 号控制流，接着进入 154 号控制流，这里面会针对自动化工具做一些检测，如下图所示：

![](https://s2.loli.net/2022/07/01/aiSEHAsfcqNvj7Q.png)

![](https://s2.loli.net/2022/07/01/8bBTeaFQWvGdnxM.png)

这里定义了一个变量 `_$iL`，检测不通过就是1，后续又把这个变量赋值给了 `_$aW`，所以我们本地保持一致，也为 false 即可（其实我们不用自动化工具的话，这一段检测就不用管直接返回 false 就行）：

![](https://s2.loli.net/2022/07/01/m45PrkF39LwBG7O.png)

### 20 位核心数组

继续跟 268 号控制流，会进入 668 号控制流，668 号控制流就两个操作，一是生成一个 16 位数组，二是取 `$_ts` 里面的 4 个变量，加到前面的 16 位后面，组成一个 20 位数组，这 20 位数组的最后 4 位是瑞数核心，其中的映射关系搞错了请求是通不过的，在五代中这部分的处理逻辑会更加复杂。

![](https://s2.loli.net/2022/07/01/qmJMSR9ZBEWK6Hc.png)

![](https://s2.loli.net/2022/07/01/4XOFlf8JeBtixVq.png)

这里不是单纯的取 `$_ts` 里的键值对，你在扣代码的时候，你也许会发现怎么本地到这里取值的时候，取出来的不是数字，而是字符串呢？就像下面这种情况：

![](https://s2.loli.net/2022/07/01/q3hLJSIyzmGM4wd.png)

实际上我们最开始得到的 `$_ts` 值，是经过了二次处理的，我们以第一个 `_$sX._$Xb` 为例，直接搜索 `_$sX._$Xb`，可以发现这么一个地方：

![](https://s2.loli.net/2022/07/01/A5sGR6iwa2JCZ3t.png)

很明显这里给 `_$sX._$Xb` 重新赋值了一遍，我们可以看到等号右边，先取了一次 `_$sX._$Xb`，其值为 `_$Rm`，这和我们初始 `$_ts` 里面对应的值是一样的，然后我们就得再看看 `_$sX["_$Rm"]` 又是何方神圣，直接搜索发现是开头赋值了一个方法，通过调用这个方法来生成新的值：

![](https://s2.loli.net/2022/07/01/1uLAdx5fgMyKzb7.png)

另外其他三个值也是同样的套路，赋值的代码分别为：

```
_$sX._$Xb = _$sX[_$sX._$Xb](_$BH, _$DP);
_$sX._$oI = _$sX[_$sX._$oI](_$ZJ, _$DS)
_$sX._$EN = _$sX[_$sX._$EN]();
_$sX._$D9 = _$sX[_$sX._$D9](_$iL);
```

实际上应该是：

```
_$sX._$Xb = _$sX["_$Rm"](_$BH, _$DP);
_$sX._$oI = _$sX["_$Nw"](_$ZJ, _$DS)
_$sX._$EN = _$sX["_$Uh"]();
_$sX._$D9 = _$sX["_$ci"](_$iL);
```

进一步来说，实际上是：

```
_$sX._$Xb = _$1k(_$BH, _$DP);
_$sX._$oI = _$jH(_$ZJ, _$DS)
_$sX._$EN = _$9M();
_$sX._$D9 = _$oL(_$iL);
```

静态分析没问题，我们可以先固定下来，但是实际应用当中这些值都是动态的，那我们应该怎么处理呢？先来多看几个对比一下找找规律：

![](https://s2.loli.net/2022/07/01/3HWaiShXfZ1mI4P.png)

![](https://s2.loli.net/2022/07/01/8pLnG93rUey7JSI.png)

可以发现每次对应的位次都不一样，但是实际上相同位置的方法点进去都是一样的，也就是说，变的只有方法名和变量名，实现的逻辑是不变的，所以我们只要知道了这四个值分别对应的位置，就能够拿到正确的值，在本地，我们就可以这样做：

1、先利用正则匹配出这四个值，如：[`_$sX._$Xb`, `_$sX._$oI`, `_$sX._$EN`, `_$sX._$D9`]；

![](https://s2.loli.net/2022/07/01/DqM4xPGiLKoFYe9.png)

2、再匹配出 VM 代码开头的 20 个赋值的语句，如：`_$sX._$RH` = `_$wI`; `_$sX._$i5` = `_$n5`; 等；

![](https://s2.loli.net/2022/07/01/GOXnUKul2C5m9Ik.png)

3、然后通过 `$_ts` 取这四个值对应的值，相当于：`_$sX._$Xb` = `_$ts._$Xb` = `_$Rm`；然后再找这四个值所定义的方法在 20 个赋值语句中的位置，相当于：查找 `_$sX._$Rm` = `_$1k`; 在 20 个赋值语句中的位置为 7（索引从 0 开始）

![](https://s2.loli.net/2022/07/01/uMWteX1CZ2YKAhS.png)

4、我们知道了这四个方法在 20 个赋值语句中的位置，那么我们直接匹配本地对应位置的名称，进行动态替换即可，当然前提是咱们本地已经扣了一套代码出来了：

![](https://s2.loli.net/2022/07/01/ta8HCN2VbsXpi3c.png)

![](https://s2.loli.net/2022/07/01/6EFZ85o3GYctPkS.png)

经过这样处理后，就能够保证这四个值的准确性了。

### 其他用到 $_ts 值的地方

除了上面说的 20 位数组里用到了 4 个 `$_ts` 的值以外，还有其他地方有 7 个值也用到了，直接搜索就能定位，这 7 个值相对较简单，每次都是固定取 `$_ts` 里面的第 2、3、4、15、16、17、19 位的值，同样的，找到对应位置，进行动态替换即可：

![](https://s2.loli.net/2022/07/01/lTzHNMZecYFr3Kq.png)

## 注意事项

特别注意 VM 代码开头，会直接调用执行一些方法，某些变量的值就是通过这些方法生成的，当你一步一步跟的时候发现某些参数不对，或者没有，那么就得注意开头这些方法了，可能一开始就已经生成了。

![](https://s2.loli.net/2022/07/01/fMXko7cYtsQeSxR.png)

## 后缀 MmEwMD 生成逻辑

后续的其他 XHR 请求中，都带有一个后缀，这个后缀的值同样是由 JS 生成的，每次都会变化，当然不同网站，后缀名不一定都是一样的，本例中是 `MmEwMD`，先下一个 XHR 断点，当 XHR 请求中包含了 `MmEwMD=` 时就断下，然后刷新网页：

![](https://s2.loli.net/2022/07/01/OmKyNM7D1UJWACi.png)

可以看到后传入 `l.open()` 的 URL 还是正常的，断下后到 `l.send()` 就带有后缀了，再看 `l.open()` 其实就是 `xhr.open()`，明显和正常的有区别，同样这个方法也在 VM 代码里，应该是重写了方法，可以和正常的做对比：

![](https://s2.loli.net/2022/07/01/5a9TnEvsc2JbI8M.png)

跟到 VM 代码里去看看，经过了 `_$sd(arguments[1])` 方法就变成了带有后缀的完整链接了：

![](https://s2.loli.net/2022/07/01/uReo9BEOqsQrpMY.png)

跟进 `_$sd` 方法，前面都是对 url 做一些处理，后面有个进入第 779 号控制流的流程，实际上就是原来我们生成 Cookie 的步骤，跟一下就行了。

![](https://s2.loli.net/2022/07/01/ewQMIyKaG2JoNnC.png)

## 善用 Watch 跟踪功能

![](https://s2.loli.net/2022/07/01/z5nkOXENYcpCVr4.png)

开发者工具的 Watch 功能能够持续跟踪某个变量的值，对于这种控制流很多的情况，设置相应的变量跟踪，能够让你知道你现在处于哪个控制流中，以及生成的数组的变化，不至于跟着跟着不知道到哪一步了。

## 结果验证

如果整个流程没问题，代码也扣得正确，携带正确的 Cookie 和正确的后缀，就能成功访问：

![](https://s2.loli.net/2022/07/01/ypLR4X5sq87DucT.png)

![](https://s2.loli.net/2022/07/01/l2RzbIPnBU94GA1.png)
