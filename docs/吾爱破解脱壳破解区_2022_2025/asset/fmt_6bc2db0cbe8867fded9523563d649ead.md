
# [CTF] 【官方WP】第一届solar杯·应急响应挑战赛官方题解

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1998153
- 抓取时间：2026-11-01T22:17:06

就在几天前，第一届Solar杯·应急响应挑战赛圆满落下帷幕。这场比赛汇聚了来自全国的网络安全精英，选手们在激烈的技术较量中展现了非凡的能力和智慧。为延续比赛的技术交流与分享精神，我们很高兴向大家呈现本次比赛的官方WP（Writeup）。本篇官方WP将深入解析比赛中的关键赛题，从流量分析到勒索病毒破解，逐步还原真实案例中的技术细节与解决思路。需要特别说明的是，文中所呈现的这些方法和思路，均基于真实应急响应场景设计，虽然可能并非所有情况下的最优解，但我们希望它们能够为您提供有价值的参考，助力您在未来的应急响应中更加高效地应对复杂多变的安全挑战。让我们一同走进这场技术盛宴，探索网络安全应急响应的更多可能！

## 1. 流量分析

### 1.1 文件排查

新手运维小王的Geoserver遭到了攻击：

黑客疑似删除了webshell后门，小王找到了可能是攻击痕迹的文件但不一定是正确的，请帮他排查一下。

从日志分析得知攻击者ip为10.0.100.22对该站点有目录扫描行为与`b.jsp`交互频繁疑似为上传webshell

![1.1.1.PNG](https://attach.52pojie.cn/forum/202501/08/145354s9z1xqunvk7v9b1q.png)  
1.1.1.PNG（563.68 KB，下载次数: 1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4MHxkNzBlMDkzZnwxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:53 上传

当访问 JSP 页面时，Tomcat 会根据 JSP 页面动态生成一个 Java 类（`.java`），然后将其编译为 `.class` 文件。生成的 `.class` 文件是 Tomcat 用来处理请求并返回响应的实际代码。这个过程是动态的，Tomcat 会在后台管理这些文件的编译和更新。路径一般为：`/work/Catalina///org/apache/jsp/`

查看`b_jsp.java`判断为哥斯拉webshell

![1.1.2.PNG](https://attach.52pojie.cn/forum/202501/08/145402x6dj98vun6qlblyn.png)  
1.1.2.PNG（314.25 KB，下载次数: 2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4MXw0OWU4Zjk1MnwxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

`base64`解码`code`

![1.1.3.PNG](https://attach.52pojie.cn/forum/202501/08/145405jfszh65b9jtis94y.png)  
1.1.3.PNG（23.79 KB，下载次数: 2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4MnwzNTJhZGQ5N3wxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

```
f!l^a*g{A7b4_X9zK_2v8N_wL5q4}
```

### 1.2 流量解密

新手运维小王的Geoserver遭到了攻击：

小王拿到了当时被入侵时的流量，其中一个IP有访问webshell的流量，已提取部分放在了两个`pcapng`中了。请帮他解密该流量。

![1.2.1.PNG](https://attach.52pojie.cn/forum/202501/08/145408z32mmsml49x44vfs.png)  
1.2.1.PNG（246.35 KB，下载次数: 1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4M3xmMmM1ZGY1NXwxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

![1.2.2.png](https://attach.52pojie.cn/forum/202501/08/145409gtmm4tclmagmm6km.png)  
1.2.2.png（204.51 KB，下载次数: 2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4NHw5Y2JiNjBlOHwxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

![1.2.3.png](https://attach.52pojie.cn/forum/202501/08/145412hgz8fdm7atxalffh.png)  
1.2.3.png（221.72 KB，下载次数: 2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4NXxjYThjY2U3ZXwxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

```
flag{sA4hP_89dFh_x09tY_lL4SI4}
```

### 1.3 文件提取

新手运维小王的Geoserver遭到了攻击：

小王拿到了当时被入侵时的流量，黑客疑似通过webshell上传了文件，请看看里面是什么。

使用流量解密工具进行解密流量并删除无用部分后另存为`pdf`

![1.3.1.png](https://attach.52pojie.cn/forum/202501/08/145416gxlrwc0awkcxthxv.png)  
1.3.1.png（834.03 KB，下载次数: 2）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4Nnw3MTczMmU3M3wxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

![1.3.2.png](https://attach.52pojie.cn/forum/202501/08/145420u46fr6nhfylnhk4e.png)  
1.3.2.png（336.39 KB，下载次数: 0）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4N3wwMzM4OWExYXwxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传

![1.3.3.png](https://attach.52pojie.cn/forum/202501/08/145423it1w5bbc4btka2bb.png)  
1.3.3.png（86.11 KB，下载次数: 1）  
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0ODQ4OHxkY2QzMWVkY3wxNzkzNTcxNDIzfDE2ODA4NnwxOTk4MTUz&nothumb=yes)  
2025-1-8 14:54 上传
