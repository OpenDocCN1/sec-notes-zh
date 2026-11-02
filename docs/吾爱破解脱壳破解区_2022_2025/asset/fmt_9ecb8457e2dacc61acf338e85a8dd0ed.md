
# [原创] DBeaver Ultimate edition 算法分析 本地服务器搭建

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1668139
- 抓取时间: 2026-11-01T23:12:14

DBeaver Ultimate edition 算法分析/本地服务器搭建

## DBeaver Ultimate edition 算法分析

官方下载地址: [//dbeaver.com/download/ultimate/" target="_blank" rel="noopener noreferrer nofollow"](https://dbeaver.com/download/ultimate/:rel="nofollow noopener" href="https)
当前分析版本号: `dbeaver-ue-22.1.0`

## 前记

> 
前几天出了 该软件的 爆破分析; 机票 :
[//www.52pojie.cn/thread-1666440-1-1.html" target="_blank" rel="noopener noreferrer nofollow"](https://www.52pojie.cn/thread-1666440-1-1.html:href="https)
这次来尝试下不一样的...(`尽量不硬编码源程序`) ;

## 环境

> 
`JAVA`/`javassist`/`Python`

流程:
1. 解析 `license` 算法, 生成注册器, 生成 `publicKey`/`privateKey`/`license`/`subscription`
2. 替换程序自带的 `publicKey`
3. 配置 `host` 映射, 搭建本地验证服务器
4. 利用第一步生成的 `license` 注册

`WIN10`/`Mac m1` 测试通过

## 分析

### license

![](https://attach.52pojie.cn/forum/202207/28/172306jice7ic3c774vusl.jpg)
1_import_error.jpg (250.44 KB, 下载次数: 263)
[href="forum.php?mod=attachment&aid=MjU0MTMzNnxlZTY2ZTg2Y3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTMzNnxlZTY2ZTg2Y3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
首先通过在程序界面 `import` 一个 `license`,
查看关键字, 定位到目标 `jar` 包 (也可以查看程序 `log` 日志, 位置: `/Users/artemis/Library/DBeaverData/workspace6/.metadata`)
可以看到, 程序先将我们输入到 `icense` 去除空格以及删除‘-’,‘#’这些特殊标记符号之后 `base64` 解密成字节数组
这里格式化字符串写的挺好,
就是你的 `license` 可以有换行, 可以有 ----PUBLIC-KEY--- 这些文件头, 也可以有 # this is comment 这些注释, 格式化之后会只留下关键的 `license`
程序操作 `publicKey`/`license`/`subscription` 都有这个逻辑; 当然; 这个并不重要..

![](https://attach.52pojie.cn/forum/202207/28/172316sooc4o4xscd4otdv.jpg)
2_import_license.jpg (95.88 KB, 下载次数: 264)
[href="forum.php?mod=attachment&aid=MjU0MTMzN3w0ZDM1NWRiMXwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTMzN3w0ZDM1NWRiMXwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

![](https://attach.52pojie.cn/forum/202207/28/172318rux2t26lwstay2t6.jpg)
3_base64_decode_license.jpg (160.49 KB, 下载次数: 258)
[href="forum.php?mod=attachment&aid=MjU0MTMzOHxkZjFmZmRhN3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTMzOHxkZjFmZmRhN3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
然后在 `licenseManager.importLicense` 将 字节数组 解析成 注册凭证,
核心的算法都在 `importLicense` 里面,我们继续看

![](https://attach.52pojie.cn/forum/202207/28/172321o4so50jkay4vg4sa.jpg)
4_getDecryptionKey.jpg (102.07 KB, 下载次数: 263)
[href="forum.php?mod=attachment&aid=MjU0MTMzOXxjZWI3ZDU5ZXwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTMzOXxjZWI3ZDU5ZXwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
可以看到, 获取了一个 `publicKey` 用来解密, 既然公钥解密, 那么加密就是私钥了 (这里有用,我们后面会生成密钥,用来注册 `license`)
刚开始以为 `publicKey` 是走接口获取到的, 毕竟里面有个 `URL` 什么的;
> 后来发现其实是 `jar` 包里的资源文件;

![](https://attach.52pojie.cn/forum/202207/28/172323hbxj434dtz5tz1ty.jpg)
5_get_resource_publickey.jpg (148.97 KB, 下载次数: 262)
[href="forum.php?mod=attachment&aid=MjU0MTM0MHw4Y2FkMGNmN3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0MHw4Y2FkMGNmN3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
我们模糊搜索下‘-public.key’, 定位到了这个 `jar` 包: `com.dbeaver.app.ultimate_22.1.0.202206121904.jar` (这里有用, 后面我们会替换该 `publicKey` )
推算出 `productId` = ‘`dbeaver-ue`’ , 这个应该就是跟着发版来走的; 而且是固定值
众所周知,`publicKey` 与 `privateKey` 是 成双成对的, 而且 我们拿不到私钥, 后面只能想办法用我们自己生成的 公钥与私钥了

![](https://attach.52pojie.cn/forum/202207/28/172326yee8peymdjph0e4e.jpg)
6_jar_with_publickey.jpg (235.43 KB, 下载次数: 259)
[href="forum.php?mod=attachment&aid=MjU0MTM0MXxkMzk1Y2U1NXwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0MXxkMzk1Y2U1NXwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
接着往下看, 拿到 `publicKey` 后,开始解密, `ras` 解密, 没什么好说的, 关键是解密后的 `license` 格式:
即使解密成功, 但是 `license` 格式不对, 也是不行的 ;

![](https://attach.52pojie.cn/forum/202207/28/172328u6xo49h36423o6ch.jpg)
7_ras .jpg (82.33 KB, 下载次数: 261)
[href="forum.php?mod=attachment&aid=MjU0MTM0M3xkNDc5ZGQ0NHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0M3xkNDc5ZGQ0NHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

![](https://attach.52pojie.cn/forum/202207/28/172331s2quq6v831vq5eq7.jpg)
8_license_format.jpg (357.05 KB, 下载次数: 264)
[href="forum.php?mod=attachment&aid=MjU0MTM0NHxiZjgxYzViMHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0NHxiZjgxYzViMHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

```
STANDARD((byte)0, 218, "Initial basic license format"),
EXTENDED((byte)1, 238, "Extended format with owner email and corporate license info");

if (buffer.capacity() != this.licenseFormat.getEncryptedLength()) {
    throw new LMException("Bad " + this.licenseFormat + " license length (" + buffer.capacity() + ")");
```

> 
可以看到 `license` 第一位字节为 凭证格式,是个枚举类型, 不同的格式 `license` 长度是不同的,
后面我们选 `STANDARD` 这种 `license` 的话 , 第一位就是0,并且 `license` 总字节长度必须是 218
后 16为 是 `license_id` 字符串, 在往后是 `licenseType`,`licenseStartTime`,`licenseEndTime`..... 这里不多解析了, 生成的时候按这个规则来就行
有一点注意到事, 必须满足位数要求, 比如 `license_id` 是16位, 你可以只写10位, 剩下的6为必须用0 或者空填充
可以看到, `license` 的读取是用的 `JAVA` 的 `Buffer` 来操作的; 我们生成的话 也可以用 `Buffer` 来操作;
好; 接下来开始手写 `license` 的生成,具体代码如下;

```
// 填充字符串

for (int i = 0; i 
```

![](https://attach.52pojie.cn/forum/202207/28/172337ktn8z0g87k18q330.jpg)
9_keys_file.jpg (63.93 KB, 下载次数: 261)
[href="forum.php?mod=attachment&aid=MjU0MTM0NXwyMGNhYTRlY3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0NXwyMGNhYTRlY3wxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
生成之后,可以看到多了这三个文件
然后复制生成的 `license.key` ,重新启动程序 (替换了 `jar` 包里的 公钥资源文件,所以需要重启) ,
> 注册一下试试;

![](https://attach.52pojie.cn/forum/202207/28/172339mnx33z3xln395x99.jpg)
10_sub_error.jpg (78.65 KB, 下载次数: 269)
[href="forum.php?mod=attachment&aid=MjU0MTM0Nnw0ZWZjNzk2ZnwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0Nnw0ZWZjNzk2ZnwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
哎,居然报错了; 跟进去再看看,
发现个坏消息 , 原来除了用 `publicKey` 解密, 还有一层校验,
程序会会带着 本地解密后的 `license` 信息 请求 服务器,来获取 `subscription` 订阅信息;
服务器的 数据 我们是没法子整的 , 但是我们也可以 本地搭建一个服务器; 让程序来请求我们自己的本地服务器来验证;

### subscription

![](https://attach.52pojie.cn/forum/202207/28/172342u01s8nc4nn4pprdp.jpg)
11_validateLicense.jpg (498.96 KB, 下载次数: 261)
[href="forum.php?mod=attachment&aid=MjU0MTM0N3wwNTYzYmQ3OHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0N3wwNTYzYmQ3OHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

![](https://attach.52pojie.cn/forum/202207/28/172344zcdrfef1kn38ey1d.jpg)
12_license_check.jpg (300.61 KB, 下载次数: 267)
[href="forum.php?mod=attachment&aid=MjU0MTM0OHxkMzA0MDY4NHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0MTM0OHxkMzA0MDY4NHwxNzkzNTc0NzMyfDE2ODA4NnwxNjY4MTM5&nothumb=yes)
2022-7-28 17:23 上传

> 
仔细瞅瞅,看到 接口响应成功的 `response` 也是有些规则的,示例如下:

```
response = "VALID:SUCCESS" + "\r\n" + "-- SUBSCRIPTION" + "\r\n" + subscription
```

> 
`VALID` 与 `SUCCESS` 为状态, 并且必须用: 切割, 程序例有教研 ,':' 的 `ascii` 为 58;
如果响应文本没有包含':'的话, 会直接报错;

```
int divPos = licenseStatusString.indexOf(58);

if (divPos == -1) {

    var19 = new LicenseCheckResult(GeneralUtils.makeErrorStatus("Bad check status: " + licenseStatusString), LMLicenseStatus.INVALID);

    return var19;

}
```

> 
除了状态, `resposne` 后面的文本才是 `subscription` 主体, 其中包含了服务器返回的订阅信息; 
如果这里为空 也是会注册失败的;
