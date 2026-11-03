
# [PC样本分析] 【攻击手法分析】勒索病毒如何轻松绕过安全设备防线：第一篇-驱动漏洞一击致命
- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990710
- 抓取时间: 2026-11-01T23:46:20

引言：在之前的文章【成功案例】`lockbit`家族百万赎金不必付！技术手段修复被加密的数据库，附溯源分析报告]中，我们提到，尽管许多企业已经部署了众多安全设备和备份解决方案，并建立了相对成熟的安全运营体系，但在如此复杂的网络环境中，依然存在漏洞，导致黑客成功发起勒索加密攻击，最终造成业务瘫痪。通过对加密样本的提取和分析，我们识别出黑客使用了一些隐蔽的攻击手法，如`Killav`和致盲等，这些技术成功绕过了现有的安全防护，未能及时被检测到。本篇文章将深入解析黑客如何利用漏洞驱动`Kill AV`攻击的全流程，下一篇则将讨论如何配合驱动进行流量致盲，从而使部署的`EDR`无感地执行加密，导致机器被加密，敬请期待！

![](https://attach.52pojie.cn/forum/202412/12/173913abefbqj5uv569quf.png)
`0.png` (174.38 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc0MzIzNXwxODNlZWIwMHwxNzkzNTc2Nzc4fDE2ODA4NnwxOTkwNzEw&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzIzNXwxODNlZWIwMHwxNzkzNTc2Nzc4fDE2ODA4NnwxOTkwNzEw&nothumb=yes)
2024-12-12 17:39 上传
模拟客户网络拓扑环境

## 1.背景

### 1.1家族介绍
关于`RansomHub`家族信息详情可参考本篇文章[【病毒分析】`Ransomhub`加密器样本-EXSI ]

### 1.2防病毒软件（AV）测试结果
通过技术手段捕获`RansomHub`家族的`killAV`程序后，我们第一时间对国内外主流的防病毒软件（`AV`）和终端检测与响应（`EDR`）系统进行了测试，发现该程序能够成功地强制关闭绝大部分杀毒软件的防护功能，意味着它具备了强大的绕过防护机制的能力。具体测试结果如下：
杀软 | 结果 | 测试截图 | 公司及产品介绍 | `Sophos home`
