
## SSL记录协议

- 提供了2种服务
  - 保密性
    - 使用共享密钥加密SSL载荷
    - 共享密钥在握手协议中定义
  - 消息完整性
    - 使用MAC
    - 密钥在握手协议中定义

## SSL记录协议——操作

![](img/0935d481fd26d10140d7cd560b6540ba_569_0.png)

Figure 17.3 SSL Record Protocol Operation

## SSL记录协议——操作

- 分段
  - 上层消息被分成若干个小于或等于2¹⁴字节的段
- 压缩
  - 无损压缩方法，且增加长度不能超过1024字节

## SSL记录协议——操作

- 增加MAC
  - MAC计算

```
hash (MAC_write_secret || pad_2 ||
hash (MAC_write_secret || pad_1 || seq_num ||
SSLCompressed.type || SSLCompressed.length ||
SSLCompressed.fragment))
```

其中：

| MAC_write_secret | = 共享密钥 |
| Hash | = MD5或SHA-1 |
| Pad_1 | = 填充字节0x36, MD5重复48次（384位）, SHA-1重复40次（320位） |
| Pad_2 | = 填充字节0x5C, MD5重复48次, SHA-1重复40次 |
| seq_num | = 序列号 |
| SSLCompressed.type | = 处理此段的上层协议类型 |
| SSLCompressed.length | = 压缩后的分段长度 |
| SSLCompressed.fragment | = 压缩后的分段（没有压缩，则为明文） |

## SSL记录协议——操作

- 加密
  - 加密对内容的增加不能超过1024字节
  - 对分组加密，填充应在MAC后、加密前进行

- 可使用的加密算法

| 分组密码 |  | 流密码 |  |
|---|---|---|---|
| 算法 | 密钥大小 | 算法 | 密钥大小 |
| IDEA | 128 | RC4-40 | 40 |
| RC2-40 | 40 | RC4-128 | 128 |
| DES-40 | 40 |  |  |
| DES | 56 |  |  |
| 3DES | 168 |  |  |
| Fortezza | 80 |  |  |

## SSL记录协议——操作

- 添加SSL记录头
  - 内容类型（8位）
    - 封装段使用的高层协议
  - 主版本（8位）
    - SSLv3的值为3
  - 从版本（8位）
    - SSLv3的值为0
  - 压缩长度（16位）
    - 明文段（如果压缩，则为压缩段）的字节长度，最大值为2^14+2048

![](img/0935d481fd26d10140d7cd560b6540ba_573_0.png)

## SSL高层协议

- 修改密码规范协议
  - 使得挂起状态被拷贝到当前状态中，用于更新此连接使用的密码组
  - 仅一个字节，值为1
- 警报协议：向对等实体传送SSL相关警报信息
- 握手协议：相互认证、协商加密和MAC算法、协商密钥

(a) Change Cipher Spec Protocol

(c) Handshake Protocol

(b) Alert Protocol

(d) Other Upper-Layer Protocol (e.g., HTTP)

Figure 17.5 SSL Record Protocol Payload

## SSL警报协议

- 向对等实体传送SSL相关警报信息
  - 由两个字节组成
    - 第1个字节表示警报的严重程度
      - warning: 值为1
      - fatal: 值为2
    - 第2个字节表示警报的信息代码
      - unexpected message, bad record mac, decompression failure, handshake failure, illegal parameter
      - close notify, no certificate, bad certificate, unsupported certificate, certificate revoked, certificate expired, certificate unknown
  - 象所有SSL数据，需要压缩和加密

## SSL握手协议

- 相互认证、协商加密和MAC算法、协商密钥
- 消息由3个域组成
  - 类型（1字节）：表明10种消息中一种
  - 长度（3字节）：消息字节长度
  - 内容（>=0字节）：与消息参数有关

### Table 17.2 SSL Handshake Protocol Message Types

| Message Type | Parameters |
|---|---|
| hello_request | null |
| client_hello | version, random, session id, cipher suite, compression method |
| server_hello | version, random, session id, cipher suite, compression method |
| certificate | chain of X.509v3 certificates |
| server_key_exchange | parameters, signature |
| certificate_request | type, authorities |
| server_done | null |
| certificate_verify | signature |
| client_key_exchange | parameters, signature |
| finished | hash value |

## SSL握手协议

协议分四个阶段

- 建立安全能力
- 服务器认证和密钥交换
- 客户端认证和密钥交换
- 完成

![](img/0935d481fd26d10140d7cd560b6540ba_577_0.png)

Figure 17.6 Handshake Protocol Action

## SSL握手协议——阶段1

阶段1: 建立安全能力

- 版本
  - 客户端所支持的最高SSL版本
- 随机数
  - 由客户端生成的随机数结构，由32位时间戳和一个安全随机数生成器生成的28字节随机数
- 会话标识: 变长的会话标识
  - 非0值意味着客户端想更新已存在连接的参数，或为此会话创建一个新的连接
  - 0值意味着客户端想在新的会话上创建一个新连接
- 密码组: 客户端支持的密码算法列表
- 压缩方法: 客户端支持的压缩算法列表

- 版本
  - 客户端支持的最低版本和服务器端支持的最高版本
- 会话标识
  - 如果客户端会话标识非0，则服务器使用与之相同的值
  - 否则，服务器会话标识包含新会话的值

## SSL握手协议——阶段1

- 密码组参数
  - 第1个元素是密码交换方法
    - RSA: 公钥证书
    - 固定Diffie-Hellman: Diffie-Hellman公钥参数证书
    - 瞬时Diffie-Hellman: 使用RSA或DSS私钥签名
    - 匿名Diffie-Hellman: 没有认证
    - Fortezza
  - 其它元素
    - 密码算法
    - MAC算法
    - 密码类型: 流或分组
    - 可出口: 真或假
    - 散列长度: 0, 16（MD5）字节, 20（SHA-1）字节
    - 密钥材料: 字节序列, 包含生成写密钥所使用的数据
    - IV大小: CBC加密使用的初始矢量

## SSL握手协议——阶段2

### 阶段2：服务器认证和密钥交换

- certificate消息（可选）
  - X.509 v3证书链（除了匿名DH外）
- server_key_exchange消息（可选）
  - 发送服务器密钥交换信息
  - 不需要此消息
    - 固定DH
    - RSA密钥交换
  - 需要此消息
    - 匿名DH
    - 瞬时DH
    - RSA密钥交换，服务器仅使用RSA签名功能
- certificate_request消息（可选）
  - 非匿名服务器需要向客户端申请证书
  - 参数：证书类型和认证中心
- server_done消息
