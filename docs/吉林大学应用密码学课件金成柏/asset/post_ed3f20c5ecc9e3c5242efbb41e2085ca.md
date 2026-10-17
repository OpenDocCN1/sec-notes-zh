
## DSA——密钥生成

- 全局共享的公钥（p, q, g）
  - 选择大素数 p，长度为 L；其中，512 ≤ L ≤ 1024，且是64的倍数
  - 选择素数 q，是 p-1 的160 bit的素因子
  - 选择 g = h^((p-1)/q) (mod p)
    - 其中，h < p-1，h^((p-1)/q) (mod p) > 1
- 用户私钥与用户公钥
  - 选择私钥 x < q
  - 计算公钥 y = g^x (mod p)

## DSA——签名

- 创建签名随机数 k
  - k < q
  - k 必须是签名后被销毁，且不能被重用
- 计算签名 (r, s)
  - r = (g^k (mod p)) (mod q)
  - s = k⁻¹ · (SHA(M) + x·r) (mod q)

## DSA——验证签名

- 验证方收到消息 M 和签名 (r, s)
- 验证签名
  - w = s⁻¹ (mod q)
  - u1 = (SHA(M) · w) (mod q)
  - u2 = (r · w) (mod q)
  - v = (g^u1 · y^u2 (mod p)) (mod q)
  - 如果 v = r，则签名有效

## 《密码编码学与网络安全》

（第三版）[美]William Stallings

## 第14章 认证的实际应用

吉林大学
金成柏 jincb@sohu.com
2008年4月27日

## 内容提要

- Kerberos
- 版本4
- 版本5
- X.509认证服务
- 证书
- 认证过程

## Kerberos

- 作为MIT的Athena计划的认证服务而开发
- 提供了集中的可信第三方认证服务
  - 用户不需要与每个应用服务器进行认证
  - 只需在集中的认证服务器上进行一次认证即可
- 版本
  - 版本4
  - 版本5
    - 成为了Internet标准草案（RFC1510）

### Kerberos——需求

- 需求
  - 安全性（security）
    - 监听者不能通过冒充其他用户获取有用信息
  - 可靠性（reliability）
    - 具有高可靠性，并使用分布式服务结构
  - 透明性（transparency）
    - 用户除了要输入口令外，不需要知道认证过程
  - 可伸缩性（scalability）
    - 能支持大量的客户端和服务器，采用模块化、分布式结构
- 基于Needham-Schroeder的认证协议
  - KDC充当认证服务器

### Needham-Schroeder协议

![](img/0935d481fd26d10140d7cd560b6540ba_450_0.png)

### Kerberos 4——概述

- 可信的第三方认证机制
  - Authentication Server (AS)
    - 用户通过与AS协商来证实自己的身份
    - AS提供票据授权票据（ticket-granting ticket, TGT）
  - 票据授权服务器（Ticket Granting Server, TGS）
    - 用户使用TGT向TGS提出对其它应用服务器的访问请求
    - TGT提供访问应用服务器的服务授权票据
  - 用户
    - 请求服务
  - 应用服务器
    - 提供服务

1. User logs on to workstation and requests service on host.
2. AS verifies user's access right in database, creates ticket-granting ticket and session key. Results are encrypted using key derived from user's password.
3. Workstation prompts user for password to decrypt incoming message, then sends ticket and authenticator that contains user's name, network address, and time to TGS.
4. TGS decrypts ticket and authenticator, verifies request, then creates ticket for requested server.
5. Workstation sends ticket and authenticator to server.
6. Server verifies that ticket and authenticator match, then grants access to service. If mutual authentication is required, server returns an authenticator.

- once per user logon session:
  - request ticket-granting ticket
  - ticket + session key
- once per type of service:
  - request service ticket
  - ticket + session key
- once per service session:
  - provide server authenticator

- Kerberos
- Authentication Server (AS)
- Ticket-granting Server (TGS)

### Kerberos 4——简单的认证会话

- 存在的问题
  - 口令安全
    - 用户口令Pc采用明文传输，监听者可窃取口令
    - 在该工作站上冒充该用户
    - 将自己的工作站配置为同样的网络地址来冒充该用户
  - 每次服务请求都需要输入口令进行认证
    - 同一个服务类型的多次访问
    - 不同服务类型的多次访问

### Kerberos 4——更安全的认证会话

- 会话
  - 每个用户仅一次的登录会话
    - (1) C → AS: IDc || IDtgs
    - (2) AS → C: EKc[Ticket tgs]
  - 每种服务类型仅一次
    - (3) C → TGS: IDc || IDv || Ticket tgs
    - (4) TGS → C: Ticket v
  - 每次服务会话仅一次
    - (5) C → V: IDc || Ticket v
  - Ticket tgs = EKtgs[IDc || ADc || IDtgs || TS1 || Lifetime1]
  - Ticket v = EKv[IDc || ADc || IDv || TS2 || Lifetime2]
- 问题的解决
  - 票据授权服务器（TGS）
    - 解决了用户口令多次输入的问题
  - 口令生成密钥 EKc
    - 解决了口令明文传输的问题。只有用户才能够生成密钥解密 EKc[Ticket tgs]

### Kerberos 4——版本4的认证会话

| 认证交换消息 |
| --- |
| **(a) Authentication Service Exchange: to obtain ticket-granting ticket** |
| (1) C → AS: IDc \|\| IDtgs \|\| TS1 |
| (2) AS → C: EKc[Kc,tgs \|\| IDtgs \|\| TS2 \|\| Lifetime2 \|\| Ticket_tgs] |
| Ticket_tgs = EKtgs[Kc,tgs \|\| IDc \|\| ADc \|\| IDtgs \|\| TS2 \|\| Lifetime2] |
| **(b) Ticket-Granting Service Exchange: to obtain service-granting ticket** |
| (3) C → TGS: IDv \|\| Ticket_tgs \|\| Authenticatorc |
| (4) TGS → C: EKc,tgs[Kc,v \|\| IDv \|\| TS4 \|\| Ticketv] |
| Ticketv = EKv[Kc,v \|\| IDc \|\| ADc \|\| IDv \|\| TS4 \|\| Lifetime4] |
| Authenticatorc = EKc,tgs[IDc \|\| ADc \|\| TS3] |
| **(c) Client/Server Authentication Exchange: to obtain service** |
| (5) C → V: Ticketv \|\| Authenticatorc |
| (6) V → C: EKc,v[TS5 +1] (for mutual authentication) |
| Authenticatorc = EKc,v[IDc \|\| ADc \|\| TS5] |
