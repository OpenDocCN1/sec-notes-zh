
# [CTF] [De1CTF2019]REVERSE的Re_Sign

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1603671
- 抓取时间: 2026-11-01T23:33:42

## [De1CTF2019]REVERSE的Re_Sign

![](https://img-blog.csdnimg.cn/b19decf079564fcda937f05063e39e55.png?)

照例下载附件，扔到 `exeinfope` 中查看信息（PS：附件在最后）：

![](https://img-blog.csdnimg.cn/a56a0491c9f3477e90052648424853aa.png?)

提示 `UPX` 壳，但是好像无法使用 `UPX -D` 来脱壳：

![](https://img-blog.csdnimg.cn/120605699d124567b86ed2d979b13d1c.png?)

那么先尝试一下手动脱壳，扔入 `OD` 中，利用 `ESP` 脱壳定律：

![](https://img-blog.csdnimg.cn/83e9112af71c4177bcb0f82e79a5c5e7.png?)

![](https://img-blog.csdnimg.cn/0c797b00d9714bc382d5af71e8769b5f.png?)

![](https://img-blog.csdnimg.cn/90e59d0b47a14827a0bf741b3cc850c7.png?)

然后嘛~脱壳后导入表乱了，用 `ImportREC` 修复也修复不了，(我曾想着自己修复，但还是太年轻了 ~ ^ ~)：

然后经友人推荐使用了 `UpxUnpacker.exe`，直接脱壳且能运行！不得不说这个几百KB的工具真的专业，希望以后能有机会了解其什么逻辑！

![](https://img-blog.csdnimg.cn/78bd4d035f0d4c7a96fe23d93a5f86dc.png?)

照例先运行下程序，查看主要回显信息：

![](https://img-blog.csdnimg.cn/3feeb02511f04fce8754031de96aeaf2.png?)

![](https://img-blog.csdnimg.cn/38f32d71bc004cafaf23716a1821368e.png?)

然后后面就开始动调了，那什么时候要动调呢？

我觉得是你看不懂小段代码，或者没法看函数内嵌套多层函数的大量代码，这时候只能用动调的结果和不断变换输入来猜想逻辑了。

![](https://img-blog.csdnimg.cn/01c8edb6ac7247e6aa48309dc2b29674.png?)

![](https://img-blog.csdnimg.cn/593a1ac3e116438a9139f6910a9ed8d7.png?)

后面就是一种逆向思想的直觉了，你得知道他大致考什么，以及你想要什么~

我输入的是 `flag{1111}`，看到可疑的 `HefBHzofEP4fCU==` 密文，但是传统 base64 加密应该是 `ZmxhZ3sxMTExfQ==`，所以如果有这种逆向思想，应该要想到是变表加密，那么进一步的，我们要找到变表。

有目的性得动调真的很重要，这题代码成百上千行，一个个看会看傻的。

然后就是这次动调感触骤深，总结了以下几点：

1. 一般的 `IDA` 动调中要反汇编和 `F5` 伪编译一起看，因为很多值 `F5` 伪编译中是不会显示或者显示不准的，要从汇编代码中才能看到精确值：

   ![](https://img-blog.csdnimg.cn/76a54a7a29d0452dac280c6d2517ee66.png?)

   ![](https://img-blog.csdnimg.cn/4f4596dc0157413fa620c04ee59ea800.png?)

2. 既然 `IDA` `F5` 伪编译不太靠得住，反正都要看汇编，`OD` 的汇编动态真的能显示很多关键字符！更直观，更适合捕抓数据！

   ![](https://img-blog.csdnimg.cn/7cc17b88df094411ae14ca8c20c042d2.png?)

   ![](https://img-blog.csdnimg.cn/2f15aba8d7ab4c5891743da88230a640.png?)

现在需要的变表捕抓到了，现在我们看一下师傅们有逆向思维的做法：

![](https://img-blog.csdnimg.cn/c640ee22e028419aa6b6febfba20f7fe.png?)

![](https://img-blog.csdnimg.cn/dc187d967a224746bb52fdf781166cad.png?)

![](https://img-blog.csdnimg.cn/f9fc5dfcb2804e8cb7407e6f4931d165.png?)

![](https://img-blog.csdnimg.cn/f5016bf5826f400e8e400750114b4de2.png?)

![](https://img-blog.csdnimg.cn/92368c91a45d4e56af60954218864e70.png?)

前面变表找到后先告一段落，后面来看看要比较的数据：

![](https://img-blog.csdnimg.cn/00fde958170e4156a2b65dd5c08c7320.png?)

![](https://img-blog.csdnimg.cn/dd011bcf74f64e6c8aa523abfffe11e5.png?)

![](https://img-blog.csdnimg.cn/cba205a0b7ee45a9a712f4e1e4dbc343.png?)

![](https://img-blog.csdnimg.cn/557702e5d7ab497a94fa22486aab4063.png?)

![](https://img-blog.csdnimg.cn/8f3e3afd776f433a99e09b003a375f06.png?)

那么我们梳理一下，`用户输入----->变表加密----->变表密文在传统表单的下标`：

一开始我想了好一会，如果变表加密的 `E`，在传统表单中下标是 4，那不还是 `E` 嘛？那直接把要比较的数据用变表解密不就行了？一开始我还想着映射中转一下再解密，果然还是对映射密码理解得不透彻。

![](https://img-blog.csdnimg.cn/57fec7ece54c4cda995af7d59b742873.png?)

![](https://img-blog.csdnimg.cn/b61a42c9435f47999e338f45fb248e0e.png?)

```
address=0x6c1848
list=[]
for i in range(48):
    list.append(Dword(address+i*4))
print(list)
```

常规解密即可：

![](https://img-blog.csdnimg.cn/96d374e42aef4d689dbe4d4579d3a006.png?)

解毕！

敬礼！
