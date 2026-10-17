

## Table 14.4 Kerberos Version 5 Flags

| INITIAL | This ticket was issued using the AS protocol and not issued based on a ticket-granting ticket. |
|---|---|
| PRE-AUTHENT | During initial authentication, the client was authenticated by the KDC before a ticket was issued. |
| HW-AUTHENT | The protocol employed for initial authentication required the use of hardware expected to be possessed solely by the named client. |
| RENEWABLE | Tells TGS that this ticket can be used to obtain a replacement ticket that expires at a later date. |
| MAY-POSTDATE | Tells TGS that a postdated ticket may be issued based on this ticket-granting ticket. |
| POSTDATED | Indicates that this ticket has been postdated; the end server can check the authtime field to see when the original authentication occurred. |
| INVALID | This ticket is invalid and must be validated by the KDC before use. |
| PROXIABLE | Tells TGS that a new service-granting ticket with a different network address may be issued based on the presented ticket. |
| PROXY | Indicates that this ticket is a proxy. |
| FORWARDABLE | Tells TGS that a new ticket-granting ticket with a different network address may be issued based on this ticket-granting ticket. |
| FORWARDED | Indicates that this ticket has either been forwarded or was issued based on authentication involving a forwarded ticket-granting ticket. |

## 14.2 X.509认证服务

- X.509是X.500系列中定义目录服务的一部分，目录是指管理用户信息数据库的服务器或一组分布式服务器，用户信息包括用户名到网络地址的映射等
- X.509定义了X.500用户目录的一个认证服务框架，目录提供公钥证书库服务，还定义了基于公钥证书的一个认证协议
- X.509是关于证书结构和认证协议的一种重要标准
- X.509首次于1988年发布，1995年第3版，2000年再次修改
- X.509是基于公钥密码体制和数字签名的服务，推荐使用RSA，数字签名需要用到散列函数

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_309_0.png)

### Figure 14.3 Public-Key Certificate Use

## 14.2.1 X.509证书

- X.509证书由可信认证机构Certification Authority（CA）创建并放入目录服务器中，包括
  - 版本号(1, 2, or 3)
  - 序列号(unique within CA) identifying certificate
  - 签名算法标识
  - 发行商名(CA)
  - 有效期(from - to dates)
  - 证书主体名(name of owner)
  - 证书主体的公钥信息(algorithm, parameters, key)
  - 发行商唯一标识(v2+)
  - 证书主体唯一标识(v2+)
  - 扩展(v3)
  - 签名(of hash of all fields in certificate)
- Notation CA<<A>>=CA{V, SN, AI, CA, T_A, A, A_p} denotes certificate for A signed by CA

## X.509 Certificates

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_311_0.png)

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_311_1.png)

(a) X.509 Certificate

(b) Certificate Revocation List

## 证书的层次结构

- 如果用户有共同的CA，则他们应该知道彼此的公钥，否则CA必须形成层次结构并互相发放证书
- 每个CA目录入口包含两种证书
  - 向前证书：由其他CA发给X的证书（forward, client）
  - 向后证书：X发给其他CA的证书（backward, parent）
- 每一个用户信任他的父节点的证书
- 通过层次化的结构，可以使一个CA下的用户验证任何其他CA下的用户的公钥证书

## CA层次结构的使用

- A从目录获得证书以建立通往B的证书路径
  - X<<W>>W<<V>>V<<Y>>Y<<Z>>Z<<B>>
- B通过如下路径获得A的公开密钥
  - Z<<Y>>Y<<V>>V<<W>>W<<X>>X<<A>>

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_314_0.png)

## 证书的撤销

- 每个证书都有一个有效期，到期失效
- 可能因为如下原因提前撤销证书
  1. 用户的私钥被认为不安全了
  2. 用户不再信任该CA
  3. CA的证书被认为不安全了
- 每个CA维护一张证书撤销列表
  - the Certificate Revocation List (CRL)
- 用户应该经常去CRL看看某个证书是否还有效

## 14.2.2 认证的过程

- X.509有三种可选的认证过程
  - One-Way Authentication
  - Two-Way Authentication
  - Three-Way Authentication
- 三种方法均使用公钥签名

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_317_0.png)

(a) One-way authentication

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_317_1.png)

(b) Two-way authentication

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_317_2.png)

(c) Three-way authentication

## One-Way Authentication

- 使用一条消息( A->B)建立认证过程
  - A的标识和A创建的消息
  - B所需要的消息
  - 消息的完整性和原创性(不能多次发送)
- 消息必须包含时间戳， nonce， B的标识符， 并由A签名

## Two-Way Authentication

- 使用两条消息建立认证过程(A->B, B->A)
  - A的标识和A创建的消息
  - B所需要的消息
  - 消息的完整性和原创性(不能多次发送)
  - B的标识和B生成的应答消息
  - A需要的消息
  - 应答的完整性和真实性
- 应答消息包括从A得到的临时交互号nonce，时间戳和B生成的临时交互号
- 消息中可以包含签名的其他信息和用A的公钥加过密的会话密钥

