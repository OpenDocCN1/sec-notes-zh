
# [PC样本分析] Shiz病毒新变种出现 多种对抗窃取信息

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1879434`
- 抓取时间: `2026-11-01T23:48:36`

近期，火绒威胁情报系统监测到`Shiz`病毒的新变种正在快速传播。`Shiz`病毒主要针对国外用户群体，在被激活后能够窃取用户电脑上的敏感信息，并可以执行其他恶意操作。不仅如此，当受害者访问杀毒软件网址时，还会被劫持到`google`的网址，对用户构成较大干扰。病毒运行后，首先，会使用对抗手段检测环境和杀毒软件，捕获的变种样本使用了多种对抗手段，除了采用`shellcode`分块执行外，还通过`PUSH RET`来混淆`IDA`反编译的调用流等手段对抗杀软；随后，将恶意模块注入到系统进程中执行，进行数据窃取、屏幕截图、`DNS`劫持等恶意操作。该病毒执行流程，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/171852akdj76wsgtpyiwjd.png)

病毒执行流程图

目前，火绒安全产品可对上述病毒进行拦截查杀，请火绒用户及时更新病毒库以进行防御。火绒工程师建议大家，安装并定期更新杀毒软件、防火墙和安全补丁，确保始终保持最新的安全防护。

![](https://attach.52pojie.cn/forum/202401/11/171903pzhl3yooyklyzz3w.png)

查杀图

## 一、样本分析

### 混淆和对抗手段

病毒启动之后由最外层的`Loader`来进行加载，在最外层的`Loader`中具有多种混淆方式，通过执行大量垃圾代码来混淆安全人员的分析，还会无意义地调用冷门`API`这种方式，检测虚拟行为沙盒环境的真实性，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/171922lquuu4ozeiqnufue.png)

混淆代码

为了防止安全人员分析，该病毒在最外层的`Loader`中还使用`PUSH + RET`的方式来进行函数调用，这样可以对`IDA`的反编译功能进行混淆，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/171934e56szwhhlhldlxwe.png)

`PUSH RET`混淆

通过解密执行`shellcode`，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/171946lpsoy3gdieeszdgg.png)

执行`shellcode`

在`shellcode`执行过程中，它被分成多个块顺序执行。执行完一个块后，执行过的块会被加密，然后再解密并执行下一个块。相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/171957j8rrvllhhgyyl8yg.png)

`shellcode`加解密执行

`shellcode`主要负责从资源中获取内层模块，解密并加载到内存中，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172009zzcr1z0hscrc4h11.png)

内存加载内层模块

在内层模块中会检测虚拟机环境，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172023o4cvmmttt8nvvvtb.png)

检测虚拟机环境

还通过调用`Windows`防火墙`API`将自身添加到`Windows`防火墙的授权应用程序列表中，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172036wa09awmrnrr6mlnr.png)

添加到防火墙白名单

该模块还会将自身拷贝到添加到`C:\Windows\apppatch`目录中，并添加开机自启动中进行持久化操作，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172051v51d7gx0jzga3e04.png)

添加自启动项

该模块执行初始化操作之后，会将`shellcode`注入到系统进程中执行，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172110dslsa3t6s9tnleli.png)

注入系统进程执行恶意模块

`shellcode`主要负责将最终的恶意模块进行内存加载，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172122quznnbeb5nskbssb.png)

`shellcode`

### 内层恶意模块

该恶意模块使用`DGA`域名生成算法来动态产生大量的域名，这样做的好处是可以避开基于静态黑名单的防御系统，因为这些系统通常是通过拦截已知的恶意域名来阻止恶意流量的，黑客可以随时注册其中的域名来使用，这样传统的基于签名的防御方法往往无法及时响应这种动态生成的域名，从而使得恶意软件能够持续与`C&C`服务器进行通信，即使其中一些域名被识别并加入黑名单。`DGA`算法，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172137qat9tunurrgrjhto.png)

使用自定义`DGA`算法生成域名

`DGA`算法计算出的部分域名列表，分析过程中，暂未发现存活的`C&C`服务器，不排除后续这些域名会被注册，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172150yqq4efnnh4hhr9u9.png)

部分域名列表

通过`DGA`域名算法，计算出`C&C`服务器之后，会连接`C&C`服务器，执行`C&C`服务器下发的命令，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172205hxoo9hxolmtwihzj.png)

指令`C&C`服务器下发的命令

#### `load`命令

从`C&C`服务器下载恶意模块并执行，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172219myt5ze9ztdxlf9et.png)

`load`命令

#### `kill_os`命令

清空磁盘`MBR`和删除关键注册表项，导致系统出错，相关代码，如下图所示：

![](https://attach.52pojie.cn/forum/202401/11/172232gf6jljfpil262djc.png)
