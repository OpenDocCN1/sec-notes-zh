
```
try {
    return (String) getStaticMethod(context, iArr, "com.zj.wuaipojie2024_2.A", "d", Context.class, String.class).invoke(null, new Object[]{context, str});
} catch (Exception e) {
    Log.e(TAG, "咦，似乎是坏掉的dex呢！");
    e.printStackTrace();
    return "";
}
```

大致是加载dex后要调用`getStaticMethod(context, , , )`这样去解密。

上面的代码中有`R.array.A_offset`，相应的在apk中也找到了`R.array.B_offset`。

接下来把刚才拿到的`2.dex`改名为`decode.dex`，再次用Frida等手段（当时都整到晚上了，具体过程有点乱没记录下来）

拿到解密后的`2.dex`， 这个就是`A`和`B`都解密过了，用NP打开找到`B.d`

![03-b-d.jpg](https://attach.52pojie.cn/forum/202402/25/130145uiun3uno7m6ihh9e.jpg)

`03-b-d.jpg` (16.68 KB, 下载次数: 1) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzUzM3xhYTI3OGYxOHwxNzkzNTcyNjM4fDE2ODA4NnwxODkzMTM0&nothumb=yes)

2024-2-25 13:01 上传

可以找到flag的算法：比如我`UID=1581363`就是

`Utils.md5(Utils.getSha1("0485312671581363".getBytes()))`

这个就可以直接用frida脚本写一下，运行直接打印出来

这道题解了差不多一整天时间，还是基础不扎实啊！

## 【06-Windows高级题】高级题我基本是解不出来的，只记录一下思考过程

两个文件，照样先查壳、脱壳`crackme2024.exe`, `UPX`改，按照论坛教程 ESP大法，用`x64dbg`成功解压缩

`crackme2024service.exe`， `Themida`壳 研究了好久，最后在github上找到一个python脱壳脚本，试了一下可以解出来但exe有7M多，可以运行

先分析`crackme2024`, `x64dbg`加载，找字符串下断点:

![06-str.JPG](https://attach.52pojie.cn/forum/202402/25/131405nfs056t0i50nf4iq.jpg)

`06-str.JPG` (207.12 KB, 下载次数: 1) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzU0MHw2MzRhN2JiN3wxNzkzNTcyNjM4fDE2ODA4NnwxODkzMTM0&nothumb=yes)

2024-2-25 13:14 上传

随便输入`uid`，`sn`，断下来，往下分析发现有个比较 `0x23`，得到`sn`长度35位

![06-uid-sn.JPG](https://attach.52pojie.cn/forum/202402/25/131601xauee02zef88dhfz.jpg)

`06-uid-sn.JPG` (113.63 KB, 下载次数: 1) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzU0MnxkZmY2ZTQwYnwxNzkzNTcyNjM4fDE2ODA4NnwxODkzMTM0&nothumb=yes)

2024-2-25 13:16 上传

再往下分析发现它调用了COM组件进行`sn`验证，看来得分析`service`了

对`service`静态分析了一圈 没有什么头绪

又用`x32dbg`对`service`动态调试了一圈，仍然没有头绪

由于对COM技术不懂 做不下去了。

分析失败。

## 【07-Android高级题】知道肯定解不出来，简单试一试

`jadx`分析一下，发现通过`System.loadLibrary("52pojie")`加载`lib52pojie.so`，再调用里面的`checkSn(md5(uid), sn)`函数进行验证。

![07-jadx.jpg](https://attach.52pojie.cn/forum/202402/25/132157c53mmky5z10f5w0n.jpg)

`07-jadx.jpg` (132.11 KB, 下载次数: 1) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzU0NHw5YWVmYTg3Y3wxNzkzNTcyNjM4fDE2ODA4NnwxODkzMTM0&nothumb=yes)

2024-2-25 13:21 上传

找到`lib52pojie.so`，用`IDA`看到许多怪名字函数 用十六进制编辑器查看到有`llvm obfuscator`字样

