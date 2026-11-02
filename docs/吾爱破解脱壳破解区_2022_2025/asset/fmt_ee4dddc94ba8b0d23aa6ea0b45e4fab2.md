
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxMXwzOTMwM2E0YnwxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:47 上传

然后刷新网页

```
const pt = new ut;
const mt = lt("nt:main/lib/js/ntPlayer")
  , z = class extends nt.DefaultConfig.loader {
    constructor(e) {
        super(e);
        d(this, "_superLoad");
        this._superLoad = super.load.bind(this)
    }
    load(e, i, a) {
        if (e.keyInfo) {
            const p = a.onSuccess;
            a.onSuccess = function(E, g, c, v) {
                const y = E.data
                  , $ = new DataView(y);
                if (z._revise) {
                    const b = z._revise;
                    $.setInt32(0, $.getInt32(0) ^ b[0]),
                    $.setInt32(4, $.getInt32(4) ^ b[1]),
                    $.setInt32(8, $.getInt32(8) ^ b[2]),
                    $.setInt32(12, $.getInt32(12) ^ b[3]),
                    E.data = $.buffer
                }
                p(E, g, c, v)
            }
        }
        this._superLoad(e, i, a)
    }
    static setRevise(e) {
        z._revise = e
    }
}
;
```

![](...)

