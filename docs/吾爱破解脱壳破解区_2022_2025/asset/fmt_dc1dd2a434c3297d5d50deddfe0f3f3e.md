
```js
o = $_BFo()["encrypt1"](de["stringify"](t["$_EJV"]), t["$_CCHU"]()),
i = p["$_HEt"](o)
```

# `o` 参数生成

`t["$_EJV"]` 是一个 `Object`，保存了一些验证码的信息。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045224881-a8a00faa-c999-4509-bc9b-f8f3f3d0e8f9.png)

`t["$_CCHU"]()` 从上面的 `i` 参数知道是一个 aes key。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045336577-b762c456-b2ad-44bb-bc62-2dcab746c4eb.png)

然后分析下 `encrypt1` 函数的逻辑，简单查看代码逻辑后基本断定是 aes 加密算法，而且从各种函数关键词来看基本不会是魔改 aes。

主动调用该函数确认是标准的 aes，并且采用 cbc 模式/pkcs7 填充，`iv` 默认是 `0000000000000000`。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045342890-107639c1-2d92-4a5b-917b-3e57fee26b85.png)

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045348329-d9823f6d-857b-41d1-bdae-7c2f4499ba51.png)

完成 aes 加密之后对每个 32 位的数字做如下转换。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045359209-10833d25-8a80-407d-b7f8-43218a32dbec.png)

其作用是将 32 位的数字转为四个 8 位的数字。

# `i` 参数生成

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045365965-78136415-a863-45ff-a8d6-6c5377b5018b.png)

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045371203-8e09e509-a948-4f35-a171-70492a955aac.png)

`i` 由 `p["$_HEt"]` 函数输入 `o` 经过一系列操作之后由 `res+end` 组成，其中 `$_HCK` 函数有多处将三个 8bit 数字换算成 24bit 数字然后转换为四个字符的逻辑，似乎是 base64 算法变体。

`t` 函数是取指定 `t` 二进制位上对应的值组合成新的数字。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045383228-c0cecd65-df1a-4c76-bab0-99de1f3681b2.png)

`$_GJI` 函数作用是取对应数字的字符，取不到则用 `.` 代替。`$_GAp="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789()"`，与标准 base64 需要的字符串差异在最后两个字符由 `+/` 变为了 `()`。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045388929-de4cdb03-ee47-47f9-a293-6c964caf2bd1.png)

`$_GCK=7274496`（二进制：`11011110000000000000000`），`$_GDu=9483264`（`100100001011010000000000`）

`$_GEk=19220`（`100101100010100`），`$_GFY=235`（`11101011`）

实现标准 base64 需要的四个数字应该是：

`0xfc0000(111111000000000000000000)`，

`0x3f000(111111000000000000)`，

`0xfc0(111111000000)`，

`0x3f(111111)`，

跟标准 base64 实现有点不同。

## 1.3 总结分析

`r` 参数保存 aes 密钥，采用 RSA 加密后取 16 进制，`i` 是 aes 加密数据后采用 base64 编码的字符串。

# 第二次 `w` 生成逻辑分析

第二个 `w` 同样是由 `fullpage.xxx.js` 生成的。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045398052-fb4a4744-e531-496b-a423-ec3a5b174bdc.png)

`n["w"] = t["$_CEAN"]`

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045404880-0eaeed84-fa53-436f-a03c-5c7621147523.png)

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045411343-bf0302ea-c7ba-4493-a488-7d994aad3ccb.png)

跟踪下 `t["$_CEAN"]` 的生成，可以发现同样是采用 aes 加密，不过加密的数据不同，这个 `r` 参数当中有很多未知的参数，似乎是一些环境检测的参数。

```json
{"lang":"zh-cn","type":"fullpage","tt":"M6(*((1Sj((sM((","light":-1,"s":"c7c3e21112fe4f741921cb3e4ff9f7cb","h":"321f9af1e098233dbd03f250fd2b5e21","hh":"39bd9cad9e425c3a8f51610fd506e3b3","hi":"09eb21b3ae9542a9bc1e8b63b3d9a467","vip_order":-1,"ct":-1,"ep":{"v":"9.1.9-r8k4eq","te":false,"$_BBp":false,"ven":"Google Inc. (NVIDIA)","ren":"ANGLE (NVIDIA, NVIDIA GeForce RTX 3070 (0x00002488) Direct3D11 vs_5_0 ps_5_0, D3D11)","fp":null,"lp":null,"em":{"ph":0,"cp":0,"ek":"11","wd":1,"nt":0,"si":0,"sc":0},"tm":{"a":1711849377534,"b":1711849377617,"c":1711849377617,"d":0,"e":0,"f":1711849377539,"g":1711849377539,"h":1711849377539,"i":1711849377539,"j":1711849377539,"k":0,"l":1711849377547,"m":1711849377614,"n":1711849377615,"o":1711849377620,"p":1711849377751,"q":1711849377751,"r":1711849377752,"s":1711849378121,"t":1711849378121,"u":1711849378121},"dnf":"dnf","by":2},"passtime":8258,"rp":"d2d182b3ce6cf55f590e9ec11c9b1635","captcha_token":"549902629","otpj":"jm4jwcx7"}
```

搜一下其中关键词例如 `"hh"` 找到代码逻辑位置。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045420990-a2257801-cb98-43aa-9cb6-e8a6df80a73c.png)

