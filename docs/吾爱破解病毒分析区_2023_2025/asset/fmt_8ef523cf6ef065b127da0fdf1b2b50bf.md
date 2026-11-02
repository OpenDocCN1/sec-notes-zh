
# 注入挖矿木马部分流程

## 1.3 挖矿木马

该挖矿木马与注入器类似，正式挖矿之前也会检查沙箱相关模块、TPM 设备等内容，不同的是挖矿木马会实时检测 `Taskmgr.exe`、`ProcessHacker.exe`、`perfmon.exe`、`procexp.exe`、`procexp64.exe` 等系统监控相关工具，若检测到上述软件则会停止挖矿，若检测到上述软件关闭则会继续挖矿。

![检测系统监控相关进程](https://attach.52pojie.cn/forum/202512/30/182420g0lgh9ahy3llailw.png)

随后该木马通过请求 [`https://rentrys.co/GzueSqAf/raw`](https://rentrys.co/GzueSqAf/raw) 或 [`https://pastebin.com/raw/WcTE2iw1`](https://pastebin.com/raw/WcTE2iw1) 获取到挖矿配置，其中 `url` 键值为矿池 IP 和端口。

![代码2](https://attach.52pojie.cn/forum/202512/30/182448y397633461d176tz.png)

随后，向该 `url` 发送登录请求，账号与密码均为 `x`。说明，在登录过程中未设置钱包地址，或者类似于公共矿池，存在独立的账号和密码。基于此，可推测该矿池为私人矿池。

![代码3](https://attach.52pojie.cn/forum/202512/30/182513u32a66gjieutht3a.png)

随后矿池开始下发计算哈希的任务，随后计算 `blob` 的哈希，利用不同的 `nonce` 来计算，试图让计算出的哈希达到一定的条件 `target`。

![代码4](https://attach.52pojie.cn/forum/202512/30/182530js60qo26worbr77d.png)

随后受害者系统中的挖矿木马会根据下发的任务配置进行 RandomX 哈希算法碰撞计算，计算出小于 `target`（`0x00007fff`）的哈希值（`00003002`），并发送至矿池服务器中，从而给私人矿池提供算力。

![代码5](https://attach.52pojie.cn/forum/202512/30/182549tr5clbobss55x00b.png)

![检查哈希是否符合 `target` 的 XMRig 开源项目中函数](https://attach.52pojie.cn/forum/202512/30/182604p9g8o9oxmi28982z.png)

火绒安全团队在此郑重提醒广大用户：切勿触碰安全红线！网络黑产常常利用色情、[//www.52pojie.cn](https://www.52pojie.cn)类软件作为载体传播病毒，此类软件背后往往暗挖矿木马、勒索病毒等恶意程序，不仅会侵占设备算力资源、导致硬件损耗加速，还可能窃取用户隐私数据，造成多重安全风险。目前，元旦、春节等节假日将至，不少用户会选择通过电脑等终端设备进行休闲娱乐，火绒安全在此特别提醒，选择电脑游戏或下载各类软件时，务必通过官方官网、正规应用平台等可信渠道获取，坚决远离来源不明的资源，避免因一时疏忽给恶意程序可乘之机。如需反馈相关病毒样本，可联系火绒安全官方客服，我们将持续跟踪各类新型恶意程序，为用户提供可靠的安全防护保障，守护大家度过一个安全舒心的假期。

## 二、附录

### C&C：

![C&C](https://attach.52pojie.cn/forum/202512/30/182625ej4f4f5nw7ojysqq.png)

### HASH：

![HASH](https://attach.52pojie.cn/forum/202512/30/182635ht7bfff877zrf8hh.png)
