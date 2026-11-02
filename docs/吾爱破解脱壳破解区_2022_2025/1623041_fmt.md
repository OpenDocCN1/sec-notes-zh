
# [Web逆向] ins 登录enc_password获取

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1623041
- 抓取时间: 2026-11-01T23:28:52

## ins 登录enc_password获取

目标网址 `aHR0cHM6Ly93d3cuaW5zdGFncmFtLmNvbS8=`

### 1. 抓包

```
POST https://www.instagram.com/accounts/login/ajax/
payload = {
    enc_password: #PWD_INSTAGRAM_BROWSER:10:1650113230:AbFQAHOTPiX8WpL1YUGC0BzN/OPlUZ1+dKwhqoZxq5HMnEUFvVBDbStPcRuatPOPWm9EuIbm9JVqKJM0X6jLK0NEm+a5rRqgVA+tlJiIItdWknJeifUCzsY9ik/MaRNRQapCIFySa6SoSsqg
    username: xxx@163.com
    queryParams: {}
    optIntoOneTap: false
    stopDeletionNonce: 
    trustedDeviceRecords: {}
}
```

很容易看出来`enc_password`是一个加密后拼接起来的参数,他就是我们今天要找的加密参数。

### 2. 调试分析

- 全局搜索 `enc_password`

![](https://img-blog.csdnimg.cn/39400168cb2f4dac80dda99ac299d32c.png?)

什么都没有，nothing，那换一个关键词试一下，`PWD_INSTAGRAM_BROWSER`是用来拼接字符串的，很可能会搜到，试一下

![](https://img-blog.csdnimg.cn/74088b8b22a84cbd8851597dd3c33c9a.png?)

有了，点进去，格式化看一下，

![](https://img-blog.csdnimg.cn/bf73f6a3ae9f4b7c8a194a1a1affb143.png?)

这个地方好像是我们要找的地方，仔细看一下，下面还有用冒号拼接的字符串，和我们最终的结果很像。那就打个断点，调试一下。

![](https://img-blog.csdnimg.cn/f975ca22892f4429818b51eae674ee2d.png?)

这个就是最终的结果，开始搞他。进入到函数里面，继续打断点，调试

![](https://img-blog.csdnimg.cn/84f5e9c5a1ef4fe0a72fe8fad3aced4d.png?)

会找到这样一个函数，`t`是你的密码，`c = n()`; 看一下`n`函数，发现是一个10位数的时间戳，这里就可以想到最终的结果里面也有一个时间戳，应该就是这个了。进入到函数中打断点，调试。

![](https://img-blog.csdnimg.cn/c90812f1a86a4a54927634d5b3fdb987.png?)

到这里就能清晰地看到，获取了`KeyId`, `PublicKey`, `Version`,三个参数加上咱们传过来的 `t`(密码), `n`(时间戳)，调用了`encryptPassword()`,继续跟进去查看。

![](https://img-blog.csdnimg.cn/3138ea34e807431790b70dac6aa6fa06.png?)

在这里 `t`,`c`,`n`,`o`,`s`是我们上面传来的参数，`u`是`o`转成的byte数组，我这里有一个自用的方法`str_to_utf8()`

```
function strToUtf8(n) {

    if ('string' != typeof n)
        throw new TypeError('expected string');

    var t, o = unescape(encodeURIComponent(n)), c = new Uint8Array(o.length);

    for (t = 0; t 
```

然后 `y`是经历一个加密函数得到的结果，最终`return`的结果应该是`base64`后的`y` 和 `s`, `n`进行一定格式的拼接。继续看`y`是怎么来的

`y = await i(d[1]).encrypt(t, c, u, f)`;

![](https://img-blog.csdnimg.cn/61d152103758427bbf7254bb1ec2ce4b.png?)

这个`y`经历了这么多，一步一步的看吧，先是 `u = o + h.length`; 然后 `const w = n(c)`; 接下来就到了 `const y = new Uint8Array(u)`;

我们需要的`y`出现了，初始化一个`u`长度的无符号整型数组,然后第0位填1，第1位填`s`,然后索引`f += 1`,下面很明显的写着`AES-GCM`,有同学可能就觉得这只是一个通用的加密算法，我们找到`key`,`iv`不就可以了吗，但是这里不止这一个算法。

继续看，`AES-GCM` `iv`是空的12位无符号整形数组，还有一个`additionalData`(添加的数据) `length:256` 所以是`aes-256-gcm`算法，

这里有关于这个算法的详细信息  [//developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/generateKey" target="_blank" rel="noopener noreferrer nofollow"](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/generateKey:rel="nofollow noopener" href="https)

使用的也是`window.crypto`, `node`里面正好有相应可以使用的算法。继续往下看，经过`aes`加密的结果为`o`,然后 `return Promise.all([n, o])` 继续下一步。

![](https://img-blog.csdnimg.cn/22c03bf7e9a94721ba49fc61129399bf.png?)

上一步的结果传到了`n`参数上面，通过`t`函数就获得了`o`的值，然后`y`的索引继续增加，`y`不断被更新，返回了最后生成的`y`，

现在的问题就是找`t`函数的逻辑,继续调试。

![](https://img-blog.csdnimg.cn/71437a9c8ea747239e75f1755dc6d93b.png?)

这里面又有三个新的函数 `c = i(d[1]).box.keyPair()`; `var l = i(d[2])(c.publicKey, n)` , `o = i(d[1]).box(t, l, n, c.secretKey)`;

第一个`c`是生成`publicKey`和`secretKey`, 然后通过这两个`key`去生成下面两个参数。这个地方就跳过了，扣出相应的函数，就能跑了

看 `l` 的生成

![](https://img-blog.csdnimg.cn/59de25c2fee54ecca84ada586d0ebaac.png?)

这里面又看到了一个算法 `blake2b`, 也就是说 `l` 是 `blake2b`生成的，复制一下需要的`js`代码,就可以生成 `l` 了，这个不是用的官方的算法库，不知道有没有魔改，我就直接抠出来代码跑的。接下来看 `o` 参数的生成。

![](https://img-blog.csdnimg.cn/b1d366ebae724c2ba4a7b2533ee4bedc.png?)

又是一个很规矩的函数，没有混淆，直接复制，`o`参数到这里就结束了。

最后把生成的 `y`数组 `base64`编码一下就获得了最后的结果，如下图，我用`node`改写的算法，获取的结果

![](https://img-blog.csdnimg.cn/b327ad6e0cb944858c081a84a967c3f0.png?)

和请求中的参数很像，放到登录程序中测试一下，

![](https://img-blog.csdnimg.cn/a1a75cabe3d946daa1556d2e32a83d2f.png?)

登陆成功，在测试一下，这个`cookie`好不好使

![](https://img-blog.csdnimg.cn/ff4e71a5de514330a54b3eeb9a294eef.png?)

完美的获取到了数据。还要注意的地方就是`ins`后端是用`django`写的，所以`post`请求需要加上`csrf_token`,还有就是`publickey`,`KeyId`,`Version` 这几个参数的获取，因为他们会变的。

今天的分享就到这里了，这是我学习中的一次尝试，记录一下，有不对的地方希望大家可以指正。谢谢大家的浏览。
