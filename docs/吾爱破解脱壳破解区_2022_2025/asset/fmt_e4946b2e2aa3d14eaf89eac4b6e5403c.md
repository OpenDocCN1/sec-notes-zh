
# 分析加密逻辑

```
encodeURI(Object(l["c"])(a.value, Object(l["f"])(Object(l["g"])(s.value))))
l["c"](a.value, l["f"](l["g"](s.value)))
```

![](https://attach.52pojie.cn/forum/202203/30/120007xwreet68twaorz8x.png)

`202203300922977.png` (12.15 KB, 下载次数: 267) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMwOHxiM2FkZTk3ZXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:00 上传

先找 `l["c"]` 方法，可以看到 `H` 方法，追进去看一下。

![](https://attach.52pojie.cn/forum/202203/30/120038dv9vhopugtehf651.png)

`202203300922977.png` (18.8 KB, 下载次数: 271) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMwOXwwMjBiZDcyMXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:00 上传

![](https://attach.52pojie.cn/forum/202203/30/122916zl2qfndy2jzskjy2.png)

`202203300922977.png` (32.43 KB, 下载次数: 274) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyMnw1OTdmZDBkOXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:29 上传

可以看到最后返回 `s.toString()`，`s = p.a.TripleDES.encrypt(e, d, l)`。

![](https://attach.52pojie.cn/forum/202203/30/122942frdozgl5r5ohifd0.png)

`202203300922977.png` (32.35 KB, 下载次数: 277) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyM3xjYzc5MDIyOXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:29 上传

这里参数 `e` 是传参过来的，`d` 也是 `p.a` 的方法生成的，`l` 也是 `p.a` 的方法生成的。找一下 `p.a` 在哪儿构建的，`ctrl+f` 搜索后发现找不到，那就找 `p` 的构建。

![](https://attach.52pojie.cn/forum/202203/30/123004gh04nyatar4yx74a.png)

`202203300922977.png` (31.67 KB, 下载次数: 263) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyNHw2NzY3ZGY0NXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:30 上传

看到这里就可以确定是 webpack 了。

# webpack 处理

## 第一种姿势：使用 AST 自动扣 webpack

这里用漁佬（@漁滒）的 AST 自动扣 webpack。

这里附上神器地址：

- 使用教程：[https://www.52pojie.cn/thread-1572529-1-1.html](https://www.52pojie.cn/thread-1572529-1-1.html)
- 源码：[https://gitcode.net/zjq592767809/webpack_ast](https://gitcode.net/zjq592767809/webpack_ast)

寻找加载器的 js 文件保存到本地。

在 `password` 断点文件中看一下有没有加载器加载函数。

![](https://attach.52pojie.cn/forum/202203/30/123033n99whgtpgzzhvzxg.png)

`202203300922977.png` (24.89 KB, 下载次数: 272) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyNXw4Njg1NGJjMnwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:30 上传

在文件开头可以看到 `r("854c")` 这个 `r` 肯定就是加载器了，在这行打一个断点，重新刷新页面。

![](https://attach.52pojie.cn/forum/202203/30/123056dz19ydylk55iwkyr.png)

`202203300922977.png` (30.3 KB, 下载次数: 282) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyNnw5MGNmNDRiZHwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:30 上传

这里可以看到 `r` 的函数体 `u()`，追进去。

![](https://attach.52pojie.cn/forum/202203/30/123121b4hkfl2fej1fiffc.png)

`202203300922977.png` (25.86 KB, 下载次数: 276) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyN3xhODNjZTZmYnwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:31 上传

这个 `u()` 就是加载器了，直接将这个 js 文件保存到本地。

![](https://attach.52pojie.cn/forum/202203/30/123140w3gmehbmgnema18e.png)

`202203300922977.png` (17.4 KB, 下载次数: 280) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyOHw1OGZiNTAzYnwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:31 上传

寻找函数模块的 js 文件保存到本地。

刚刚发现 `p` 是通过 `t.n(t('3452'))` 加载出来的。在 `s = t("3452")` 打个断点重新刷新页面，发现这个的 `t` 就是我们的 `u` 构造器，那就好办了。

![](https://attach.52pojie.cn/forum/202203/30/123201sgb8x9kdxjgumkkx.png)

`202203300922977.png` (20.63 KB, 下载次数: 275) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMyOXxjMDA3NjAxOXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:32 上传

![](https://attach.52pojie.cn/forum/202203/30/123237kv42hqhjdj8n6n8f.png)

`202203300922977.png` (22.64 KB, 下载次数: 279) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzMXwwYmEzMTdhZXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:32 上传

`F11` 单步追 `t("3452")` 就可以找到函数模块，把这个文件保存到本地。

![](https://attach.52pojie.cn/forum/202203/30/123311uson07sdnaney7yy.png)

`202203300922977.png` (79.79 KB, 下载次数: 276) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzMnwyNWEwODFhZXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:33 上传

使用大佬写好的 AST 代码。现在目录下有 3 个文件。

![](https://attach.52pojie.cn/forum/202203/30/123335vgratjoj7bmrhh8g.png)

`202203300922977.png` (23.88 KB, 下载次数: 271) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzM3w4MWFkYmEyY3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:33 上传

执行：

```
node .\webpack_mixer.js -l .\main.03a4fbe9.js -m .\chunk-vendors.f4efd280.js -o webpack_sdk.js
```

生成了扣好的 webpack 代码。

![](https://attach.52pojie.cn/forum/202203/30/123356mwfcpcpd0alffzdv.png)

`202203300922977.png` (15.3 KB, 下载次数: 280) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzNHwzYjNhYTk2NnwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:33 上传

## 第二种姿势：自己扣 webpack

往上看找加载器，可以看到开头 `function(e)` 先扣下来。将加载器 `(function(e) {` 里面的全部内容 `}` 扣下来，封装为匿名函数。

![](https://attach.52pojie.cn/forum/202203/30/123418z9bz2f9jujz99jvj.png)

`202203300922977.png` (48.45 KB, 下载次数: 281) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzNXw1YjNlN2MyN3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:34 上传

声明一个全部变量，再将加载器 `u` 赋值给 `use`。

![](https://attach.52pojie.cn/forum/202203/30/123435re40nho75o6746nr.png)

`202203300922977.png` (32.78 KB, 下载次数: 276) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzNnw3ZGIxZDBkYXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:34 上传

![](https://attach.52pojie.cn/forum/202203/30/123508ae05k51znh13tx0p.png)

`202203300922977.png` (21.44 KB, 下载次数: 278) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzN3wwNzg1NjE5NnwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:35 上传

先把函数模块 js 文件保存下来。

![](https://attach.52pojie.cn/forum/202203/30/123528nhdqt8w5rdtzrapw.png)

`202203300922977.png` (100.48 KB, 下载次数: 272) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzOHw0ZjllYTgzOXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:35 上传

搜索 `3452`，将 `3452` 传给匿名函数。

![](https://attach.52pojie.cn/forum/202203/30/123546hw1gtztwjzwdq6y2.png)

`202203300922977.png` (16.91 KB, 下载次数: 270) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTMzOXw3YWQwNjQ2Y3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:35 上传

加载一下 `3452` 函数，`window` 未定义，补上 `window`。

![](https://attach.52pojie.cn/forum/202203/30/123627koaaxtxg9ah2lgx8.png)

`202203300922977.png` (16.91 KB, 下载次数: 272) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0MHwwZDQ0NTIzZHwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:36 上传

![](https://attach.52pojie.cn/forum/202203/30/123654dwc2uxqzum7u97qd.png)

`202203300922977.png` (13.36 KB, 下载次数: 274) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0MXxkYWEzMWYyNXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:36 上传

![](https://attach.52pojie.cn/forum/202203/30/123714bpec30x8zx95c89c.png)

`202203300922977.png` (20.16 KB, 下载次数: 283) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0MnwxMTkzZGQ1N3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:37 上传

再次运行发现 `TypeError: Cannot read property 'call' of undefined` 这个错误，这个错误就是少函数模块，然后发现 `3452` 里面加载了其他函数模块。

![](https://attach.52pojie.cn/forum/202203/30/123737zv1e5ygyyr55j1re.png)

`202203300922977.png` (13.02 KB, 下载次数: 295) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0M3w1YTdkMGYyM3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:37 上传

![](https://attach.52pojie.cn/forum/202203/30/123757inb5bpo85114npbk.png)

`202203300922977.png` (25.02 KB, 下载次数: 269) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0NHwyMDAxNjA1ZHwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:37 上传

将其他函数模块粘过来，粘完之后运行就不报错了。

![](https://attach.52pojie.cn/forum/202203/30/123816relc9zvcmvr63jjj.png)

`202203300922977.png` (80.53 KB, 下载次数: 280) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0NXw4ODM4YzBlMXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:38 上传

# 构造 password 生成代码

将 `H()` 函数扣下来，生成 `p`。

![](https://attach.52pojie.cn/forum/202203/30/123835rkvwdei7p6i2c676.png)

`202203300922977.png` (43.42 KB, 下载次数: 270) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0Nnw5ZmFmNjc2N3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:38 上传

这里用漁佬的生成加载器赋给了 `module.exports`，所以用 `module.exports` 加载 `3452`。

![](https://attach.52pojie.cn/forum/202203/30/123852qir8qp82r7vdg32q.png)

`202203300922977.png` (33.27 KB, 下载次数: 274) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM0OXwxMTJhNjAxNHwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:38 上传

![](https://attach.52pojie.cn/forum/202203/30/123911jmm4o9p6c6977h7p.png)

`202203300922977.png` (2.82 KB, 下载次数: 278) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM1MHw5NDlkOTQ0Y3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:39 上传

![](https://attach.52pojie.cn/forum/202203/30/123931hikgjgvk5tcpir8p.png)

`202203300922977.png` (46.8 KB, 下载次数: 268) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM1MnxmNTgyMTAxY3wxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:39 上传

调用 `H` 函数（这里我将 `u` 赋值给了 `use`，所以用 `use` 加载 `3452` 函数）发现不报错了。

![](https://attach.52pojie.cn/forum/202203/30/123953i2pnv4s4uprpwvuu.png)

`202203300922977.png` (48.3 KB, 下载次数: 273) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwOTM1M3wyNTY1ZWE0ZXwxNzkzNTc1ODY3fDE2ODA4NnwxNjEzNDY2&nothumb=yes) 2022-3-30 12:39 上传

之前美化的调用 `h` 方法的代码就变成了
