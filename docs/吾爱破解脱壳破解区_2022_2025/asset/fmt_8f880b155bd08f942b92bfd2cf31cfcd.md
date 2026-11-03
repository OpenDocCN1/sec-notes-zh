
# [Web逆向] 一个在线php加密文件的解密

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1912173
- 抓取时间：2026-11-01T22:33:22

免费加密网站：`aHR0cDovL2RlcGhwLm5ldC9lbmNyeXB0Lmh0bWw=`

## 加密前

加密前代码：

```
```

运行如下：

![image.png](https://attach.52pojie.cn/forum/202404/11/011115b42nzz49kps3kg89.png)

`image.png`（13.92 KB，下载次数：0，2024-4-11 01:11 上传）

## 加密后

给php加密：

![image.png](https://attach.52pojie.cn/forum/202404/11/011507lkmnq9ro9szqfnw9.png)

`image.png`（50.82 KB，下载次数：1，2024-4-11 01:15 上传）

加密后代码：

```
```

访问：

![image.png](https://attach.52pojie.cn/forum/202404/11/011806b8epq1chp8fh3tpz.png)

`image.png`（13 KB，下载次数：0，2024-4-11 01:18 上传）

## 分析过程

过程：

代码格式化，发现一堆奇形怪状的函数：

![image.png](https://attach.52pojie.cn/forum/202404/11/011910bgozjncjonj7iioq.png)

`image.png`（63.11 KB，下载次数：1，2024-4-11 01:19 上传）

二话不说，输出来看看：

```
```

关键代码长这样：

```
echo $Kyddly." ".$ltJjKV." ".$sCFCgN." ".$dhGfsR." ".$Kyddly."

 ";
```

刷新网页得结果：

![image.png](https://attach.52pojie.cn/forum/202404/11/012114arxxcx6chc6mmcm0.png)

`image.png`（15.77 KB，下载次数：0，2024-4-11 01:21 上传）

即 `.$Kyddly` 是 `base64_decode`。

果断把 `eval` 后面的 `$Kyddly` 替换成 `base64_decode`，

运行，正常，发现猜测没错：

![image.png](https://attach.52pojie.cn/forum/202404/11/012520ozw3aff3hnaw7gf3.png)

`image.png`（15.53 KB，下载次数：0，2024-4-11 01:25 上传）

`eval` 函数改成 `echo`：

![image.png](https://attach.52pojie.cn/forum/202404/11/012657wewwswkeweyjs1sa.png)

`image.png`（64.74 KB，下载次数：0，2024-4-11 01:26 上传）

吾爱破姐专用变成了看不懂的字符串：

![image.png](https://attach.52pojie.cn/forum/202404/11/012750xsq4nmn3nmdhmrsl.png)

`image.png`（39.84 KB，下载次数：0，2024-4-11 01:27 上传）

全部内容如下：

```
$ijsHOv="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";eval('?>'.$Kyddly($ltJjKV($sCFCgN($ijsHOv,$dhGfsR*2),$sCFCgN($ijsHOv,$dhGfsR,$dhGfsR),$sCFCgN($ijsHOv,0,$dhGfsR))));
```

## 解码验证

猜测 `$ijsHOv` 还是 `base64_decode` 你就错了，`base64_decode` 得到乱码，我就不掩饰不拐弯墨迹了，直接来干活！

他后面还有一段 `eval('?>'.$Kyddly($ltJjKV($sCFCgN($ijsHOv,$dhGfsR*2),$sCFCgN($ijsHOv,$dhGfsR,$dhGfsR),$sCFCgN($ijsHOv,0,$dhGfsR)))`，去掉 `eval`，设 `$text = $ijsHOv`，把这些函数用上面输出的名称替换后得：

```
base64_decode(strtr(substr($text,52*2),substr($text,52,52),substr($text,0,52)))
```

写点代码：

```
$pattern = '/="([^"]+)"/';

preg_match_all($pattern, $decodestr1, $matches);
$text = implode('', $matches[1]);
echo $text;
echo "
 ";
$decodestr2 = strtr(substr($text,52*2),substr($text,52,52),substr($text,0,52));

//运行看看
echo $decodestr2;
echo "
再 base64_decode看看
";
echo base64_decode($decodestr2);
```

![image.png](https://attach.52pojie.cn/forum/202404/11/015733mbn6btsloonu6zsb.png)

`image.png`（135.62 KB，下载次数：0，2024-4-11 01:57 上传）

## 浏览器查看

一开始用骨锅，看了好久看不出啥，这里得用360浏览器看看。

看到的内容：

```
```
