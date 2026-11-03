
![图片15.png (26.28 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/132344tgq3mxcqcusevua3.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU0NHwwOGQ1YWE4M3wxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:23 上传

![图片16.png (55.98 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/132415y6691cmj6itjttzk.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU0NXxlOGEwYjYwYXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:24 上传

## 第三步

### 3.3

打印出来之后 我们不要着急直接就去扣取 因为这里不是最终出现密文的地方 我们还需要单步调试

我们打印在控制台之后我们断点也就会失效 所以我们需要重新分页发包

我们需要按照 3.1 从新操作 然后一直单步执行下去 直到下图

![图片17.png (33.84 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/132508l6bb2xlzh549z540.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU0Nnw1Y2U1YzNjOHwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:25 上传

### 3.4

到这里 我们看到 `case4` 像是在做某些加密 但不知道是在干啥 所以但我们还需要单步执行

因为我们需要看清它的流程 如果是我们要的密文 看清楚整个流程 我们才能更好更快的扣取出

所需模块和 方法 所以我们接着单步执行

![图片18.png (73.66 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/132604arfz98r7sfr0s9rn.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU0N3w0YTk0MTZhYnwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:26 上传

### 3.5

跳过当前函数之后我们发现他直接结束了 也就是没有进入到 平摊里里面 所以我们 接着单步执行看看它是怎样运行的

我们单步运行之后 经过一些杂七杂八的函数 最后用回到了这里并且发现 `switch`条件变为的 0 也就是要执行 `case 0` 了 ，

那么接着我们就要分析一下 `case0`里面有什么 ，跟我们的加密有没有关联

![图片19.png (59.74 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/132727q41o22n5u4rt1o7f.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU0OHw0YzZjMzE1Y3wxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:27 上传

## 第四步 分析这个平摊流

### 4.1

开头`switch` 条件是0 那么他就会 往下执行 `case 0` 处 就是下图的 1，

而下图一中 返回的里面有几个关键的 代码 就是 `t = new d[“a”]` 还有 `A.sent` 然后就是

`Q.a.post("/open/homepage/public");` 他们分别对应 第一个毫无疑问就是在 创建`RSA` 实例对象

第二个是将 4 赋值给 `A.sent` 那么在开头直到 条件 是由`A.sent` 给予的 第三个就是在请求

我们开头分析的public数据包 也就是获取密钥

![图片20.png (30.62 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/132937pggt64a4qo6jtj20.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU0OXw1MTc5MjYyMXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:29 上传

### 4.2

所以我们就单步执行 看看效果 在`Q.a.post("/open/homepage/public");` 要跳过因为这是一个请求包没必要看，

下图中 1 2 3 4都是跳过当前函数 ，到第四步 就接着单步执行 ，

会经过一些杂七杂八的函数和过程 不过最终会回到这里 也就是最终结果处

![图片21.png (33.39 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133059v06mq0m0yfa06ym0.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1MHw4MDg2NDhiNXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:30 上传

### 4.3

我们让 程序 进入到 `case 4` 处 并且在最后下断点 在执行到断点处之前我们可以清晰的看到

这是一个加密，至于是不是 加密最终结果我们移动到断点处 查看结果

![图片22.png (40.46 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133148iueeewwpwe0a0m7g.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1MXxmZjFmMjQ5N3wxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:31 上传

### 4.4

我们移动到段之后可以看到 密文 出来了 至于是不是 加密结果 我们放掉断点

然后 只保留这个断点因为万一是加密处，我们就可以快速的回到这里

![图片23.png (90.35 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133230qtqqntob0l0onwqv.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1MnxhYjUwYmJiMXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:32 上传

![图片24.png (60.87 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133313rbef2ci232id2d6f.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1NHwxZjgyNWIwNnwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:33 上传

### 4.5

我们看到加密位置是正确的所以接下来我们就可以开始扣取代码了，也就是 `case4` 的那部分

---

## 代码扣取与调用

## 第五步

### 5.1

打开Code 将`case 4` 的代码全部放入我们创建好的新文件，这个代码的流程初步可以看到 `r`是密钥，

将密钥传入 `t.setpubickey(r)` 完成 `RSA`加密准备 `a` 是里面的`sign` 是`MD5` 所以 其他的就可以看报错 缺哪个补那个

![图片25.png (23.17 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133532i9umlwulyaq37ubg.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1NXw4ZThlYTQwNHwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:35 上传

### 5.2

所以第一步我们就要找 `t` ,我们回到浏览器就可以看到 这个平摊流中就有`t` 的定义处，

经过前几步的流程 得知`t`的步骤 所以我们就将他也扣出

![图片26.png (34.36 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133629lillp4bimo27l1o5.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1Nnw0ZGI4MjU2MHwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:36 上传

### 5.3

扣出 `t` 定义之后,我们就要看 刚扣出这个 所以我们就可以找`d`的定义处 要像上找，

因为要使用 `d` 那就要先定义后使用 ，找到之后看到 `d` 是某个某块赋值的 那么这种格式就和`webpack`很相似

那么对于`webpack` 我们就要去找加载器

![图片27.png (29.72 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133739q2lnnm4syxy7x2cz.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1N3w2YzhhODFhMXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:37 上传

### 5.4

我们按照下图操作 ，根据多次调试 ，这个站可以只扣取需要的模块就可以 ，

所以我们将他自带的模块一并清除 只放入我们需要的就行，然后我们导出加载器

![图片28.png (44.74 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133822qgpfng3cepcepnwg.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1OHxjOGJmYjA1NHwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:38 上传

### 5.5

按照一二三步进行处理 第二步的作用就是打印出所用到的模块，

可以帮助我们快速知道是哪个模块没有 第二步因该是 `console.log(e);`

![图片29.png (44.18 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/133957irh3de7x7sxxrskk.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU1OXxjODdhN2IzMXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:39 上传

![图片30.png (31.51 KB, 下载次数: 0)](https://attach.52pojie.cn/forum/202503/03/134050leo66mmmj8b6b4u7.png)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc1ODU2MHwyOThmMmYwOXwxNzkzNTcxMjUwfDE2ODA4NnwyMDEwOTkx&nothumb=yes)

2025-3-3 13:40 上传

## 第六步

### 6.1

那么接下来我们就要根据浏览器调用的模块我们也调用 `d`是由 `t("9816");`

赋值的所以我们也要这样用和浏览器保持一样，可以看到因为我们没有`9816` 这个模块

所以报错并且给我们打印出了当前所需要的模块
