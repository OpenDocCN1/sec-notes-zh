
# [Android 原创] WYY快播1.6正式版破解（代码混淆）

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1950101`
- 抓取时间: `2026-11-01T16:42:12.265Z`

app：网易云快播1.6正式版(`https://wwb.lanzoub.com/iHCan23t1gxa`)
工具：`jadx`、`Burp`、`frIDA`、雷电模拟器
方法：修改网络响应 难点：代码混淆、动态控制流（`while`中套多个`case`的`switch`）

## app界面

![](https://attach.52pojie.cn/forum/202407/31/190631faraefsfahf671ze.png)

## 尝试

### 一：搜索关键字

1. 搜卡密、提交、有问题、请输入等等均无合适结果，可以说是无论搜界面中的啥，都是没有结果

![](https://attach.52pojie.cn/forum/202407/31/201018qualqf31pahbafa5.png)
![](https://attach.52pojie.cn/forum/202407/31/201034c5xffmy44o4ez5yg.png)
![](https://attach.52pojie.cn/forum/202407/31/201103szbag6kbbtj9e6gy.png)
![](https://attach.52pojie.cn/forum/202407/31/201203od1nz1cwk2vq22c1.png)

### 二：查询活动控件

1. 直接`adb shell dumpsys activity | findstr "mResumedActivity"`查看当前界面信息，然后再编写代码查看是谁调用了这个；但是其实这个输出的是主页信息，并不是让你输入卡密的对话框信息，所以根本没有

![](https://attach.52pojie.cn/forum/202407/31/201753dzon05qykq795by1.png)

2. 我想着用开发助手的布局查看试试，但是收费；之后我想着用mt管理器中Activity记录试试看；结果发现是`com.stardust.autojs.inrt.SplashActivity` > `cd.o4`，我其实看不懂这个（有没有大佬能解释一下），但是还是尝试去搜了一下`cd.o4`，然后去hook它里面的方法，但是发现都没调用

![](https://attach.52pojie.cn/forum/202407/31/202523zwyqe6qiqyz68644.png)
![](https://attach.52pojie.cn/forum/202407/31/202810xmpy811g1myw8l1q.png)

### 总结尝试

正常来说，拿到软件后一般都是从最简单的搜关键字符串开始，但是此app混淆了代码，根本无从下手，然后从活动控件下手，想着找到显示输入卡密的对话框，直接将它hook掉就行，但是我对安卓活动控件的获取缺乏相关知识，导致无从下手，最后只能尝试抓包

## 正文

### 一：抓包

1. 直接输入`aaa`后在`burp`上看抓包信息，发现是个`get`请求，并且都是明文，而`secret=aaa`就是我输入的卡密

![](https://attach.52pojie.cn/forum/202407/31/204626xrrorr3uzq6xx35g.png)

2. 所以直接去`jadx`中搜`secret`看看，有47条，但是都是无用的，之后我又尝试搜了一下网址信息，发现都是空的

![](https://attach.52pojie.cn/forum/202407/31/205215t2ege0k552j1xjjo.png)
![](https://attach.52pojie.cn/forum/202407/31/205247pgdjb1hjxlb666dl.png)
![](https://attach.52pojie.cn/forum/202407/31/205307zmllmevapnvmm68i.png)
![](https://attach.52pojie.cn/forum/202407/31/205326vp15vlg15gpxzti5.png)

3. 之后我就直接搜`app_id`；发现只有2个，网络请求中携带了id那么它肯定调用了`return app_id`去获取id并放入发送请求中

![](https://attach.52pojie.cn/forum/202407/31/205838uvur59dm57qq2dxu.png)
![](https://attach.52pojie.cn/forum/202407/31/210117mpuuplnicykezene.png)

4. 编写hook代码验证是否调用（感谢@debug_cat 大佬为我解答`jadx`可以直接复制成`frida`代码，我之前是直接手动复制“ &#1766;&#1766;&#1767;&#1765; ”这个吊符号，导致hook找不到目标）；发现果然app发包时就调用了这个

![](https://attach.52pojie.cn/forum/202407/31/210521fjytv5az5z4vc15z.png)
![](https://attach.52pojie.cn/forum/202407/31/210915zffxcxssrwf91vvx.png)

### 二：回溯网络请求

1. 查看是谁调用了`m2144`这个方法；因为混淆，一眼根本看不出来是谁，只能一个个试，但是运气不错，第二个就是

![](https://attach.52pojie.cn/forum/202407/31/212139njvgt08jjorjo4gt.png)

2. 我们直接hook看看是不是；可以看到`public static java.lang.String f(java.lang.String r33, int r34, java.util.Mapjava.lang.String, java.lang.String> r35)`这个方法第一参数就是网络请求的地址，第二个是0，第三个是个object,而且它的返回值就是服务器的响应信息；说明这个方法就是用来发送网络请求并且返回网络响应

![](https://attach.52pojie.cn/forum/202407/31/212335dybqtgyabtf1zfga.png)
![](https://attach.52pojie.cn/forum/202407/31/212611x3b6d0dpmkz0ykdz.png)

3. 继续跟进看看是谁调用了它；我们继续hook它看看都有啥有用的信息；`m5181`就是获取请求地址，0，还有输入的卡密信息，然后调用`f()`返回服务器的响应信息；我们最终的目的是找到判断服务器响应的代码，然后修改它，继续回溯

![](https://attach.52pojie.cn/forum/202407/31/213511pxcs63y6qq0u01z0.png)
![](https://attach.52pojie.cn/forum/202407/31/213824m1t8981c6ftaubot.png)

### 三：回溯服务器响应的判断代码

1. 查看是谁调用了`m5181`，可以发现有两个调用，但是第一个就是我们刚才才看的`cd.m8.f()`中的，我们直接看第二个

![](https://attach.52pojie.cn/forum/202407/31/214805x6w6vtq99iycz464.png)

2. 在`public static /* synthetic */ void e(String str, Map map, f3 f3Var)`中`m5181`的值被传给`str4`,之后有两处关键点对`str4`（也就是服务器的响应信息）进行判断和处理，我们先查看第一次`str4`做判断的地方(`C0120.m3149(str4)`)

![](https://attach.52pojie.cn/forum/202407/31/215921nbqpvpvpkeytp1zt.png)

3. 直接跟进，卧槽！！！布尔值！！！难不成就是它？？？直接hook让它返回`true`试试！！！其实我当时看见布尔的那一刻，我高兴坏了，以为这个就是判断服务器响应信息的，没想到啊，它是用来判断是否输入卡密的

![](https://attach.52pojie.cn/forum/202407/31/220740ecszz8wyo7gs71r4.png)
![](https://attach.52pojie.cn/forum/202407/31/221116a3rid838ixsli3zl.png)
![](https://attach.52pojie.cn/forum/202407/31/221329dsweiw5lhlssssp3.png)

4. 排除了一个，就只剩最后一个 `C0118.m3008(f3Var, str4)`，这里肯定就是对服务器的响应进行提取然后判断；直接跟进看看

![](https://attach.52pojie.cn/forum/202407/31/221857tnd1bdlot11dhbzj.png)

5. 结果发现`m3008`里面先进行一个判断，然后再执行`((f3) obj).c(obj2)`，跟进去后发现`f66`是定值`-365`，之后hook发现`C0145.m4569()`的值是定值`553`，说明它一定执行了`((f3) obj).c(obj2)`，我们直接跟进`c`看看

![](https://attach.52pojie.cn/forum/202407/31/222425jwycxlzl9qdbxsfx.png)
![](https://attach.52pojie.cn/forum/202407/31/222519rkfjmzfllududlly.png)
![](https://attach.52pojie.cn/forum/202407/31/222600y2shv2v2zh7ezu22.png)

6. 在`c`这里，通过分析这里是100%要被调用的，但是hook后显示没有调用；猜测应该是`m3008`中的`obj`参数是个接口啥的对`obj`进行处理；（技术有限，我的分析就止步于此了，欢迎各位大佬继续分析）

![](https://attach.52pojie.cn/forum/202407/31/223412jsckxkn4kc3sec0e.png)
![](https://attach.52pojie.cn/forum/202407/31/223637ycboaffnbfmamnba.png)
![](https://attach.52pojie.cn/forum/202407/31/224054x1qvoq8e2yxya0ib.png)

### 四：大胆尝试

1. 思考本质：

这个app的本质不就是获取用户输入的卡密然后发送给服务器进行验证，然后服务器返回结果，之后app再从服务器返回的结果中来判断是否成功

2. 猜想：

如果我直接修改服务器返回的结果呢？我管它服务器返回啥，我直接自己去自定义一个成功的返回值不就行了嘛

3. 实践：

3-1. 修改`msg`的值为“卡密正确”，可以看到在只修改`msg`值的时候，app会显示卡密正确，并且没有出现闪退报错等情况，但是也只是改变了消息，并没有实现功能

![](https://attach.52pojie.cn/forum/202407/31/225604ii6isgiik4xkdskt.png)

3-2. 修改`datas`的值，当它的值为一个时是正常的，但是有两个后就报错了，说明`datas`的值可能就是是一个

![](https://attach.52pojie.cn/forum/202407/31/230207qd05xjj7vcehhu0e.png)
![](https://attach.52pojie.cn/forum/202407/31/230318pb405g4fq4fga2a9.png)

3-3. 修改`code`的值，我尝试修改了好几个数字，但都是正常的，并没有报错

![](https://attach.52pojie.cn/forum/202407/31/230602fct8t7cc7qut7qd7.png)

3-4. `ts`的值跟`code`的值一样，修改多次依然正常，而且仔细看就知道它是个时间戳，我们直接修改`status`的值；将`status`的值修改为`1`后，app成功完成[破解](https://www.52pojie.cn/)！！！并且一切功能全部正常使用

![](https://attach.52pojie.cn/forum/202407/31/231020frv2uevln56lujx8.png)
![](https://attach.52pojie.cn/forum/202407/31/231152kere5uxkvslz6zcs.png)

总结：此app的卡密验证就是验证服务器返回值中的`status`的值是否为`1`，如果为`1`，卡密正确，如果为`0`，卡密错误；虽然它做了代码混淆，但是它的包都是明文，而且只判断服务器返回中`status`的值，这就给了我很大的运气完成了这次破解，这也是我首次完成对混淆代码的破解，虽然有很大的运气成分，对于混淆代码，只要慢慢分析，还是可以知道它的逻辑的，然后再慢慢hook不断验证猜想，你也可以完成对混淆代码的破解，感谢支持，大家互相学习学习

![](https://static.52pojie.cn/static/image/smiley/laohu/laohu13.gif)
