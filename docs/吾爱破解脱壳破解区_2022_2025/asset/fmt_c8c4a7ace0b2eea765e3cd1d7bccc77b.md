
```
{
    pow_msg: _ᖄᖆᖗᕺ + h,
    pow_sign: p
}
// 很明显搞清楚,让我们一步步还原。

_ᖄᖆᖗᕺ = _ᕶᖆᕷᕵ + _ᕶᖃᖁᕹ(111) + _ᖀᖄᕴᖄ + _ᕸᖙᕹᕷ(111) + _ᖄᕿᖚᕺ + _ᕶᖃᖁᕹ(111) + _ᕹᕿᖆᖀ + _ᕶᖃᖁᕹ(111) + _ᕶᕴᕹᕶ + _ᕶᖃᖁᕹ(111) + _ᕶᖉᖃᕾ + _ᕸᖙᕹᕷ(111) + _ᕿᕵᖆᕾ + _ᕸᖙᕹᕷ(111);

// 简单还原一下：
_ᖄᖆᖗᕺ = _ᕶᖆᕷᕵ + '|' + _ᖀᖄᕴᖄ + '|' + _ᖄᕿᖚᕺ + '|' + _ᕹᕿᖆᖀ + '|' + _ᕶᕴᕹᕶ + '|' + _ᕶᖉᖃᕾ + '|' + _ᕿᕵᖆᕾ + '|';
// 那么这个参数就是几个值同 '|' 拼接而成的
// _ᕶᖆᕷᕵ --> 1 定死
// _ᖀᖄᕴᖄ --> 0 定死
// _ᖄᕿᖚᕺ --> 'md5' 看情况，他有三种模式，有md5,sha1,sha256 这个是决定 p 的生成的。
// _ᕹᕿᖆᖀ --> '2025-04-16T14:29:07.516729+08:00' 一个时间戳还原代码后面写。
// _ᕶᕴᕹᕶ --> captcha_id
// _ᕶᖉᖃᕾ --> lot_number
// _ᕿᕵᖆᕾ --> ''

h = _ᕺᖗᕾᖗ['guid']() // 跟栈你会发现他就是 e() + e() + e() + e()
function e() {
    var _ᖚᖆᖀᖃ = _ᖀᕴᖘᕺ.$_Dr()[0][7];
    for (; _ᖚᖆᖀᖃ !== _ᖀᕴᖘᕺ.$_Dr()[3][6]; ) {
        switch (_ᖚᖆᖀᖃ) {
        case _ᖀᕴᖘᕺ.$_Dr()[3][7]:
            return (65536 * (1 + Math['random']()) | 0)['toString'](16)['substring'](1);
            break
        }
    }
}
//这个函数看着那么吓人，其实就执行了一步就是(65536 * (1 + Math['random']()) | 0)['toString'](16)['substring'](1);
//所以h函数就可以写为：
function e(){
    return (65536 * (1 + Math['random']()) | 0)['toString'](16)['substring'](1);
}
h = e() + e() + e() + e()

// 接着看p，这个你看到分支你就笑吧。搞清楚入参，带入算算，如果是标准加密就爽了，下面这个都不用扣算法了。
// 入参就是pow_msg
switch (_ᖄᕿᖚᕺ) {
    case 'md5':
        p = (new (_ᖉᕹᕺᖀ[_ᕸᖙᕹᕷ(9)][_ᕸᖙᕹᕷ(771)]))[_ᕶᖃᖁᕹ(793)](l);
        break;
    case 'sha1':
        p = (new (_ᖉᕹᕺᖀ[_ᕸᖙᕹᕷ(9)][_ᕸᖙᕹᕷ(789)]))[_ᕶᖃᖁᕹ(793)](l);
        break;
    case 'sha256':
        p = (new (_ᖉᕹᕺᖀ[_ᕶᖃᖁᕹ(9)][_ᕶᖃᖁᕹ(778)]))[_ᕸᖙᕹᕷ(793)](l)
}

// 我们走入分支直接测试即可
// (new (_ᖉᕹᕺᖀ[_ᕸᖙᕹᕷ(9)][_ᕸᖙᕹᕷ(771)]))[_ᕶᖃᖁᕹ(793)]('12345') -->  '827ccb0eea8a706c4c34a16891f84e7b' 标准小写md5-32位

// 那么这部分加密我们就完成了
```

继续分析参数：

- `So89` 定值，我刷新了几次没变
- `2e424091` --> 变值，自己本身就是，这个参数就在上面可以直接看到

继续分析，本人还是继续从代码角度分析：

