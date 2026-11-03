
# [PC样本分析]【病毒分析】交了赎金也无法恢复--针对国内某知名NAS的LVT勒索病毒最新分析

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=1961868](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1961868)
- 抓取时间：2026-11-01T23:47:06

## 1. 背景

### 1.1 来源

接到客户通知发现NAS遭到勒索病毒攻击，其中的文件无法打开并且被添加了`lvt`后缀。经排查该次勒索攻击是一款名为LvtLocker的勒索软件所为，而该勒索软件的攻击目标主要是国内某知名的NAS设备系统。

### 1.2 入侵路线图

![](https://attach.52pojie.cn/forum/202409/06/091839o9ozhqjshlbb9sqo.jpg)

## 2. 溯源分析

### 2.1 查看日志

远程访问客户中招服务器，通过排查NAS web日志，默认SSH端口为9222，登录用户名为root，密码与web端admin用户相同。

查看nginx日志`/var/log/nginx/access.log`，可以发现在2024年3月1日黑客利用`CVE-2022-24990`漏洞远程下载木马、添加执行权限并执行的行为。

![](https://attach.52pojie.cn/forum/202409/06/091808qlnn71ja5q5x1aac.png)

### 2.2 排查异常文件

查看NAS web路径：`/usr/www/`，发现该系统在2023年就遭受过非勒索团伙的攻击，发现了遗留的webshell。

webshell地址：

```
/usr/www/my.php
/usr/www/include/dwz.php
```

最终通过在web目录中查找ELF文件发现加密器路径：

```
/usr/www/module/tnasupdate
```

![](https://attach.52pojie.cn/forum/202409/06/091810ywk11ene8nynenoy.png)

![](https://attach.52pojie.cn/forum/202409/06/091812c5hfaqeqoqmmmpe1.png)

### 2.3 漏洞验证

漏洞名称：任意命令执行漏洞（`CVE-2022-24990`）

POC：[https://github.com/lishang520/CVE-2022-24990/tree/main](https://github.com/lishang520/CVE-2022-24990/tree/main)

验证结果：存在漏洞

![](https://attach.52pojie.cn/forum/202409/06/091814iov8wnu2nke7p4k1.png)

### 2.4 钱包溯源

通过区块链浏览器查看勒索信中的黑客钱包地址：

```
3JD9xwA9HQuZSSEKZemmuTTpEmtM1L8Uwt
```

![](https://attach.52pojie.cn/forum/202409/06/091816bkei8aia8l8nek80.png)

查看该地址的交易记录，可得出结论：黑客每次进行勒索的金额为0.01BTC(约为4927元)，一共完成7次与受害者的交易，其中最早的交易可追溯到2024年2月23日。

![](https://attach.52pojie.cn/forum/202409/06/091818cqgqzf9el582deku.png)

最终在2024年2月29日黑客将所有BTC转入新地址：

```
1LLh7KvmjmJuEELxSN2PKQKEywt6FfrfYi
```

![](https://attach.52pojie.cn/forum/202409/06/091820mvocb9i9h5ccta7s.png)

后续有受害者尝试通过邮箱联系黑客，但均未得到答复，期间新地址钱包的BTC被转出，怀疑黑客已跑路。

## 3. 恶意文件基础信息

### 3.1 加密器基本信息

| 文件名 | 编译器 |
| --- | --- |
| lvt勒索病毒加密器 |  |