![07-llvm.jpg](https://attach.52pojie.cn/forum/202402/25/132800jooztxox44tet1ua.jpg)

`07-llvm.jpg` (26.35 KB, 下载次数: 0) [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY3NzU0NnxiZmY1Mjk1M3wxNzkzNTcyNjM4fDE2ODA4NnwxODkzMTM0&nothumb=yes)

2024-2-25 13:28 上传

百度了一下，`ollvm`这个东西太难搞了，跟着教程 又是准备`unidbg`又是搞[Kali环境](https://www.52pojie.cn/thread-661779-1-1.html)

搞了好几天，无奈在`unidbg`加载so时始终出错(`/ by zero`)，找不到原因，无法继续下去。

分析失败！

## 【08-Web题】

每年的web题会有许多有意思的地方，可以说是技术与艺术的完美结合~解题过程也是既紧张又开心的

下面按flag顺序列出：

flag1{52pj2024}，视频有个点状图，中间黑色部分是字符。将速度调到0.5X慢慢看 就能看出来

flag2{xHOpRP}, 打开二维码网址后，F12打开开发人员选项，点“网络”，刷新一次，在http响应头中找找，有`X-flag2`字样

flag3{GRsgk2}, 在视频开头的一段雪花点图形序列中隐含，可以反复慢放能看出来 也可以用`ffmpeg`逐帧截图分析

flag4{YvJZNS}, 打开网址后，开发人员选项，同样在“网络”中可看到请求了`flag4_flag10.png`，下载即可看到

flag5{P3prqF}, 网页源码中有注释 `flag5_flag9`，下面有一长段代码 中间有断断续续的明文字母 提取即可

flag6{20240217}, 看源码，得知要求 `md5(某数字)` = '1c450bbafad15ad87c32831fa1a616fc'; 直接从`cmd5.com`查询得到

flag7{Djl9NQ}, 打开作者的github网址，上面有“删除不小心发出来的flag”字样。于是去找旧版本的 `commit`，在`wip`下找到

flag8{OaOjIK}, 直接玩游戏或通过 溢出 方法攻击得到

flag9{KHTALK}, 网页源码中，有一长段代码 将`flag5`删除，剩余的以点开头进行换行，就会组成一个ASCII图形

flag10{6BxMkW}， 前面下载的`flag4_flag10.png`，发现大小比同尺寸png大了一倍。试着用`HEX`编辑器将 宽度`05 00`改为`0A 00` 保存再打开查看 ，会隐约看到字符，这里我猜错了好几次 后来用百度找工具才看清楚

flag11{HPQfVF}， 像是拼图游戏，在源码里找到`--var1`和`--var2`，改变数值慢慢试

flag12{HOXI}, 查看代码后，直接穷举[www.52pojie.cn](https://www.52pojie.cn)，每100万个输出一下`log`免得好像没反应 (开始从0起往上加，发现等了好久，所以改成从`4294967295`往下减 很快就出来了)

```
WebAssembly.instantiateStreaming(fetch('flag12.wasm'))
        .then(({instance}) => {
            const get_flag12 = (secret) => {
                let num = instance.exports.get_flag12(secret);
                let str = '';
                while (num > 0) {
                    str = String.fromCodePoint(num & 0xff) + str;
                    num >>= 8;
                }
                return `flag12{${str}}`;
            }
            document.querySelector('button').addEventListener('click', (e) => {
                e.preventDefault();
                let i = 4294967295;
                for( ; i>0; i-- ) {
                     let s = get_flag12(i);
                     if( s.length >8 ) {
                                console.log(i); console.log(s);
                document.querySelector('#result').textContent = s;
                                break;
                     }
                    if( i % 1000000 == 0 ) console.log(i);
               }
                
            });
        });
```

得到 `4005161829` ，输入即可

flagA：修改`cookie`，将`flagA`的值给`uid`，再`fetch('auth/uid/')`

flagB：玩游戏，`v50`后提示 溢出，试了一下有提示“购买后钱变多，有bug”，通过尝试发现大约是`QWORD`数据(8字节)，通过计算 `0x10000000000000000 / 999063388 = 18464037713`，在购买数量输入后成功！

     同样方法可以大量买上面的道具，包括`flag8`。只不过用`0x10000000000000000 / 10000` 后不行。后来试了用 `0x60000000000000000 / 10000` 才行

flagC：用网站提示图片就会发现，有几张提示类别正确、位置错误的信息。下载图片，用`ps`将类别正确的图片多次尝试，先找类别，再找位置，很容易出来

## 【总结】

靠运气偶有得手，多学习才是正道！

祝大家龙年天天开心，事事顺心！