```
var i = (0,_ᕸᖙᕹᕷ[_ᖄᕿᖚᕺ(68)])(_ᕶᖆᕷᕵ[_ᖀᖄᕴᖄ(693)], _ᕶᖆᕷᕵ[_ᖄᕿᖚᕺ(461)]),

r = (0,_ᕸᖙᕹᕷ[_ᖄᕿᖚᕺ(68)])(_ᕶᖆᕷᕵ[_ᖀᖄᕴᖄ(652)], _ᕶᖆᕷᕵ[_ᖀᖄᕴᖄ(461)])
```

规律如下图：

![](https://attach.52pojie.cn/forum/202504/16/185631xp0rpyq2vybvgx2r.png)

`10.png` (61.77 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3MTc5MHxiZWFkZTM1NXwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5MHxiZWFkZTM1NXwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

打上断点，让我们看看他的逻辑

```
var i = _ᕸᖙᕹᕷ['getStringByIndexes'](_ᕶᖆᕷᕵ['lot'],_ᕶᖆᕷᕵ['lotNumber']),
r = _ᕸᖙᕹᕷ['getStringByIndexes'](_ᕶᖆᕷᕵ['lotRes'],_ᕶᖆᕷᕵ['lotNumber'])

// lot_number 为入参
function _ᖚᖆᖀᖃ(_ᕶᖉᖃᕾ, _ᕶᕴᕹᕶ) {
    // var _ᖄᕿᖚᕺ = _ᖀᕴᖘᕺ.$_Cl
    //     , _ᕶᖆᕷᕵ = ["$_DBF_"].concat(_ᖄᕿᖚᕺ)
    //     , _ᖀᖄᕴᖄ = _ᕶᖆᕷᕵ[1];
    // _ᕶᖆᕷᕵ.shift();  这部分你都不用看，没啥用，ob混淆解混用的
    var _ᕹᕿᖆᖀ = _ᕶᖆᕷᕵ[0];
    return _ᕶᖉᖃᕾ[_ᖄᕿᖚᕺ(48)](function(_ᖚᖆᖀᖃ) {
        var _ᕶᖉᖃᕾ = _ᖀᕴᖘᕺ.$_Cl
            , _ᖄᕿᖚᕺ = ["$_DCAl"].concat(_ᕶᖉᖃᕾ)
            , _ᕶᖆᕷᕵ = _ᖄᕿᖚᕺ[1];
        _ᖄᕿᖚᕺ.shift();
        var _ᖀᖄᕴᖄ = _ᖄᕿᖚᕺ[0];
        return _ᖚᖆᖀᖃ[_ᕶᖉᖃᕾ(48)](function(_ᖚᖆᖀᖃ) {
            var _ᕶᖉᖃᕾ = _ᖀᕴᖘᕺ.$_Cl
                , _ᖄᕿᖚᕺ = ["$_DCFk"].concat(_ᕶᖉᖃᕾ)
                , _ᕶᖆᕷᕵ = _ᖄᕿᖚᕺ[1];
            _ᖄᕿᖚᕺ.shift();
            var _ᖀᖄᕴᖄ = _ᖄᕿᖚᕺ[0];
            var _ᕹᕿᖆᖀ = _ᖚᖆᖀᖃ[_ᕶᖉᖃᕾ(64)]
                , _ᖉᕹᕺᖀ = _ᕹᕿᖆᖀ[0]
                , _ᕺᖗᕾᖗ = 1
```

放个图，你可以看看我是如何猜想和思考的。

![](https://attach.52pojie.cn/forum/202504/16/185633lhv9jh1fwvfwvee7.png)

`11.png` (99.63 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3MTc5MXw1OWNkZGRhNHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5MXw1OWNkZGRhNHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

我的猜想是它按照固定数组在字符串中取值。让我们着手试试。结果如下：

![](https://attach.52pojie.cn/forum/202504/16/185635gjtw1oz646ww18vy.png)

`12.png` (70.42 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3MTc5Mnw2NzNiZjhjMHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5Mnw2NzNiZjhjMHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

那么 `r` 也是，结果如下：

![](https://attach.52pojie.cn/forum/202504/16/185637qksqajjbnffqspz6.png)

`13.png` (26.75 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3MTc5M3w1ZDc3Y2I0NXwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5M3w1ZDc3Y2I0NXwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

那么入参已经全部弄清楚了。接下来就是加密。还是一样层层分析。

## 逆向算法分析

`_ᖆᕿᖄᖀ['default'](_ᕹᖀᖃᖙ['default']['stringify'](_ᕶᖉᖃᕾ), _ᖉᕹᕺᖀ)`

这是全文最难的地方。

![](https://attach.52pojie.cn/forum/202504/16/185638yc8bu494pt884ubn.png)

`14.png` (158.62 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc3MTc5NHw4Yjg0MDU0OXwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5NHw4Yjg0MDU0OXwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

你看看我找了什么，`key`、`iv`???

![](https://attach.52pojie.cn/forum/202504/16/185638b31kqj71k9jxx4mj.png)

`15.png` (80.89 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3MTc5NXwzM2ExNDI5OHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3MTc5NXwzM2ExNDI5OHwxNzkzNTcxMDMzfDE2ODA4NnwyMDI0ODAw&nothumb=yes)

2025-4-16 18:56 上传

这不会是一个标准的ase吧？？？不急，先猜想，后验证，大不了不对在扣呗。有的是机会。

把相关信息保存一下,我们猜测是：

```
猜想加密算法2 --> ase

ase :  模式 cbc

       key e() + e() + e() + e()随机

       iv 0000000000000000
```

我怀着好奇把 `new (_ᕶᖃᖁᕹ[_ᖄᕿᖚᕺ(9)])` 输出看了看，欸，这不是rsa的标志吗(每次结果都不一样我就在怀疑了，当然也可能是时间戳或者其他算法)？但是我看到了 `65537` 这是一个常用的 RSA 公钥指数。而且，它的数组跟了大数里面跟了一个小数37，正好是前面37的个数，这里就相当于表示这个大整数 `n` 拆分为 37 个段。

猜归猜，我们还是要搞清楚整体流程：

```
// 初始化前面的加密算法之后，我们一步步看看干了什么，我把核心流程抽取出来了
var o = _ᖀᖄᕴᖄ(878) === _ᕺᖗᕾᖗ[_ᖀᖄᕴᖄ(641)] // true
    , a = _ᕺᖗᕾᖗ[_ᖄᕿᖚᕺ(641)]  // '1' 
    , _ = _ᖆᖚᖉᕾ[a]['asymmetric']['encrypt'](_ᖂᖈᗕᕾ); // 这里把ase的key，放到加密算法1里面加密
while (o && (!_ || 256 !== _[_ᖀᖄᕴᖄ(84)]))  // false 不走
    _ᖂᖈᗕᕾ = (0,
    _ᕴᕺᖙᕷ[_ᖄᕿᖚᕺ(58)])(),
    _ = (new (_ᕶᖃᖁᕹ[_ᖄᕿᖚᕺ(9)]))[_ᖄᕿᖚᕺ(955)](_ᖂᖈᗕᕾ);

var u = _ᖆᖚᖉᕾ[a]['symmetrical']['encrypt'](_ᕶᖉᖃᕾ, _ᖂᖈᗕᕾ);  // ase的key 和最开始的入参计算加密，生成字节数组
return (0,
_ᕴᕺᖙᕷ['arrayToHex'])(u) + _ // 两个加密结果相加
```

这里结合之前的分析，我们猜测是流程是：

- `_` 是 随机 key 通过 rsa 加密
- `u` 是 之前的入参和随机 key 进行 aes 加密
- 最后将两个结果合并返回。

第一步我们只需要把自己生成的rsa给他替换了，看看能不能过就知道是否是正确的(这个方法很常用哦)。我测过了，是可以过的。代码如下：

```
import random

from Crypto.Util.number import bytes_to_long, long_to_bytes

n_segments = {

    欸嘿

}

# 还原 n（拼接成一个大整数）

n = 0

for i in range(n_segments["t"] - 1, -1, -1):

    n = (n > 6))

            result.insert(1, 0x80 | (code & 0x3F))

        else:

            result.insert(0, 0xE0 | (code >> 12))

            result.insert(1, 0x80 | ((code >> 6) & 0x3F))

            result.insert(2, 0x80 | (code & 0x3F))

    return result

# ---- 构造带 PKCS#1 padding 的明文块 ----

def rsa_pkcs1_v15_pad(plaintext: str, total_length: int = 128) -> bytes:

    data = reverse_utf8_bytes(plaintext)

    ps_length = total_length - len(data) - 3  # 3 = 0x00 0x02 0x00

    if ps_length  整数 M

c = pow(m, e, n)                         # 执行 C = M^e mod n

cipher_hex = hex(c)[2:]                 # 转十六进制去掉 '0x' 前缀

# 可选补0，补齐256位 hex 字符

if len(cipher_hex) % 2 != 0:

    cipher_hex = '0' + cipher_hex

print("加密前（明文）:", plaintext)

print("Padding 后字节（十六进制）:", padded_bytes.hex())

print("加密后（十六进制）:", cipher_hex)
```

第二步，看看这个ase是否猜测正确。我们先简单编写一些aes代码，如下：
