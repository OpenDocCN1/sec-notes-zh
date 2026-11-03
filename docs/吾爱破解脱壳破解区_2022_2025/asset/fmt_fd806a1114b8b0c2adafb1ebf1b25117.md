
```
case 5157:
  S = b = [U + 64 * ye];
  li = 19041;
  break;
case 5184:
  R = k = f;
  if (bo = (P = (fe = fe  (Y = (A = 215 | (bo = !hi)) 
```

瀏覽器替換用的是多層`switch`嵌套的版本，主要處理了三目运算符、逗号表达式等阻礙調試的東西，看大佬的那篇文章有移除自執行函數這步，但在227移除自執行函數會有變量作用域問題，因此我就保留了（這也導致`ast`要多判斷一些條件）。

（可以看到多層嵌套`switch`控制流，若要向上跟，要考慮多個狀態變量，十分麻煩!!!!）

![](https://attach.52pojie.cn/forum/202403/19/211644vndd9vbzujzudjvj.png)

Untitled15.png (50.96 KB, 下载次数: 3)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mzg2OHxjYzllOTM3MXwxNzkzNTcyNDc3fDE2ODA4NnwxOTAzMTQx&nothumb=yes)

2024-3-19 21:16 上传

## 4> 瀏覽器跟值

主要目標是找到軌跡相關的邏輯。

跟進`o.__fy.getFYToken`，會走到`i(40, e)`這裡。

![](https://attach.52pojie.cn/forum/202403/19/211646ivnnd8nmwj14ljh1.png)

Untitled16.png (10.14 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mzg2OXw5NzhiYWYwMHwxNzkzNTcyNDc3fDE2ODA4NnwxOTAzMTQx&nothumb=yes)

2024-3-19 21:16 上传

單步走幾步，發現`s`數組，看它的內容很像是一個環境的特徵數組，`s[90]`還特別像是與軌跡有關，經測試後發現確實如此（所謂測試就是鼠標多滑幾下或少滑幾下，看看這個`s[90]`的長度是否符合預期），接下來看看這個`s`數組的某幾項：

![](https://attach.52pojie.cn/forum/202403/19/211648gta543hh1yahwmms.png)

Untitled17.png (29.09 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mzg3MHwwYjlhMzEwMHwxNzkzNTcyNDc3fDE2ODA4NnwxOTAzMTQx&nothumb=yes)

2024-3-19 21:16 上传

![](https://attach.52pojie.cn/forum/202403/19/211650kj18l2x2i81jqrjl.png)

Untitled18.png (18.57 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mzg3MXxhNWNhYjE2YnwxNzkzNTcyNDc3fDE2ODA4NnwxOTAzMTQx&nothumb=yes)

2024-3-19 21:16 上传

- `s[5]`：鼠標點擊前的軌跡數組長度 - 1（計`mousedown`那一下的軌跡）

- `s[46]`：與`s[5]`類似，但不計`mousedown`那一下的軌跡，因此比`s[5]`小1，由`i(25)`觸發

- `s[26]`：前兩項是`WebGLRenderingContext`的東西，第3項的`"undefined"`是觸發`DeviceOrientationEvent`事件而來，第4、5項是觸發`MouseEvent`而來，鼠標在頁面不同位置時，對應的`target`（`html`頁面中的某個元素）也不一樣，取的是`target["id"]`

![](https://attach.52pojie.cn/forum/202403/19/211652pr33i6glo6glmyog.png)

Untitled19.png (13.29 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mzg3MnxiMGU4YWU2N3wxNzkzNTcyNDc3fDE2ODA4NnwxOTAzMTQx&nothumb=yes)

2024-3-19 21:16 上传

- `s[23]`：由`ScriptProcessorNode`的`onaudioprocess`事件生成，而`onaudioprocess`事件是通過觸發全局的`mousedown`而來，具體值的來源是`AnalyserNode.prototype.getFloatFrequencyData`

- `s[48]`：與`timestamp`有關的數組

- `s[91]`：`promise`的回調（看堆棧就知道）

- `s[52]`：同樣是某個`promise`的回調觸發的

- `s[3]`：由`FocusEvent`觸發

- `s[90]`：最重要的軌跡數組，有以下幾個要點：

  - 由`mousedown`、`mousemove`觸發（全局正常會有2個`mousemove`、`mousedown`事件，要用第2個），每次會生成一個軌跡

  - 軌跡主要取`MouseEvent.clientX`、`MouseEvent.clientY`、`timestamp`、`MouseEvent.pageY`、`MouseEvent.pageX`等，並進行了異或等咚悖�因此直接看`s[90]`會與軌跡值對不上

  - 這個軌跡數組並非鼠標點擊後才開始`push`，而是只要在頁面上動就會一直在`push`，因此軌跡數組可能要分成點擊前和點擊後

  - 有兩種軌跡，分別是`length`為17和`length`為18的，嘗試過找為什麼有兩種不同的軌跡，但放棄了，最後我在本地手動構建軌跡時發現我的`s[90]`同樣有這兩種，就沒有深究了

- 快速獲取軌跡的方法：在`i`函數一開始下條件斷點

    ```
    o instanceof MouseEvent && ( window.slideArr ? window.slideArr.push({"pageX":o.pageX, "pageY":o.pageY, "clientX":o.clientX, "clientY":o.clientY}) : window.slideArr = [], false)
    ```

- `s[36]`：由滑塊的與鼠點擊事件觸發

## 3> 本地調用結果

![](https://attach.52pojie.cn/forum/202403/19/211655r84o3igz4d53g8f7.png)

Untitled20.png (102.31 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY4Mzg3M3w2MGE3NGEyY3wxNzkzNTcyNDc3fDE2ODA4NnwxOTAzMTQx&nothumb=yes)

2024-3-19 21:16 上传

## 3> 參考

感謝大佬們的分享

- [//blog.csdn.net/weixin_44862184/article/details/135159526](https://blog.csdn.net/weixin_44862184/article/details/135159526)

- [//www.52pojie.cn/forum.php?mod=viewthread&tid=1818486&extra=&highlight=226&page=1](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1818486&extra=&highlight=226&page=1)

- [//zhuanlan.zhihu.com/p/673660020](https://zhuanlan.zhihu.com/p/673660020)

- [//mp.weixin.qq.com/s/s6e4TmOJHdYTsJ4p1ZJGDw](https://mp.weixin.qq.com/s/s6e4TmOJHdYTsJ4p1ZJGDw)

- [//mp.weixin.qq.com/s/almQM3N9VMBk86-Jv2LPow](https://mp.weixin.qq.com/s/almQM3N9VMBk86-Jv2LPow)
