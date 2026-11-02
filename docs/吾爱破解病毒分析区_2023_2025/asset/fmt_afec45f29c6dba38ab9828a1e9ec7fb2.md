
## “互斥体”设置

### fork 的进程

样本后续通过 `fork` 分支了多个子进程，子进程之间互相关联且涉及到配置中的 `mgr_pid` 和 `monitor_pid` 进程，梳理如下：

![Image-17.png](https://attach.52pojie.cn/forum/202311/29/171801vqqd8pldh86lgkkn.png)

*`Image-17.png` (182.02 KB)*

## 流程图

![Image-18.png](https://attach.52pojie.cn/forum/202311/29/171809me8l1xyfx7yo8o8p.png)

*`Image-18.png` (9.75 KB)*

## 后台执行

接着 fork 出子进程 2，并设其为 `mgr_pid` 值，用于管理“上线”的操作。并接着在 `start_monitor_child` 函数中开启子进程 3，用于执行监控操作。在开启子进程 2、3 之后，子进程 1 进入到监听消息的循环当中，通过指定的消息执行不同的操作。

![Image-19.png](https://attach.52pojie.cn/forum/202311/29/171817z8oazdjpj9f7s70t.png)

*`Image-19.png` (31.89 KB)*

## 监听消息

监听的特定消息应由“上线”后以发出，包括 `restart`、`update`、`uninstall` 3 种管理方式。其中这三种操作的对象都指向进程 3 在 `install_path` 和 `install_path_bak` 中存放的最新配置的“启动器”：

![Image-20.png](https://attach.52pojie.cn/forum/202311/29/171826q3v0zbcucmi6thc2.png)

*`Image-20.png` (26.37 KB)*

## 管理分支进程

### 子进程3：配置更新

在 `start_monitor_child` 中开启的子进程 3 负责监控操作，其获取配置中的 `mon_interval`（10 秒）值作为监控周期，循环执行：

![Image-21.png](https://attach.52pojie.cn/forum/202311/29/171835kcxawte9y401axkx.png)

*`Image-21.png` (24.91 KB)*

**循环检测**

对于配置中设置了的 `"cmd%d"` 命令，则会在该进程中遍历执行：

![Image-22.png](https://attach.52pojie.cn/forum/202311/29/171852d4pp8ppch45onvn5.png)

*`Image-22.png` (42.92 KB)*

**命令执行**

该进程最关键的还是对于配置的更新，根据配置中的 `install_path` 和 `install_path_bak` 值，把自身复制到对应的目录下并设置 `is_worker` 为空（相当于启动器），目的是在“上线”后根据交互时刻更新命令配置。

![Image-23.png](https://attach.52pojie.cn/forum/202311/29/171902jpphilr8wgjjl2kg.png)

*`Image-23.png` (61.55 KB)*

**配置更新**

### 子进程2：维持通信

子进程 2 负责“上线”操作，在通信之前会校验防火墙配置，如果设置了值，则执行 `iptables -F` 清除防火墙规则：

![Image-24.png](https://attach.52pojie.cn/forum/202311/29/171914d79kg8eg559jl75l.png)

*`Image-24.png` (17.78 KB)*

**防火墙设置**

随后在循环中遍历配置中的 `host%d`，获取 C2 域名进行连接：（拽写本文时，C2 已失效）

![Image-25.png](https://attach.52pojie.cn/forum/202311/29/171923q1zuz5ow6a0emywo.png)

*`Image-25.png` (46.49 KB)*

**C2 连接**

在连接前，获取 DNS 解析的 IP 地址进行连接，构造特定的报文发送来获取特定的返回数据：

![Image-26.png](https://attach.52pojie.cn/forum/202311/29/171932nhriw6vylmkllxgn.png)

*`Image-26.png` (67.38 KB)*

**报文构造**

![Image-27.png](https://attach.52pojie.cn/forum/202311/29/171941qptiksgpjpz13arr.png)

*`Image-27.png` (58.45 KB)*

**流量查看**

连接成功后根据配置中的 `protocol` 来选择要使用的通信协议进行管理：

![Image-28.png](https://attach.52pojie.cn/forum/202311/29/171952k40m578mmv2mcsz7.png)

*`Image-28.png` (31.69 KB)*

**协议选择**

## 关于 SIGCHLD 信号

在前期对信号的处理操作中注册的 `SIGCHLD` 信号处理函数是对“工作者”的进程 2、3 进行处理的。因为当第一个子进程终结后，`SIGCHLD` 的处理函数还要等待第二个子进程终结才能进入条件。在该条件句中，执行的操作是重置 `mgr_pid` 和 `monitor_pid` 的值。

![Image-29.png](https://attach.52pojie.cn/forum/202311/29/172001bvyz177djsq774fm.png)

*`Image-29.png` (44.82 KB)*

**SIGCHLD 信号**

## 关于壳与配置

样本是用常规 `upx` 壳压缩的，并没有什么修改。但是对于配置部分，程序采取的是将加密后的配置附加在文件末尾，这就导致了自动[`//www.52pojie.cn/forum-5-1.html`](https://www.52pojie.cn/forum-5-1.html)失败：

![Image-30.png](https://attach.52pojie.cn/forum/202311/29/172023b83c1owd3w06coez.png)

*`Image-30.png` (25.59 KB)*

**配置写入**

![Image-31.png](https://attach.52pojie.cn/forum/202311/29/172034soiyqtoithjoaozt.png)

*`Image-31.png` (310.78 KB)*

**相关字节码**

## 二、附录

### HASH

![Image-32.png](https://attach.52pojie.cn/forum/202311/29/172045f9gg2x8z8q9bmx92.png)

*`Image-32.png` (20.29 KB)*

### C&C

![Image-33.png](https://attach.52pojie.cn/forum/202311/29/172053frocfajx6gfxjzf6.png)

*`Image-33.png` (1.12 KB)*
