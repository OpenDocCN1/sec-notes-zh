
## PGP——消息的发送与接收

(a) Generic Transmission Diagram (from A)

(b) Generic Reception Diagram (to B)

Figure 15.2 Transmission and Reception of PGP Messages

![](img/0935d481fd26d10140d7cd560b6540ba_484_0.png)

## PGP——会话密钥和密钥环

- PGP使用三种类型密钥
  - 会话密钥：加密消息使用的一次性对称密钥
  - 公钥/私钥：用于加密会话密钥和签名
  - 口令（passphrase）：用于保护私钥

- 密钥需求
  - 会话密钥
    - 充分的随机性
  - 用户拥有多个公钥/私钥对
    - 发送方可能需要使用不同的密钥对与不同的用户通信
    - 接收方可能需要旧的密钥对来处理某些消息
      - 如：使用旧密钥加密的信息
    - PGP中使用两个文件管理密钥对
      - 一个文件管理自己的公钥/私钥对
      - 另一个文件管理其它用户的公钥

## PGP——会话密钥

- 每个消息需要一个会话密钥
  - 对称加密算法不同，密钥长度不同
    - 128-bit CAST和IDEA, 168-bit 3DES
- 伪随机数生成（CAST-128）
  - 使用密码反馈机制
  - 基于ANSI X12.17
  - 用途
    - 会话密钥
- 真随机数生成
  - 键盘击键产生
    - 击键时间和击键内容
  - 用途
    - 生成RSA密钥对
    - 为伪随机数发生器提供种子和其它输入

## PGP——会话密钥

![](img/0935d481fd26d10140d7cd560b6540ba_487_0.png)

Figure 15.12 PGP Session Key and IV Generation (steps G2 through G8)

## PGP——密钥标识

- 问题
  - 用户如果需要使用多个公钥/私钥对
    - 如何知道用哪个私钥解密会话密钥？
    - 如何知道用哪个公钥验证消息签名？
- 解决方案
  - 1. 随消息发送完整的公钥，但效率低
  - 2. 每个公钥与唯一的密钥标识对应
- 密钥标识
  - 定义
    - 密钥标识=KUa mod 2^64
  - 密钥标识重复的可能性非常小
    - 重复的可能性=1/ 2^64

## PGP——消息的一般格式（从A到B）

说明

KeyID=密钥标识

Timestamp=时间戳

Content

Operation

Session key component

Signature

Message

Key ID of recipient's public key (KU_B)

Session key (K_s)

Timestamp

Key ID of sender's public key (KU_A)

Leading two octets of message digest

Message Digest

Filename

Timestamp

Data

E_KU_B

E_KR_A

ZIP

E_K_s

R64

Notation:
E_KU_B = encryption with user B's public key
E_KR_A = encryption with user A's private key
E_K_s = encryption with session key
ZIP = Zip compression function
R64 = Radix-64 conversion function

Figure 15.3 General Format of PGP Message (from A to B)

## PGP——密钥环

- 每个PGP用户维护一对密钥环（keyrings）
  - 公钥环
    - 包含所有已知的其它PGP用户的公钥
  - 私钥环
    - 包含自己的所有公钥/私钥对

## PGP——私钥环

### Private Key Ring

| Timestamp | Key ID* | Public Key | Encrypted Private Key | User ID* |
| --- | --- | --- | --- | --- |
| • • • | • • • | • • • | • • • | • • • |
| Tᵢ | KUᵢ mod 2⁶⁴ | KUᵢ | E_H(Pᵢ)[KRᵢ] | User i |
| • • • | • • • | • • • | • • • | • • • |

* = field used to index table

- 时间戳（Timestamp）
  - 密钥对生成日期/时间
- 密钥标识（KeyID）：公钥标识
- 公钥（Public Key）：公钥部分
- 被加密的私钥（Encrypted Private Key）
  - 使用口令作为密钥进行加密
- 用户标识（UserID）
  - 一般使用用户的电子邮件地址

## PGP——公钥环

Public Key Ring

| Timestamp | Key ID* | Public Key | Owner Trust | User ID* | Key Legitimacy | Signature(s) | Signature Trust(s) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| • • • | • • • | • • • | • • • | • • • | • • • | • • • | • • • |
| T_i | KU_i mod 2^64 | Ku_i | trust_flag_i | User i | trust_flag_i |  |  |
| • • • | • • • | • • • | • • • | • • • | • • • | • • • | • • • |

* = field used to index table

- 所有者信任度（Owner Trust）
  - 用户定义的那该公钥的信任值
- 签名（Signature）
  - 由其它公钥对本公钥的签名
- 签名信任度（Signature Trust）
  - 表示对该签名信任程度
- 密钥合法性（Key Legitimacy）
  - 用于度量对该公钥的信任程度，可根据签名信任度的值计算得到

## PGP——消息生成

Figure 15.5 PGP Message Generation (from User A to User B; no compression or radix 64 conversion)

![](img/0935d481fd26d10140d7cd560b6540ba_493_0.png)

## PGP——消息接收

Figure 15.6 PGP Message Reception (from User A to User B; no compression or radix 64 conversion)

## PGP——公钥管理

- PGP不依赖于证书权威CA
- 每个PGP用户都是自己的CA
  - 能够为直接相识的用户签名
- 从而形成了信任网（web of trust）
  - 信任的密钥被签名
  - 能够信任某个其它用户签名的密钥，如果存在一个签名链到达这些用户

## PGP——信任关系

### Table 15.2 Contents of Trust Flag Byte

| (a) Trust Assigned to Public-Key Owner (appears after key packet; user defined) | (b) Trust Assigned to Public Key/User ID Pair (appears after User ID packet; computed by PGP) | (c) Trust Assigned to Signature (appears after signature packet; cached copy of OWNER TRUST for this signator) |
|---|---|---|
| **OWNERTRUST Field** — undefined trust — unknown user — usually not trusted to sign other keys — usually trusted to sign other keys — always trusted to sign other keys — this key is present in secret key ring (ultimate trust) **BUCKSTOP bit** — set if this key appears in secret key ring | **KEYLEGIT Field** — unknown or undefined trust — key ownership not trusted — marginal trust in key ownership — complete trust in key ownership **WARNONLY bit** — set if user wants only to be warned when key that is not fully validated is used for encryption | **SIGTRUST Field** — undefined trust — unknown user — usually not trusted to sign other keys — usually trusted to sign other keys — always trusted to sign other keys — this key is present in secret key ring (ultimate trust) **CONTIG bit** — set if signature leads up a contiguous trusted certification path back to the ultimately trusted key ring owner |
