
```
hook let UZModuleContext = Java.use("com.uzmap.pkg.uzcore.uzmodule.UZModuleContext");
UZModuleContext["optString"].overload('java.lang.String').implementation = function (str) {
    console.log(UZModuleContext.optString is called: str=${str});
    let result = this["optString"](str);
    console.log(UZModuleContext.optString result=${result});
    return result;
};
```

![](https://picss.sunbangyan.cn/2023/10/26/8253222735c80a7052a06bff672d9237.png)

![](https://picss.sunbangyan.cn/2023/10/26/8e838907ce1da701296e021901f6984d.png)

他向里面添加了 73921 --> 这里是参数的返回值。

![](https://picdl.sunbangyan.cn/2023/10/24/55e37b641b8148aa0788ffa3b41756c6.png)

73921 得到 b65ca9e9f5667139 猜测一下是不是 `md5`

先测试一波：

对比发现就是 `md5` 的中间 16 位

![](https://picdm.sunbangyan.cn/2023/10/24/18007fa6163d7f249a50001cd7eea843.png)

接着咱们 `hook` 一下 `md5` 就得出

加密方法其实就是 `str` `md5` 取 16 位得出。

### 结尾撒花！

就是 `aes` `cbc` `iv` 固定是 `0809060801020609` `key` 是返回值的参数经过 `md5` 后取得中间 16 位得出。
