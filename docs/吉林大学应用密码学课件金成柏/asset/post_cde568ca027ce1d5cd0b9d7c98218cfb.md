
## 内容提要

- 公钥密码学
  - 公钥密码体制
  - 公钥密码应用
  - 公钥密码要求
  - 公钥密码分析
  - RSA算法
    - 算法描述
    - 计算方面的问题
    - RSA的安全性

## 公钥密码学

- 公钥密码学的发展是4000年密码学历史中最伟大的一次革命
- 公钥密码学与传统密码学区别
  - 使用两个密钥——公钥和私钥，而不是单密钥
  - 基于数论概念，而不是代换和置换
- 在1976年，Whitfield Diffie和Martin Hellman 在Stanford大学首次公开提出了公钥密码学概念

## 公钥密码学——公钥密码体制

- 公钥密码体制（双密钥密码/非对称密码）
  - 组成部分
    - 明文、加密算法、公钥、私钥、密文、解密算法
  - 每个用户都产生一对密钥：公钥、私钥
    - 公钥：对外公开的密钥，用于
      - 消息加密
      - 签名验证
    - 私钥：只被拥有者访问，用于
      - 解密消息
      - 创建签名
  - 每个用户都可以访问到其他用户的公钥
    - 这些公钥构成一个公钥环

## 公钥密码体制——加密

![](img/0935d481fd26d10140d7cd560b6540ba_287_0.png)

## 公钥密码体制——认证

![](img/0935d481fd26d10140d7cd560b6540ba_288_0.png)

(b) Authentication

## 公钥密码体制——加密和认证

![](img/0935d481fd26d10140d7cd560b6540ba_289_0.png)

Figure 9.4 Public-Key Cryptosystem: Secrecy and Authentication

## 公钥密码体制——算法表示

- 记法
  - 明文X、密文Y
  - A和B的公钥/私钥分别记为KUa/KRa和KUb/KRb
- 加密/解密
  - A用B的公钥加密生成密文 Y = E_KUb(X)
  - B用B的私钥解密还原明文 X = D_KRb(Y)
- 认证（签名）
  - A用A的私钥加密签名 Y = E_KRa(X)
  - B用A的公钥解密认证 X = D_KUa(Y)
- 加密和认证
  - A用A的私钥签名，然后用B的公钥加密生成密文
    - Y = E_KUb(E_KRa(X))
  - B用B的私钥解密，然后用A的公钥解密认证
    - X = D_KUa(D_KRb(Y))

## 公钥密码学——公钥密码应用

- 公钥密码应用分为三类
  - 加密/解密
    - 提供保密性
  - 数字签名
    - 提供认证
  - 密钥交换
    - 会话密钥的交换

不同算法，应用范围不同

**Table 9.2 Applications for Public-Key Cryptosystems**

| Algorithm | Encryption/Decryption | Digital Signature | Key Exchange |
|----------|------------------------|-------------------|--------------|
| RSA | Yes | Yes | Yes |
| Elliptic Curve | Yes | Yes | Yes |
| Diffie-Hellman | No | No | Yes |
| DSS | No | Yes | No |

## 公钥密码学——公钥密码要求

- 公钥密码要求
  1. B产生一对密钥（公钥KUb,私钥KRb）在计算上是容易的
  2. 已知接收方B的公钥和要加密的消息M，发送方A产生相应的密文在计算上是容易的 C = E_KUb(M)
  3. 接收方B使用其私钥对接收的密文解密在计算上是容易的 M = D_KRb(C)
  4. 已知公钥KUb，攻击者要确定私钥KRb在计算上是不可行的
  5. 已知公钥KUb和密文C，攻击者要恢复明文M在计算上是不可行的
  6. （可选）加密和解密函数的顺序是可交换的
- 到目前为止，满足上述所有条件并普遍接受的算法只有RSA和ECC

## 公钥密码学——公钥密码要求

单向函数是满足下列性质的函数

- Y = f(X) 容易
- X = f¹(Y) 不可行

单向陷门函数是满足下列性质的函数

- 若k已知, 计算 Y = f(k,X) 容易
- 若k已知, 计算 X = f¹(k,Y) 容易
- 若k未知, 计算 X = f¹(k,Y) 不可行

公钥密码体制的关键就是寻找合适的单向陷门函数

## RSA算法

- 在1977年，Rivest, Shamir和Adleman在MIT 提出，已被广泛接受并实现。
- RSA是分组密码
  - 其明文和密文均为0至n-1之间的整数，通常n的大小为1024位二进制数

## RSA算法——密钥生成

- 密钥生成
  - 随机选择两个大素数 p，q
  - 计算模 n = p·q， ø(n) = (p-1)(q-1)
  - 随机选择加密密钥（公钥） e
    - 满足 1<e<ø(n)， gcd(e, ø(n))=1
  - 解下列等式，找解密密钥（私钥） d
    - e·d=1 mod ø(n) and 0≤d≤n
  - 公开发行公钥 KU={e,n}
  - 安全保存私钥 KR={d,n}

## RSA算法——密钥生成

## Key Generation

- Select p, q
  p and q both prime, p ≠ q
- Calculate n = p × q
- Calculate φ(n) = (p − 1)(q − 1)
- Select integer e
  gcd(φ(n), e) = 1; 1 < e < φ(n)
- Calculate d
  d ≡ e⁻¹ mod φ(n)
- Public key
  KU = {e, n}
- Private key
  KR = {d, n}

## RSA算法——加密/解密

- 加密
  - C=M^e mod n，其中M是消息且满足 0≤M<n
- 解密
  - M=C^d mod n，其中C是密文

Encryption
Plaintext: M < n
Ciphertext: C = M^e (mod n)

Decryption
Ciphertext: C
Plaintext: M = C^d (mod n)

## RSA算法——证明

- 根据Euler定理
  - a^φ(n) mod n = 1，其中a和n互素，即gcd(a,n)=1
- 在RSA算法中
  - n=p·q
  - φ(n)=(p-1)(q-1)
  - e和d 模φ(n)互逆，因此，存在k使得 e·d=1+k·φ(n)
- 证明
  - C^d = (M^e)^d = M^(1+k·φ(n)) = M^1·(M^φ(n))^k = M^1·(1)^k = M^1 = M mod n

## RSA算法——例子

## 密钥生成

1. 选择素数 p=17 和 q=11
2. 计算n=pq=17×11=187
3. 计算φ(n)=(p-1)(q-1)=16×10=160
4. 选择e=7，满足gcd(e,160)=1
5. 确定d=23，满足de=1 mod 160且d < 160
6. 公开发行公钥 KU={7,187}
7. 安全保存私钥 KR={23,187}

## 加密/解密

假设消息M={88}

加密：C = 88⁷ mod 187 = 11

解密：M = 11²³ mod 187 = 88

## RSA算法——幂运算

- 反复平方和乘法
  - 快速、高效的方法
- 原理
  - 计算 a^b，b 可表示如下
    b = Σ_{b_i≠0} 2^i
    因此
    a^b = a^{(Σ_{b_i≠0} 2^i)} = ∏_{b_i≠0} a^{(2^i)}
    a^b mod n = [∏_{b_i≠0} a^{(2^i)}] mod n = (∏_{b_i≠0} [a^{(2^i)} mod n]) mod n
  - 例如：3^129 = 3^128 · 3^1 = 3^2*2*2*2*2*2*2 · 3^1 = 5 · 3 = 4 mod 11
  - 共进行 7+1=8 次乘法（否则，需要进行 128 次乘法）
