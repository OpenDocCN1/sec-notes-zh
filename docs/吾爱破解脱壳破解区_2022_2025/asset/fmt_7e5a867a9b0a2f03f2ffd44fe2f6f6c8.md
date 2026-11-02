
```
    for (var r = encrypted, o = r.ciphertext.words, i = r.ciphertext.sigBytes, s = [], a = 0; a >> 2] >>> 24 - a % 4 * 8 & 255;

        s.push(c);

    }

    return s;

}
```

### 4> 进一步处理 `l`

最后一步 `e = DWYi[ymDv(1137)](l)`，将 `l` 的值经过了 `tc_t` 这个方法进行处理，就会得到最终 Request Payload 的一部分。

![](https://s1.ax1x.com/2023/04/13/ppvghtI.png)

跟进这个 `tc_t` 方法，又是熟悉的 `return e["res"] + e["end"]`，同样和某验其他产品一样的。

![](https://s1.ax1x.com/2023/04/13/ppvg4ht.png)

跟到处理 `e` 的这个方法里，最后返回的是 `{"res": a, "end": s}`，没啥特别的，直接扣即可，这里注意和某验其他产品里的方法有些小区别，里面有些常量的值是不一样的，最开始我直接复用了其他产品的方法，发现结果是错的。

![](https://s1.ax1x.com/2023/04/13/ppvgo1f.png)

自此整个流程分析完毕，最终 `e + h[AUJ_(1173)]` 的值与 Request Payload 的值一致。

![](https://s1.ax1x.com/2023/04/13/ppvgbng.png)

![](https://s1.ax1x.com/2023/04/13/ppvgTc8.png)

### 3> 结果验证

![](https://s1.ax1x.com/2023/04/13/ppvg7jS.png)
