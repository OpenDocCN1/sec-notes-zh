
# [Android 原创] [原创]某视频软件得登录分析（想不到既然上传手机大量的设备信息）

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1866860
- 抓取时间: 2026-11-01T21:36:17

为了发新帖子注册的吾爱，大家支持一下

免责声明：该帖子仅用于学习交流，不做其他用途

## 准备工作：

包：`5ZOU5ZOp5ZOU5ZOpXzYuMTguYXBrCnR2LmRhbm1ha3UuYmlsaQ==`

该软件涉及 `hook` 检测和抓包检测：

`hook` 检测参考：[https://bbs.kanxue.com/thread-277034.htm](https://bbs.kanxue.com/thread-277034.htm)

抓包：用 `lsp` 的 `JustTrustMe`

- `frida`：16.1.3
- `frida-tools`：12.2.1
- 安卓10
- `python` 3.11.4

## 抓包：

好了，下面就可以开心的使用使用 `frida` 进行 `hook` 抓包了

先抓一个登录包

![](https://attach.52pojie.cn/forum/202312/28/180437p69j44s9y0hmvmd4.png)

`image.png` (61.15 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYzM3w5ZDQxYzA0NXwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-28 18:04 上传）

登录包是这样的，有四个参数需要处理的

### `sign`:

`hook` `newStringUtf` 可以找到 `sign` 的值

```javascript
function hook_netString(){
  var symbols = Process.getModuleByName("libart.so").enumerateSymbols();
  var newStringUtf = null;
  for (let i = 0; i 
```

打印调用堆栈可以定位到这个 `so`

```text
args:  2d0fd1f3b61fd50921c0f71a3ee2cf0b
0xb2b611a5 libbili.so!0x31a5
```

定位到了这个地方

![](https://attach.52pojie.cn/forum/202312/13/144910rln2687m1it7q86t.png)

`image(1).png` (126.59 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYzNHxkYTBhN2U0ZHwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-13 14:49 上传）

![](https://attach.52pojie.cn/forum/202312/13/144819q1bv1vr1prrp3uhe.png)

`image(2).png` (69.01 KB, 下载次数: 2)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYxMXwxMzdmNTRhMnwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-13 14:48 上传）

分析一下函数的来源，上面几个函数很关键的

下面是挨个进去看一下，这里面都做了啥

![](https://attach.52pojie.cn/forum/202312/13/144822y47ykawabizf7d4q.png)

`image(3).png` (21.69 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYxMnw1MzA3YTRkZnwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-13 14:48 上传）

很幸运是的第一个函数里面，对MD5的模数进行初始化了

拿下面就是MD5相关的函数了

下面找一个MD5的源码对着函数进行重名了和类型的修改

![](https://attach.52pojie.cn/forum/202312/13/144824qcivg7v99g7lvvmz.png)

`image(4).png` (48.26 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYxM3w2NTQ5MDJhZHwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-13 14:48 上传）

修改好了就是这样的，代码简洁明了，就是加盐的MD5

下面对 MD5 的 `MD5Update` 和 `MD5Final` 进行 `hook`

```javascript
let libbili = Module.findBaseAddress("libbili.so");

if (libbili != null) {
    console.log("so addr:", libbili);
    Interceptor.attach(libbili.add(0x2F88 + 1), {
        onEnter: function(args) {
            console.log("明文1:", JSON.stringify((args[1])), args[2], args[3]);
        }
    })
    Interceptor.attach(libbili.add(0x22B0 + 1), {
        onEnter: function(args) {
            console.log("明文1:", ptr(args[1]).readCString(), hexdump(args[1]))
        }
    })
    Interceptor.attach(libbili.add(0x2AE0 + 1), {
        onEnter: function(args) {
            this.args1 = args[0];
        },
        onLeave: function(args) {
            console.log("返回：", ptr(this.args1).readCString(), hexdump(this.args1))
        }
    })
}
```

打印参数的时候这里调用两个，第一个包后面分析，我们先看这个 `sign` 的加密

![](https://attach.52pojie.cn/forum/202312/13/144826g54fi4f2wvejpa4x.png)

`image(5).png` (200.95 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYxNHwyZTRlMzY1NXwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-13 14:48 上传）

ok成功抓到 `sign`，看一下是怎么来的

这里第一个 `MD5Update` 里面是明文数据

后面三个加盐的数据

最后两个是 `MD5Final` 里面调用的，可以忽略了

明文就是前面所有参数的总和：太长了就不贴了，后面会分析都是啥

后面加盐的4个参数是这样的：

- `60698ba2`
- `f68e01ce`
- `44738920`
- `a0ffe768`

拼接在后面，使用MD5加密后，和我们上面抓到的数据一模一样的

![](https://attach.52pojie.cn/forum/202312/13/144829g236g62jjuup3ufv.png)

`image(6).png` (85.05 KB, 下载次数: 1)

[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2MzYxNXxkODFhNmNhZHwxNzkzNTY4OTc0fDE2ODA4NnwxODY2ODYw&nothumb=yes)（2023-12-13 14:48 上传）

这四个参数在我的手机上面是固定的，如果对于怎么来的比较感兴趣在分析一下

看一下java层哪里调用了一个函数，使用这个脚本

```javascript
function hook_RegisterNatives() {

    var symbols = Process.getModuleByName("libart.so").enumerateSymbols();

    var RegisterNatives_addr = null;

    for (let i = 0; i 
```

定位到了这个函数，通过 `hook` 发现 `i2` 是 1，`i3` 是 0
