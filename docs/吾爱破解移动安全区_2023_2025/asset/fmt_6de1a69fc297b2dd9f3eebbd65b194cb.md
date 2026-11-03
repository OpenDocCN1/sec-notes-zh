
![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115165229260.png)

- 那这个函数刚好也有设置值的位置；

  ![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115165147790.png)

- 这个函数的目的应该就是设置这个返回值解密后的数据了；

- 并且参数我们也都解决了，算法主要就是`des`解密，和`init`的魔改函数是同一对；

### 3.4 `nativeGetSign`

- 首先把`unidbg`的`call`打开，别忘了`call_nativeSetToken`；到这个算法其实就简单了，直接`hook`对应的`md5`函数先；

```
emulator.attach().addBreakPoint(module.base + 0x1C18);
```

- 第一次端下来是之前获取应用签名的位置，第二次是这样的：

  ![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115165827209.png)

- 这个着实有些眼熟，是`setToken`函数解密出来的后半部分，原来是用在这里了；继续往下跟；

  ![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115170053663.png)

- 这个结果有点熟悉；

  ![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115170110537.png)

- 就是主动调用的结果，所以算法很简单，就是标准的`md5`，参数的话是这种组成形式；

  ![](https://www.xiayutian.xyz/usr/assets/android/%E5%BF%AB%E5%AF%B9%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251115170222076.png)

- 第一部分在前面详细的说了来源，`f061614527adba41881c85c075b6bf52`这一部分是前面`t2BOaKTul1` `md5`结果，后面一部分是传进来的参数，至此算法分析完毕；

## 4. 总结

- 整体的算法不算太难，但是分析过程属实花了不少时间，`des`扣表的时候请仔细，因为我就踩了不久的坑；

- 最后对整体我们做了什么做个总结，针对三个方面：入参、返回值、算法；

  - `nativeInitBaseUtil`函数：

    - 入参：`context`、`cuid`，总体来说`cuid`与设备有关，可能是设备id；

    - 返回值：一组16进制结果，后续会用做`nativeSetToken`函数的入参；

    - 算法：魔改`des`加密、`md5`等；`des`的入参也分析的很透彻了；

  - `nativeSetToken`函数：

    - 入参：`cuid`、`nativeInitBaseUtil`返回值、一个请求的响应结果；

    - 返回值：没有返回值，函数用于给全局变量赋值，此结果会在`nativeGetSign`函数使用；

    - 算法：魔改`des`解密、`md5`等；与`init`函数紧密联系；

  - `nativeGetSign`函数：

    - 入参：`base64`形式的明文，前面也分析过；

    - 返回值：最终的`sign`；

    - 算法：`md5`等；

- 总体的算法难度非常小，但综合性非常强，分析每一个明文的来源才是这篇文章的目的；

- by：2025-11-15；
