
# [Android 原创] 某消费金融算法分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2070898
- 抓取时间：2026-11-01T23:54:30

## 南银消金算法分析

### 1. 基础信息

- 版本：7.4.4
- 包名：`cn.com.njxmxbank.mbank`
- 加固：梆梆加固企业版
- 目标：服务 -> 惠聚生活+ -> 更多
- 接口：`POST /xmxappmid/xmxcustexpand__queryHuiJuLife`
- 会有frida检测，我使用的是魔改过的可以直接过，读者可以自行寻找魔改版的frida，应该不严格，server就能直接过；
- 关于壳，fart就可以脱完，在线的也可以脱，自行选择即可；
- 由于众所周知的原因，请以 `-f` 的方式进行重启hook；

### 2. 抓包

- 点击下图位置抓包，因为登录有验证码，懒得去划了，这里的算法是一样的；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104101717786.png)

- 直接抓包就可以，效果如下：

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104101521010.png)

- 可以发现，一共是有两个位置需要分析的，请求体和响应体，都是密文，请求头没有什么东西；对两个位置的密文进行一定程度的分析，首先是请求体；
- 经过对密文的观察，发现其是由三段组合而成的；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104101836996.png)

- 中间由 `|` 隔开，后续按照分段去分析；其次是响应体，很长，但是没有什么特征，需要注意其中包含了很多 `\n` 换行，可以大致猜测这两个都是有 `base64` 编码的参与的，接下来开始算法分析；

### 3. 响应体分析

- 之所以先看这个是因为它是java层的，肯定是先把简单的说了，特征比较明显，搜一下 `passworddata` 看看有没有什么收获；这里提一嘴，如果返回值没有什么特征，就只能去hook一些系统库了，如 `getbytes` 方法等等；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104102148448.png)

- 一共就两处位置，这里去看哪一个不用多说了；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104102231131.png)

- 继续跟进就可以看到具体的算法；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104102417958.png)

- 至于 `key` 也是固定的，点进去就行，这里就直接解密测试了；需要注意，抓包的时候提过了，里面有大量的 `\n`，上一张截图也可以看到是去掉了的，测试的时候自行去掉就行；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104102549979.png)

- 解密出来了，这个没什么可说的；

### 4. 请求体分析

- 接下来分析请求体的算法，这里回顾抓包的时候提到的内容，密文说是分了三段的，后续注意一下；
- 至于定位，同样是直接搜索即可，请读者不要拘泥在定位方式太过简单的思想上，都是需要慢慢积累的，样本很多，定位方式也各有不同，在分析的过程中逐渐积累即可；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104102851079.png)

- 先hook一下看看吧，入参出参整清楚先；

```
function hook_java() {
    Java.perform(function () {
        let PEJniLib = Java.use("com.csii.njaesencryption.PEJniLib");
        PEJniLib["getNativeValue"].implementation = function (str, str2) {
            console.log(`PEJniLib.getNativeValue is called: str=${str}, str2=${str2}`);
            let result = this["getNativeValue"](str, str2);
            console.log(`PEJniLib.getNativeValue result=${result}`);
            return result;
        };

    })
}

hook_java()
```

- 前面说了，由于众所周知的原因，这里需要以 `spawn` 的方式启动，不想hook太多次怎么办？请将hook代码取消掉，目标接口触发前再解开；日志如下：

```
[Pixel 6::cn.com.njxmxbank.mbank ]-> PEJniLib.getNativeValue is called: str={"securityTime":1762243248605,"CHFTimeStamp":"1762243229325","CHFStr":"i6wb6ial3s57dw85","version":"157"}, str2=bcf7d443244521b377b1fb9dd40adfc8421be6e5c83823beb29cb0e9baccb68483bf60f2281ba6a0bd2df03354182d36419bdf8714b391f171ca83843a466802ddcd98004bcf2f8570ef5ac2ec87eb862392bff1d83c440601899704ffbd27711eef309e6963416a3dfb84d693dcb98320fce8e9baf1818a68e62c2d8dbc038f

PEJniLib.getNativeValue result=XL8fSFeF8YsjOhZP7rdHA+RssAH2FF7Ek+6BiVQhfc5ZzGIsBErwH2CZC9xk0nycsXuMu8UWICvDYSnm62svWrWqUazJh8IyodwKn42KSnAWfuyaMC3ohfdXfJ+eP5BGy0b4cg26FaIHilsfes5uTy1mr7kLzX2uoXrumznIGXg=|fb5b1d2V4XMtpsrFduCNnDu21TvIW7mT+WeOxizwzYYQycEgUOLoB38hCWUPtJXZZ44pegqraCaLAF1ZMI+1tmgmxCJa3hhK63e7QsAMiISnfcW83WBKp2X8RtPaAUZjdWznZaafIwPqMSyZS/YxDA==|dZnWjn0DUuNUUwGO9JRGyw==
```

- 接下来可以写一个主动调用，方便后续算法还原；

