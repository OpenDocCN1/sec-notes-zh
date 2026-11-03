
# [Android CTF] 【2023】春节解题领红包之四

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1742785>
- 抓取时间：2026-11-01T21:49:49

## 前言

在开始阅读本解析前，强烈建议去看看正己大佬的[这篇帖子](https://www.52pojie.cn/thread-1695141-1-1.html)，写的肥肠滴好，B站也有对应教程，相信认真看完之后一定会很快解出 Android 的初级题。

如果依旧没有思路，再来看看这篇解析也不迟。看完嘎嘎乱杀好吧。

## 工具

JEB（本次以 JEB 为主，使用 NP/MT 也可以，不过需要些许的 Android 和 Java 基础）

## 静态分析

首先安装我们的 App，看看能否收集到一些有用的信息。可以看到输入测试用例，会有 `flag` 错误的提示。由于只有这一个页面，所以可以肯定我们要找的相关内容都在 `MainActivity` 中。可以看到整个页面我们需要关注的就是两个文本框，和一个按钮，大体逻辑应该是输入 `uid` 和 `flag`，点击验证会将输入的数据读取，所以我们重点要关注按钮的点击事件。

![image-20230206232313590.png](https://attach.52pojie.cn/forum/202302/07/020339n2kh8088e08t5jq4.png)

拖入 JEB 中查看，等待分析完成，在左下角窗口中选择 `MainActivity` 双击，然后右键选择 `Decomplie`，弹窗点击 ok 即可，可以看到我们的源码在右侧直接就会显示出来。在上一篇文章中已经比较详细的介绍了使用 NP 管理器（MT 管理器同理）如何反编译源码，静态分析以源码为主，使用 JEB 和 NP/MT 并无大差异。

![image-20230206234740949.png](https://attach.52pojie.cn/forum/202302/07/020344wtmikwai9vmjbvbw.png)

在右侧窗口直接查看源码，或者选择底部窗口选项中的 `Quick Search`，输入错误提示信息查看。由于 JEB 已经将源码分析很彻底，因此直接从源码入手也可以，可以看到右侧窗口中有一个 `Button` 对象和两个 `EditText` 对象，重点对 `Button` 对象分析。还有此处的 `b()` 方法，如果进行查看可以发现其读取了包名等信息，其实这是签名校验，也就是说，如果我们使用上一篇文章中的方法直接对其打包签名必然会出现白屏崩溃等现象，因此我们避其锋芒，对源码分析即可。

![image-20230206234247696.png](https://attach.52pojie.cn/forum/202302/07/020342vjorjjsotjaf2htf.png)

不难发现错误信息的提示在此处的 `button0` 的监听事件中，而这个 `button0` 其实就是我们上图中的 `Button` 类实例化的对象，当点击验证按钮时，此处的方法将被调用。要使得返回正确信息，就要保证 `a0.B()` 函数返回的结果为 true，此函数传入了两个参数，`uid` 和 `flag`，使用了 `trim()` 将其多余的空格去掉，并都转换为 `String` 类型。直接对 `a0.B()` 方法进行分析即可。

![image-20230207002811885.png](https://attach.52pojie.cn/forum/202302/07/020347tqq3ci83fv8qzq38.png)

在这里的 `B()` 方法上双击，出现弹窗点击 ok 即可。进入 `A` 类中，可以看到函数的执行流程如下：对 `uid` 和 `flag` 判空，校验 `flag` 是否以 `flag{` 开头、以 `}` 结尾，对 `flag` 内容进行分割，也就是取出花括号的内容，再将 `uid` 和 `"Wuaipojie2023"` 拼接并使用 `B` 类的 `encode()` 方法编码后（`encode()` 方法中将传入的字符串转为 `char` 类型数组，从后向前逆向遵循交错异或的方式重新赋值，如数组末端元素先和 53 异或，则它的前一个元素和 50 异或，再往前又是 53，以此类推。因为编写此文时 JEB 频繁闪退，故此处没有对 `encode()` 方法的截图，见谅），将其字符格式设置为 UTF-8，并转为 `byte` 数组，再将其转为 `Base64` 编码后，再使用 `MD5` 加密，然后将得到的字符串传入 `C` 类的 `cipher()` 中，也就是说 `B()` 方法获取我们输入的 `flag`，再将我们输入的 `uid` 通过一系列的加密操作后，比较两者是否相等，相等则提示正确。

![image-20230207004347261.png](https://attach.52pojie.cn/forum/202302/07/020349u5h6xxjx9rl69959.png)

在 `cipher()` 方法上双击，根据我们上一步传入的参数可知，这里调用了返回值为 `String` 的 `cipher()` 方法，这里的 `cipher()` 方法主要是区分数字和字母，如果是字母会调用返回值为 `char` 的 `cipher()` 方法，此方法中对字母进行了偏移量为 5 的凯撒密码加密，若为数字则直接使用可变字符类 `StringBuilder` 拼接后转为 `String` 对象成为最终的 `flag`，至此代码逻辑分析完成。

![image-20230207005719722.png](https://attach.52pojie.cn/forum/202302/07/020351rked85le26jreoww.png)

## 代码

根据上文的分析，可以写出由 Java 编写的 `flag` 计算方法供大家参考。

```java
package puzzle3;

import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.Base64;
import java.util.Scanner;
import java.security.NoSuchAlgorithmException;

class Solution {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入您的UID");
        String uid = sc.nextLine();
        String str = encode(uid + "Wuaipojie2023");
        byte[] byteArr = str.getBytes(StandardCharsets.UTF_8);
        String base64Str = Base64.getEncoder().encodeToString(byteArr);
        String md5Str = MD5(base64Str);
        System.out.println("您的flag为");
        System.out.println(cipher(md5Str, 5));
        sc.close();
    }

    /**
     * 编码
     */
    public static String encode(String s) {
        char [] arr = s.toCharArray();
        boolean singal = false;
        for(int i = arr.length - 1; i >= 0 ; i--){
            if(!singal){
                arr[i] ^= 53;
            }else{
                arr[i] ^= 50;
            }
            singal = !singal;
        }

        return new String(arr);
    }

    /**
     * MD5加密函数
     * @Param str
     * @return
     */
    public static String MD5(String str){
            try {
                MessageDigest md = MessageDigest.getInstance("MD5");
                md.update(str.getBytes());
                byte[]byteDigest = md.digest();
                int i;
                StringBuffer buf = new StringBuffer("");
                for (int offset = 0; offset 
```

由于包含了 `Scanner` 函数，请使用如下网站运行（将上边的代码复制到此网站，根据图片操作即可），或者使用 Java 的 IDE 运行代码，根据提示输入 `uid` 即可。

[java.jsrun.net](http://java.jsrun.net/)

![image-20230207012410538.png](https://attach.52pojie.cn/forum/202302/07/020353tjg1ynzggju8ireg.png)

## 验证

输入 `uid` 和计算后的 `flag` 显示正确。

![image-20230207015924205.png](https://attach.52pojie.cn/forum/202302/07/020354n36ycrzyg33qey62.png)

## 动态调试

如果对静态分析的过程不是很清楚的话，可以选择动态调试的方式，不过正己大佬已经将方法讲的非常清楚了，且有对应的学习视频，本题目对应的[课程](https://www.52pojie.cn/thread-1714727-1-1.html)，大家可以去学习尝试一下。强烈建议从第一课开始看，一定要动手！！！对于这道题目由于有签名校验，可以直接使用 NP/MT 管理器的去除签名校验功能，也可以自己手动去除，为什么要去签名校验呢？因为我们要使用动态调试的话，第一种方式就是修改 `AndroidManifest.xml` 文件中的 `debuggable` 权限，改动文件必然需要重新签名，因此必须去除签名校验才能正常运行，当然不修改文件也可以，建议大家可以把几种方法都尝试一下，想必一定会收获颇深的，如有相关问题可在本贴留言讨论。

## 其它题解

- 【2023】春节解题领红包之二：<https://www.52pojie.cn/thread-1742329-1-1.html>
- 【2023】春节解题领红包之三：<https://www.52pojie.cn/thread-1742449-1-1.html>
- 【2023】春节解题领红包之 web 篇——不完美题解：<https://www.52pojie.cn/thread-1743080-1-1.html>
