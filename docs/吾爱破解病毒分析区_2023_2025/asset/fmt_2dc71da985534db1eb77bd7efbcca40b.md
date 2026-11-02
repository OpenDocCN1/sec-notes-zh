
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzAzOXw4MDFmZjRjMHwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:10 上传*

*杏仁桉域名奇安信威胁分析结果*

根据“天津拂云科技有限公司”法人“刘科”的杏仁桉域名邮箱“liuke@apkevery.com”在搜索引擎中进行搜索，可以搜索到其提交代码对应的一个自动构建网站记录。

![天津拂云联系邮箱搜索结果](https://attach.52pojie.cn/forum/202511/11/162224dd0chq9zl3l5wcmc.png)

*天津拂云联系邮箱搜索结果.png（44.36 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0MHxhNDU5MTQ0MHwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:22 上传*

再针对该网站进行搜索所有提交记录的邮箱地址，可以发现相关人员的QQ邮箱。

![其他提交构建信息搜索结果](https://attach.52pojie.cn/forum/202511/11/162243yb5xmbm7nnwexzqn.png)

*其他提交构建信息搜索结果.png（24.22 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0MXw4ZmExZmRhNHwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:22 上传*

通过对 QQ 邮箱进行对应搜索查找相关 QQ 账号，发现该 QQ 昵称由较为特殊的三个字构成，据此推断此昵称可能为姓名。进一步访问`mail.ludashi.com`网站，利用其找回密码功能获取信息，发现所得到的手机尾号与通过 QQ 找回密码流程中显示的尾号一致。综上所述，基本可以判定为同一人。

## 2.4 小鸟壁纸信息查询

![根据小鸟壁纸搜索得到的工作日志网站](https://attach.52pojie.cn/forum/202511/11/162256ybpabvpcoelbbpze.png)

*根据小鸟壁纸搜索得到的工作日志网站.png（37.91 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0Mnw0YTk3OTZjN3wxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:22 上传*

当在搜索引擎中以关键词`360wallpaper_bird`搜索小鸟壁纸安装包时，可以查询到一个记录了一系列工作日志的网站信息，即1.4节中提到的外网工作日志。在网站中，可见1.4节中提到的一系列信息。

![网站中可见的部分信息内容](https://attach.52pojie.cn/forum/202511/11/162309fn2p00n9l51nobb1.png)

*网站中可见的部分信息内容.png（46.14 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0M3wwMjQ3YzA3N3wxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:23 上传*

# 【三】手段揭露

为了防止其具有争议性的推广行为被技术人员分析发现、被公众舆论讨论揭露，躲避自动化分析软件的监控和拦截，隐藏背后编织的巨型利益输送网络，这些推广模块采用了多种技术对抗手段。它们通过云控配置实时调整自身行为，专门规避技术人员等高危群体，精准针对普通用户投放。推广模块会根据用户的系统环境、地理位置、使用时长、是否充值等多维度信息判断用户价值，仅对“安全”目标、“小白”用户启动推广行为，使推广行为更加隐蔽。

## 3.1 各项针对检测

从服务器下发云控配置开始，软件就开始了其针对性的检测：

- 地区相关：以鲁大师为例，软件会根据用户所在地区针对性的投放推广云控配置。测试得出，对北京地区的用户会减少或不下发推广相关的云控配置。
- 渠道相关：有些软件会专门区别用户是否从软件官网下载，并且针对从非官网渠道下载的用户推送推广云控配置，而官网用户将会减少或不下发推广相关云控配置。
- 用户相关：以鲁大师为例，获取云控时会通过遍历检测当前系统信息的方式判断是否为技术人员、是否在[//www.52pojie.cn/thread-661779-1-1.html](https://www.52pojie.cn/thread-661779-1-1.html)中、是否为业务会员等相关数据，从而针对性调整云控配置。

## 3.2 多种推广方式

为了实现其推广目的，掇取用户流量价值，相关软件实现了多种多样的推广方式：

推广模块以可动态载入的插件的形式存在，从而使得大部分推广策略都可以在云端动态调整。下图中展示的内容为鲁大师软件中包含的部分推广方式。

![多种推广方式](https://attach.52pojie.cn/forum/202511/11/162326ar0jp555ejcw0a5w.png)

*多种推广方式.png（461.66 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0NHxiMzg5NTM4NnwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:23 上传*

## 3.3 各种规避手段

为与用户及监管进行“捉迷藏”，软件采用多种方式以规避各类情形。

![规避手段表](https://attach.52pojie.cn/forum/202511/11/162343wo973gvpo7g7z307.png)

*规避手段表.png（142.16 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0NXxiMzQ4MDlhNnwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:23 上传*

自上次监控到“重庆赫赫有盾科技有限公司”开发的软件“DXRepair”利用云控下发推广配置以来，火绒安全情报中心一直持续关注相关威胁情报的动向。外网数据显示，包含类似推广模块的软件仍在大量传播。其中，一款名为“TabXExplorer”的软件同样会弹窗安装软件并劫持浏览器，其签名信息显示来源为“成都奇鲁科技有限公司”。

鲁大师作为在国内具有极高传播度的硬件检测软件，许多用户在新购置电脑时会安装此软件，用以开展跑分测试并查看硬件信息。然而，用户很难预料到，这一看似正常的使用行为，会在未得到充分告知或被故意模糊告知的情形下，使自己的电脑沦为相关企业的“流量变现工具”。

# 【四】技术分析

本次主要围绕鲁大师软件展开技术分析，并提出该系列推广软件的共性。在溯源过程中，我们发现小鸟壁纸等软件同样包含类似的推广模块，其中Lua脚本执行模块和云控配置请求模块是这些带有推广插件的软件的共同特征。

## 4.1 云控配置的下发

不同软件中的云控配置请求模块名称各异，但功能相同。例如：

![云控模块名称表](https://attach.52pojie.cn/forum/202511/11/162401g5qha05tetnnmmr5.png)

*云控模块名称表.png（53.76 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0Nnw0MDdlYjdjN3wxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:24 上传*

在软件中通常有一个控制大部分推广行为的云控配置，下文中称为“总体云控配置”，在该配置中包含大部分的推广相关配置。

### 4.1.1 加密算法

本次分析中，所有软件的推广模块基本采用相同的加密方式：使用 BlowFish 算法对原始数据加密，再通过 Base64 算法编码后进行数据收发。解密操作主要发生在以下场景：

接收服务器云控配置、解析云控配置字段值、下载 Lua 脚本、读取软件中以加密形式存储的配置文件。

![数据加密不同场景](https://attach.52pojie.cn/forum/202511/11/162416h89ty27m8twck3zw.png)

*数据加密不同场景.png（336.79 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0N3xkZmFiMjU2ZXwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:24 上传*

### 4.1.2 规避

分析过程中发现，推广模块在获取云控配置时也会进行规避，根据用户地区和下载渠道筛选推广目标，避免向高风险用户投放从而规避可能的风险。

#### 4.1.2.1 地区规避

整体云控配置下发：

云控配置包含大量推广相关参数。通过测试发现，非北京IP与北京IP获取到的配置存在显著差异，通常非北京IP会收到更多的推广配置，具体对比如下所示。

![整体云控配置下发地区区别](https://attach.52pojie.cn/forum/202511/11/162430uz306ixw8x5iq00x.png)

*整体云控配置下发地区区别.png（172.16 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0OHxlNWU0ODRlYXwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:24 上传*

获取京粉佣金链接：

在获取京粉佣金链接时，系统同样会对北京IP进行规避。例如，北京IP获取的云控配置中`data.land_url`字段为空，而太原等其他地区的IP则会返回该字段的具体值，如下所示。

![云控配置下发规避北京区域](https://attach.52pojie.cn/forum/202511/11/162447a0xi596yot4fczt5.png)

*云控配置下发规避北京区域.png（215.69 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA0OXw2YWUyMjJiMHwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:24 上传*

#### 4.1.2.2 渠道

非官网渠道下载会产生更多推广：

一般情况下，其他公司的官网渠道代号为`home`，鲁大师的官网渠道代号为`ludashi`。经过多次测试发现，通过官网渠道下载的软件在推广行为上会有所收敛：减少或不下发推广相关的云控配置，又或者在安装包中减少或移除推广模块。

例如，迅读 PDF 大师会根据访问官网时的URL参数动态切换安装包下载地址。官网渠道提供的安装包不包含推广插件，而其他渠道的安装包则内置推广插件。

![不同渠道不同安装包](https://attach.52pojie.cn/forum/202511/11/162510t6343nnq5f0xv33c.png)

*不同渠道不同安装包.png（208.12 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA1MHw1MGM5YzgyMnwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:25 上传*

#### 4.1.2.3 用户环境

请求云控配置时，客户端会上报环境检测数据，包括：`sr`（杀毒软件）、`vm`（虚拟机）、`vip`（会员状态）、`advance`（360 广告弹窗开关与弹窗拦截开关）、`dev`（技术人员检测）等信息，服务端据此动态下发差异化的推广配置。

`sr`（杀毒软件）：

检查`SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`注册表路径中`DisplayIcon`与`InstallLocation`键值中是否存在以下杀软程序名，如果存在，则将其收集到`sr`键值中。

![杀毒软件程序名](https://attach.52pojie.cn/forum/202511/11/162552mva2xex6qbg4ccba.png)

*杀毒软件程序名.png（16.11 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA1MXwxMDM1NjZjMXwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)

*2025-11-11 16:25 上传*

`vm`（虚拟机）：

检测`HKEY_CLASSES_ROOT\Applications\VMwareHostOpen.exe\shell\open\command`是否存在且默认值有效则会判断为虚拟机。

![检测虚拟机](https://attach.52pojie.cn/forum/202511/11/162608zzilbhcjhqb2sjdj.png)

*检测虚拟机.png（55.75 KB，下载次数：0）*

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxMzA1MnwxZjg3ZDk4MHwxNzkzNTc2NjEwfDE2ODA4NnwyMDcxNTUx&nothumb=yes)