```
function call_java() {
    Java.perform(function () {
        let PEJniLib = Java.use("com.csii.njaesencryption.PEJniLib");
        let instance_method = PEJniLib.$new();
        let str1 = "{\"securityTime\":1762243248605,\"CHFTimeStamp\":\"1762243229325\",\"CHFStr\":\"i6wb6ial3s57dw85\",\"version\":\"157\"}"
        let str2 = "bcf7d443244521b377b1fb9dd40adfc8421be6e5c83823beb29cb0e9baccb68483bf60f2281ba6a0bd2df03354182d36419bdf8714b391f171ca83843a466802ddcd98004bcf2f8570ef5ac2ec87eb862392bff1d83c440601899704ffbd27711eef309e6963416a3dfb84d693dcb98320fce8e9baf1818a68e62c2d8dbc038f"
        let result = instance_method.getNativeValue(str1, str2);
        console.log(`call_java result=${result}`);
    })
}
```

- 去看看so的情况，名称为：`csii_AESTelecomModule_v1_0`；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104161504057.png)

- 第一反应，这个so很不正常，先来看一个正常的so；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104161538559.png)

- 它们有本质的区别，一般来说代码段ida都会是蓝色的，目标的so并不是，所以它应该是加固了；使用yang神的项目一把梭了；

```
python dump_so.py libcsii_AESTelecomModule_v1_0.so
```

- 得到这样的结果，直接是被修复后的so，免去修复这一步了，非常好用，可以去给一个star；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104161828166.png)

- 现在打开就正常了许多，下一步寻找native方法在哪，去搜索java发现不是静态注册，那就去hook `registernatives`，这里还是用yang神的项目，最近应该是集成了另一种方法，两周前添加的，所以是能hook出来的；如果是以前的脚本是没办法找到位置的，但是这里直接去看 `jni_onload` 更简单，随性就行；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104164236611.png)

- 这里我们按照以往的脚本来hook是没有结果的，因为走的更加底层，用更新后的脚本来hook；

```
[Pixel 6::cn.com.njxmxbank.mbank ]-> lookup_RegisterNative_method("com.csii.njaesencryption.PEJniLib")
fnPtr='0x7959907770 libcsii_AESTelecomModule_v1_0.so!_Z10aesDecryptP7_JNIEnvP8_jobjectP8_jstring' method='private native java.lang.String com.csii.njaesencryption.PEJniLib.aesNativeDecrypt(java.lang.String)'
so_base 0x79598c1000
fnPtr='0x79599076f8 libcsii_AESTelecomModule_v1_0.so!_Z9getValuesP7_JNIEnvP8_jobjectP8_jstringS4_' method='private native java.lang.String com.csii.njaesencryption.PEJniLib.getNativeValue(java.lang.String,java.lang.String)'
so_base 0x79598c1000
fnPtr='0x79599077b4 libcsii_AESTelecomModule_v1_0.so!_Z23setAesRanKeyWithPaddingP7_JNIEnvP7_jclassihh' method='private static native void com.csii.njaesencryption.PEJniLib.setNativeAesKeyMode(int,boolean,boolean)'
so_base 0x79598c1000
```

- 可以看到有对应的结果，但是没有打印函数的偏移，所以添加了打印so基址的代码，我们手动减一下就好了；

```
aesNativeDecrypt  0x46770

getNativeValue  0x466f8

setNativeAesKeyMode  0x467B4
```

- 去ida查看一下目标函数；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251104164756621.png)

- 应该没错，对应的逻辑就是 `getvalue` 方法，后续原本的计划是用unidbg模拟执行后辅助算法还原，但是这个so跑不起来；所以还是使用frida来hook；
- 进入之后发现反编译是不太正确的，我们传入的明文和参数2都没怎么被用到；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251105112326864.png)

- 读者可以使用其他版本的ida查看，我只有9.0和9.2，这两个没差别；那主要就需要看汇编了，首先我们看这个 `f5` 后的函数能得到这样一些信息：
    - `RSA` public key；
    - `AES_cbc_encrypt`、`AES_set_encrypt_key`；
- 再看汇编能得到些什么：
    - `AES_cbc_encrypt`、`AES_set_encrypt_key`；
    - `get_MD5_Hash`；
    - `"%s|%s|%s"`；
    - `"10001"`、`RSA_public_encrypt`；
- 应该说几乎告诉了我们答案，虽然反编译并不全，但我们能汇总出这样的信息：首先，结果的确是我们前期猜测的那样，结果就是三段，而且理应包含 `AES`、`RSA`、`MD5`、`BASE64` 这几种算法或编码；
- 基础信息了解了开始分段还原算法，那么这三段分别是什么算法？找到 `"%s|%s|%s"` 的位置反编译看看；

![](https://www.xiayutian.xyz/usr/assets/android/%E5%8D%97%E9%93%B6%E6%B3%95%E5%B7%B4%E6%B6%88%E9%87%91%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90_img/image-20251105113746768.png)

- 好家伙，一派胡言乱语，到这里我本身想去trace看汇编了，突然想起来前段时间同样遇到过一个ida反编译不理想但是binary ninja却非常出色的样本，于是我使用bn来反编译，结果果然非常理想；