## 2.1 `s` 参数分析

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045428665-8ece0828-4325-4108-b0a8-1c07c3035f67.png)

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045435893-7a916d41-117c-4333-aa2b-6c298c6e6426.png)

关键代码 `H(p["$_HD_"](t))`，其中 `p["$HD"]` 根据上文分析已知是 base64 变体算法，`t` 是一个字符串，`H` 函数是标准 md5 算法。

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045444558-1302e0bd-48af-4567-a17c-7b8fe24dafc3.png)

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045451529-9e77621f-e706-46c7-8dde-cff18dce1fcf.png)

`t` 这个字符串来自 `$_BICT` 函数生成，`$_BICT` 函数用于收集鼠标操作事件并采用 base64 编码，base64 字符映射为 `"()*,-./0123456789:?@ABCDEFGHIJKLMNOPQRSTUVWXYZ_abcdefghijklmnopqrstuvwxyz~"`。

收集到的鼠标事件信息会进行一系列转换成二进制字符串后进行 base64 编码，核心代码如下：

```js
for (var t = [], n = [], r = [], o = [], i = 0, s = e["length"]; i 
```

其中 `t` 是鼠标事件数组，例如：`["move", "move", "down", "move", "up"]`，鼠标事件包括：move/down/up/scroll/focus/blur/unload。

`n` 是鼠标事件发生的毫秒级时间戳数组（例如：`[1711952404794, 1711952404794, 1711952404798, 1711952404798, 1711952404798]`）。

如果传入该函数的 `e` 参数每个元素是一个长度 3 的数组，即每个元素只记录了 `[事件类型, [x 坐标, y 坐标], 毫秒级时间戳]` 信息，例如：`["move", [123, 456], 1711952404794]`、`["down", [123, 456], 1711952404798]`，则 `r` 和 `o` 参数分别是记录 x 和 y 坐标的数组。

## 2.2 `h` 参数分析

```js
n = i["$_BJDB"]["$_BICT"]();

["h", H(p["$_HD_"](n))];
```

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045463753-3e61c324-5165-4a04-894d-b95ae6bafca8.png)

`i["$_BJDB"]["$_BICT"]` 函数作用是检测一个 `Object` 对象（该对象为空 `Object`）指定的属性是否存在，存在则返回该属性的值，不存在则返回 `-1`，由此组成一个数组并用 `"magic data"` 字符串 `joi` 拼接这个数组得到。检测的属性列表如下：

```js
["textLength","HTMLLength","documentMode","A","ARTICLE","ASIDE","AUDIO","BASE","BUTTON","CANVAS","CODE","IFRAME","IMG","INPUT","LABEL","LINK","NAV","OBJECT","OL","PICTURE","PRE","SECTION","SELECT","SOURCE","SPAN","STYLE","TABLE","TEXTAREA","VIDEO","screenLeft","screenTop","screenAvailLeft","screenAvailTop","innerWidth","innerHeight","outerWidth","outerHeight","browserLanguage","browserLanguages","systemLanguage","devicePixelRatio","colorDepth","userAgent","cookieEnabled","netEnabled","screenWidth","screenHeight","screenAvailWidth","screenAvailHeight","localStorageEnabled","sessionStorageEnabled","indexedDBEnabled","CPUClass","platform","doNotTrack","timezone","canvas2DFP","canvas3DFP","plugins","maxTouchPoints","flashEnabled","javaEnabled","hardwareConcurrency","jsFonts","timestamp","performanceTiming","internalip","mediaDevices","DIV","P","UL","LI","SCRIPT","touchEvent"]
```

## 2.3 `tt` 参数分析

```js
e = i["$_CAAG"]["$_BIBg"]();
```

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045485551-a1f5677e-5495-4ae8-be5e-81da8e6bfefb.png)

`tt` 参数作用是根据服务器返回的 `c` 和 `s` 参数截取鼠标事件 base64 编码后的字符串，可能是用于校验。

`c` 是一个纯数字字符串，每两个字符代表一个 16 进制数字；`s` 是一个数组，取其中下标 `0`、`2`、`4` 三个数字。

## 2.4 `hh` 参数分析

```js
["hh", H(n)]
```

`hh` 参数是 `n` 的 md5 值，跟 `h` 参数区别是 `h` 参数是 base64 编码 `n` 之后取 md5 值。

## 2.5 `hi` 参数分析

```js
var e = t["$_BJDB"]["$_BIBg"]();

t["$_CCFY"] = e;

["hi", H(i["$_CCFY"])]
```

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045507290-5c734e9a-67cc-4162-b7f7-2252d9854892.png)

`t["$_BJDB"]["$_BIBg"]` 函数的作用与 2.2 `h` 参数分析中的 `n` 变量的生成相似，只是组成的数组用 `"!!"` 字符串 `join` 拼接得到。

## 2.6 `ep` 参数分析

```js
["ep", i["$_CEDy"]() || -1]
```

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045524427-eb428b51-9fa1-4ec1-a14c-ea8f17cb1cbc.png)

![](http://sbaydf3b2.hd-bkt.clouddn.com/1712045534486-40e173d0-e764-48db-aa78-c05971ffe4b8.png)

`ep` 参数应该是一些环境检测的汇总数据，各属性可以在代码中看出来，汇总如下：

| 属性名 | 作用 |
| --- | --- |
| `v` |  |
