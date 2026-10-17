
# 信息安全理论与实践

冯　涛

兰州理工大学计算机与通信学院

## 背景

➢ 信息安全的第一次变革：数据处理设备

自动化的工具保护文件和其他存储信息

| 物理机制：加载<br>管理机制：检查 | → | 计算机安全 |

➢ 信息安全的第二次变革：网络与通信设施

网络安全措施保护数据

| 计算机安全 | → | 网络安全 |

## 计算机安全:

- 提供自动化信息系统的保护
- 确保信息系统资源的完整性、有效性和机密性
- 信息系统资源包括硬件,软件,防火墙,信息和数据库以及通信安全

![](img/682122af1912e8a90686cd55e4d3ecdb_4_0.png)

## 课程目标

- 我们主要讨论密码学理论与实践
- 在信息传输或存储期间使用的一系列方法
- 阻止、预防、检测等方法以及纠正信息传输中出现的安全问题的措施等。

![](img/682122af1912e8a90686cd55e4d3ecdb_5_0.png)

## 安全威胁案例

- A向B传送文件，C无权接该文件，但是监听并截获
- A向B发出命令，C截获并修改后发送给B
- C假冒A向B发送命令
- A向B发出命令，事后A否认发出命令
- A向B发出命令，C截获并延迟发送给B

![](img/682122af1912e8a90686cd55e4d3ecdb_6_0.png)

## 信息安全面临的挑战

1. 满足安全机制非常复杂
2. 必须考虑潜在的、隐藏的攻击
3. 特定的处理过程经常违反直觉
4. 安全机制必须考虑使用场所
5. 安全机制包括多个算法与协议
6. 攻击者和设计者、管理者之间的较量
7. 安全的好处不能直接感受
8. 需要持续的监视
9. 经常是设计开始没有考虑
10. 被看成是使用系统的障碍

## 安全概念及关系

![](img/682122af1912e8a90686cd55e4d3ecdb_8_0.png)

## 安全概念及关系

![](img/682122af1912e8a90686cd55e4d3ecdb_9_0.png)

图1-2 安全概念及其关系

## 脆弱性与攻击

- 系统资源的脆弱性可能是：
  - 被破坏（缺乏完整性）
  - 泄漏（缺乏保密性）
  - 不可用（缺乏可用性）

- 攻击是被实施的威胁
  - 被动的
  - 主动的
  - 内部的
  - 外部的

## 对策

- 用于对付攻击的方法：防止、检测、恢复
- 可能导致新的弱点
- 可能残留弱点
- 目标是最大限度的降低风险

## 威胁后果

> 未授权的泄露
  - 暴露, 截获, 推理, 入侵
> 欺骗
  - 冒充, 伪造, 抵赖
> 破坏
  - 失能, 损坏, 阻碍
> 篡夺
  - 盗用, 误用

## 计算机与网络安全事故分类

![](img/682122af1912e8a90686cd55e4d3ecdb_13_0.png)

| 分类 | 内容 |
|---|---|
| Attackers | Hackers；Spies；Terrorists；Corporate raiders；Professional criminals；Vandals；Voyeurs |
| Tool | Physical attack；Information exchange；User command；Script or program；Autonomous agent；Toolkit；Distributed tools；Data tap |
| Vulnerability | Design；Implementation；Configuration |
| Action | Probe；Scan；Flood；Authenticate；Bypass；Spoof；Read；Copy；Steal；Modify；Delete |
| Target | Account；Process；Data；Component；Computer；Network；Internetwork |
| Unauthorized result | Increased access；Disclosure of information；Corruption of information；Denial of service；Theft of resources |
| Objectives | Challenge status, thrill；Political gain；Financial gain；Damage |

## 计算机与网络安全事故分类

![](img/682122af1912e8a90686cd55e4d3ecdb_14_0.png)

## 脆弱性报告

![](img/682122af1912e8a90686cd55e4d3ecdb_15_0.png)

## 事件报告

![](img/682122af1912e8a90686cd55e4d3ecdb_16_0.png)

(b) 报告的事件

## 安全发展趋势

![](img/682122af1912e8a90686cd55e4d3ecdb_17_0.png)

## 安全发展趋势

![](img/682122af1912e8a90686cd55e4d3ecdb_18_0.png)

## 计算机安全损失

![](img/682122af1912e8a90686cd55e4d3ecdb_19_0.png)

Source: Computer Security Institute/FBI 2006 Computer Crime and Security Survey

## 计算机安全损失

![](img/682122af1912e8a90686cd55e4d3ecdb_20_0.png)

## 采用的安全技术

![](img/682122af1912e8a90686cd55e4d3ecdb_21_0.png)

Source: Computer Security Institute/FBI 2006 Computer Crime and Security Survey

## 采用的安全技术

![](img/682122af1912e8a90686cd55e4d3ecdb_22_0.png)

## OSI 安全框架结构

- ITU-T：X.800 = “OSI安全框架”
- 定义一种系统化的方法，该方法定义和提供了安全需求，以及对安全产品和政策进行评价和选择。
- 从理论上说，它为涉及的许多概念提供了一些有用的概貌

## X.800 关注的安全内容

- 安全攻击
- 安全机制
- 安全服务

![](img/682122af1912e8a90686cd55e4d3ecdb_24_0.png)

## 安全攻击

> 采用的任何危及组织信息安全的活动
> 信息安全是关于怎样去预防攻击、怎样检测针对信息系统的攻击
> 威胁与攻击细化的概念是不同的
> 有两种基本的攻击类型
  - 被动攻击：消息内容的泄露，流量分析（预防）
  - 主动攻击：伪装，重放，消息修改，拒绝服务

## 被动攻击

![](img/682122af1912e8a90686cd55e4d3ecdb_26_0.png)

Darth

read contents of message from Bob to Alice

Internet or other comms facility

Bob

Alice

## 主动攻击

![](img/682122af1912e8a90686cd55e4d3ecdb_27_0.png)

Darth

Capture message from Bob to Alice; later replay message to Alice

Internet or other comms facility

Bob

Alice

## 安全服务

> X.800:
> “它是通信开放系统的协议层提供的服务，确保系统或数据传输的安全”

> RFC 2828:
> “由系统提供的一种处理或通信服务，以对系统资源提供某种特定的保护”