`15 Snipaste_2023-11-02_15-39-19.png` (161.74 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxMnwwMmRjM2FmNnwxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:48 上传

```
z._revise

[3854078970, 2917115795, 3887476043, 3350876132]
```

发现该变量，在下面有用到，于是全局搜索，共4处引用，挨个下断，再次刷新网页

![](...)

`16 Snipaste_2023-11-02_15-43-47.png` (85.48 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxM3xmYzkwNWI3MXwxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:48 上传

发现这里的入参是这些值，那么从堆栈里往上跟，就能找出来了。

![](...)

`17 Snipaste_2023-11-02_15-45-38.png` (73.36 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxNHw0ZDE3ODNiNXwxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:49 上传

```
        const t = document.querySelector(".player[nt-main-skey]")
          , e = document.querySelector("#app-key")
          , i = document.querySelector(".end-tips")
          , a = document.querySelector("[nt-buy-vip]")
          , l = parseInt(t.getAttribute("nt-video-id") || "0")
          , p = t.getAttribute("nt-main-poster");
        this.existSrt = t.getAttribute("nt-srt") == "1",
        this.skey = t.getAttribute("nt-main-skey") || "";
        const E = JSON.parse(`[${e.dataset.keys}]`);
        t.removeAttribute("nt-main-poster"),
        t.removeAttribute("nt-main-skey"),
        t.removeAttribute("nt-srt"),
        e.remove(),
        this.playLog = new Et,
        this.pageState = JSON.parse(localStorage.getItem(st) || "{}"),
        tt("pageState: %o", this.pageState),
        this.player = new _t(t,{
            poster: p || "",
            logo: ct,
            logoWidth: 120,
            autoplay: !0,
            muted: this.pageState.muted || !1,
            volume: this.pageState.volume || 1,
            playbackRate: this.pageState.playbackRate || 1,
            enablePlaybackRate: !0,
            enableCue: this.existSrt,
            revise: E
        });
```

![](...)

`18 Snipaste_2023-11-02_15-47-04.png` (179.87 KB, 下载次数: 3)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxNXw0NjVjYTEyM3wxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:49 上传

于是找到了关键位置

```
e = document.querySelector("#app-key")

...

const E = JSON.parse(`[${e.dataset.keys}]`);
```

`querySelector()` 方法返回文档中匹配指定 CSS 选择器的一个元素。

于是到 HTML 里搜索 `app-key` 看看

![](...)

`19 Snipaste_2023-11-02_15-50-07.png` (32.37 KB, 下载次数: 3)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxNnxmZTBlOTgyMHwxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:49 上传

果然找到了

接着在 Nodejs 补环境，完成脱机调用，也就是还原解密过程。

### 3> 纯算法 Nodejs

![](...)

`20 Snipaste_2023-11-02_15-55-56.png` (145.32 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MzMxN3w2MDVlOGY2OHwxNzkzNTczMDMyfDE2ODA4NnwxODUxOTU1&nothumb=yes)

2023-11-2 19:50 上传

```
var z = {

    '_revise' : [3854078970, 2917115795, 3887476043, 3350876132] // app-key 的值

}

// console.log(z)

var key = [128, 245, 244, 139, 212, 166, 215, 255, 208, 226, 106, 4, 240, 217, 14, 134]; // 网页 .key文件返回的加密值

let buffer = new ArrayBuffer(16);

let data = new Uint8Array(key);

let view = new DataView(buffer);

for (let i = 0; i 
```

封装成解密函数

```
function decrypt(app_key, en_key) {
    var z = {
        '_revise': app_key
    }

    let buffer = new ArrayBuffer(16);
    let data = new Uint8Array(en_key);
    let view = new DataView(buffer);
    for (let i = 0; i 
```

### 3> `Blob`和`ArrayBuffer`是什么鬼？

- 最早是数据库直接用 `Blob` 来存储二进制数据对象，这样就不用关注存储数据的格式了。在 web 领域，`Blob` 对象表示一个只读原始数据的类文件对象，虽然是二进制原始数据但是类似文件的对象，因此可以像操作文件对象一样操作 `Blob` 对象。
- `ArrayBuffer` 对象用来表示通用的、固定长度的原始二进制数据缓冲区。我们可以通过 `new ArrayBuffer(length)` 来获得一片连续的内存空间，它不能直接读写，但可根据需要将其传递到 `TypedArray` 视图或 `DataView` 对象来解释原始缓冲区。实际上视图只是给你提供了一个某种类型的读写接口，让你可以操作 `ArrayBuffer` 里的数据。`TypedArray` 需指定一个数组类型来保证数组成员都是同一个数据类型，而 `DataView` 数组成员可以是不同的数据类型。

`TypedArray` 视图的类型数组对象有以下几个：

- `Int8Array`：8 位有符号整数，长度 1 个字节。
- `Uint8Array`：8 位无符号整数，长度 1 个字节。
- `Uint8ClampedArray`：8 位无符号整数，长度 1 个字节，溢出处理不同。
- `Int16Array`：16 位有符号整数，长度 2 个字节。
- `Uint16Array`：16 位无符号整数，长度 2 个字节。
- `Int32Array`：32 位有符号整数，长度 4 个字节。
- `Uint32Array`：32 位无符号整数，长度 4 个字节。
- `Float32Array`：32 位浮点数，长度 4 个字节。
- `Float64Array`：64 位浮点数，长度 8 个字节。

- `Int8Array`：1 个字节的有符号整数类型，范围在 -128 ~ 127 之间；
- `Uint8Array`：1 个字节的无符号整数类型，范围在 0 ~ 255 之间；
- `Uint16Array`：2 个字节的无符号整数类型，范围在 0 ~ 65535 之间；
- `Int16Array`：2 个字节的有符号整数类型，范围在 -32768 ~ 32767 之间；
- `Uint32Array`：4 个字节的无符号整数类型，范围在 0 ~ 4294967295 之间；
- `Float32Array`：4 个字节的单精度浮点数类型；
- `Float64Array`：8 个字节的双精度浮点数类型。

`DIV` 标签的 `app-key` 值

```
Uint32Array
[3854078970, 2917115795, 3887476043, 3350876132]

Uint8Array
[229, 184, 147, 250, 173, 223, 167, 147, 231, 182, 45, 75, 199, 186, 79, 228]
```

加密 `key` 文件返回的是：

```
Uint8Array
[128, 245, 244, 139, 212, 166, 215, 255, 208, 226, 106, 4, 240, 217, 14, 134]

hex
80f5f48bd4a6d7ffd0e26a04f0d90e86
```

真实 `key`：

```
Uint8Array
[101, 77, 103, 113, 121, 121, 112, 108, 55, 84, 71, 79, 55, 99, 65, 98]

十六进制（hex）
654d67717979706c3754474f37634162

字符串
eMgqyypl7TGO7cAb
```

### 3> `Uint32Array` 转换 `Uint8Array`

```
let uint32Arr = new Uint32Array([1902595429, 1819310457, 1330074679, 1648452407]);

// 创建新的Uint8Array，长度为Uint32Array的4倍
let uint8Arr = new Uint8Array(uint32Arr.length * 4);

// 遍历Uint32Array并转换每个元素为4个字节序列
for (let i = 0; i > 8 * j & 0xFF;
  }
}

console.log(uint8Arr); // 输出转换后的Uint8Array
// [101, 77, 103, 113, 121, 121, 112, 108, 55, 84, 71, 79, 55, 99, 65, 98]
```

### 3> 纯算法 Python

```
def int_to_bytes(n):

    # 使用 '>I' 表示大端序（Most Significant Byte First）的无符号整数格式

    return n.to_bytes((n.bit_length() + 7) // 8, 'big')

def decrypt(app_key,en_key):

    int_list = app_key

    byte_list = [int_to_bytes(i) for i in int_list]

    app_key = b''.join(byte_list)
```
