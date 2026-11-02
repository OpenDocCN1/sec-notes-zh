
数据以队列的方式在内存中准备，每条数据携带其生成时的时间戳和将被发送的加密数据，队列中可见 `SteamId`、`features` 等字段，推测这些数据为分散式队列加密发送程序在运行期间所收集到的信息。

[![抓包加密数据图](https://attach.52pojie.cn/forum/202510/27/215624uhnnjdbh1fyflw2n.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDEwMnxmMGYzMjI1YnwxNzkzNTc2NjEyfDE2ODA4NnwyMDY4Mjc2&nothumb=yes)

`抓包加密数据图.png` (871.44 KB, 下载次数: 0) — 2025-10-27 21:56 上传

6. 程序修改 Steam 配置项禁用自动更新。

   通过设置 Steam 根目录 `\steam.cfg` 中 `BootStrapperInhibitAll = Enable` 的方式，禁用 Steam 客户端自动更新，防止程序失效。

   [![禁用Steam客户端自动更新](https://attach.52pojie.cn/forum/202510/27/215437lffnjjgiv7m47gv6.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDEwMXw0ODllYzEwZHwxNzkzNTc2NjEyfDE2ODA4NnwyMDY4Mjc2&nothumb=yes)

   `禁用.png` (57.01 KB, 下载次数: 0) — 2025-10-27 21:54 上传

## 三、关键风险总结

1. 部分售卖该程序的商家在发布或推销时通过模糊的说辞欺骗并诱导用户下载程序。
2. 程序通过修改 Steam 客户端的方式提供服务，通过恶意手段破坏官方客户端的安全性以实施注入操作，使用户数据陷入风险。
3. 程序将用户的敏感数据，甚至是登录令牌收集至自身数据库进行存储与传输。
4. 程序所采用的一系列运行方式与病毒的行为特征极为相似。
5. 程序配置可以由云端服务器随时修改控制，存在极强的安全隐患。

   从 Gitee 仓库日志能够得知，在文章完成定稿的前 14 天，作者仍通过云端仓库对下发的数据进行修改。

   [![云端仓库下发数据](https://attach.52pojie.cn/forum/202510/27/215315vrxwmoz0x3arxn1a.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDEwMHw2NzUwYWExY3wxNzkzNTc2NjEyfDE2ODA4NnwyMDY4Mjc2&nothumb=yes)

   `云端仓库下发数据.png` (86.93 KB, 下载次数: 0) — 2025-10-27 21:53 上传

## 四、附录

### HASH

[![HASH](https://attach.52pojie.cn/forum/202510/27/220935stn5susnjuupbpbn.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDExM3xlMDVmZmE2MnwxNzkzNTc2NjEyfDE2ODA4NnwyMDY4Mjc2&nothumb=yes)

`hash.png` (65.97 KB, 下载次数: 0) — 2025-10-27 22:09 上传

### C&C

[![C&C](https://attach.52pojie.cn/forum/202510/27/220923xce83888ejjhn33x.png)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMDExMnw4M2U0M2M0ZnwxNzkzNTc2NjEyfDE2ODA4NnwyMDY4Mjc2&nothumb=yes)

`ccc.png` (51.42 KB, 下载次数: 0) — 2025-10-27 22:09 上传
