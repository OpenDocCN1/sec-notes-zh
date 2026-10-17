
## 基于公钥密码的对称密钥分配

- 只要分配了公钥，通信各方就可以进行安全通信
  - 认证和数据加密/解密
  - 由于公钥密码算法运算效率较低，很少使用公钥密码进行消息的加密/解密
- 更经常的方法
  - 公钥密码用于对称密钥（秘密密钥）的分配
  - 对称密钥用于消息的加密/解密

## 简单的秘密密钥分配

![](img/0935d481fd26d10140d7cd560b6540ba_321_0.png)

Figure 10.5  Simple Use of Public-Key Encryption to Establish a Session Key

- **1979年，由Merkle提出**
  1. A生成密钥对 {$KU_a$, $KR_a$}，并将含有$KU_a$和其标识$ID_A$的消息发送给B。
  2. B生成秘密密钥（会话密钥）$K_s$，并用A的公钥对$K_s$加密后发送给A。
  3. A计算 $D_{KR_a}[E_{KU_a}[K_s]]$ 获得秘密密钥$K_s$。

## 简单的秘密密钥分配——安全性分析

中间人攻击（主动攻击）

1. A生成密钥对{KUa, KRa}，并将含有KUa和其标识IDA的消息发送给B
2. E截获该消息，生成其密钥对{KUe, KRe}，并将Kue||IDA发送给B
3. B生成秘密密钥（会话密钥）Ks，并发送EKue[Ks]
4. E截获该消息，并计算DKRe[EKue[Ks]]获得秘密密钥Ks
5. E发送EKUa[Ks]给A
6. A计算DKRa[EKUa[Ks]]获得秘密密钥Ks

这时，E获得了A、B通信的秘密密钥Ks

## 安全的秘密密钥分配

### 秘密密钥分配步骤

1. A用B的公钥KU_b加密A的标识（ID_A）、临时交互号（N₁），然后发送该消息给B
2. B用KU_a对A的临时交互号（N₁）和B所产生的临时交互号（N₂）加密，并发送给A
3. A用B的公钥对N₂加密并发送给B，以使B相信其通信伙伴是A
4. A生成秘密密钥K_S，并将E_KU_b [E_KR_a [K_S]]发送给B
5. B计算D_KU_a [E_KR_b [K_S]]获得密钥K_S

![](img/0935d481fd26d10140d7cd560b6540ba_324_0.png)

Figure 10.6 Public-Key Distribution of Secret Keys

## Diffie-Hellman密钥交换

- 首次提出的公钥机制
  - Diffie & Hellman在1976年提出
  - 该算法仅限于进行密钥交换
  - 许多商业产品都使用这种密钥交换技术
- 公钥分配机制
  - 公钥/私钥不能用于交换消息
  - 而用于建立只有双方知道的共享密钥
- 基于有限域（模素数或多项式）上的幂运算
  - 容易
- 安全性依赖于计算离散对数的困难性
  - 很难

### Diffie-Hellman密钥交换——步骤

1. 全局公开的参数（所有用户都知道）
   - 大素数q
   - 素数q的本原根α
2. 生成公钥/私钥对（每个用户）
   - 随机选择x_A（私钥），满足 x_A < q
   - 计算公钥 y_A = α^x_A mod q
3. 计算 A & B 的共享密钥 K_AB
   - K_AB = α^x_A·x_B mod q
   - = y_A^x_B mod q  (which B can compute)
   - = y_B^x_A mod q  (which A can compute)
   - K_AB 可被用做 A 和 B 之间安全通信的会话密钥

### Diffie-Hellman密钥交换

#### Global Public Elements

q
prime number

α
α < q and α a primitive root of q

#### User A Key Generation

Select private X_A
X_A < q

Calculate public Y_A
Y_A = α^X_A mod q

#### User B Key Generation

Select private X_B
X_B < q

Calculate public Y_B
Y_B = α^X_B mod q

#### Generation of Secret Key by User A

K = (Y_B)^X_A mod q

#### Generation of Secret Key by User B

K = (Y_A)^X_B mod q

Figure 10.7 The Diffie-Hellman Key Exchange Algorithm

#### User A

Generate random X_A < q;
Calculate Y_A = α^X_A mod q

Calculate K = (Y_B)^X_A mod q

#### User B

Generate random X_B < q;
Calculate Y_B = α^X_B mod q;
Calculate K = (Y_A)^X_B mod q

Y_A

Y_B

Figure 10.8 Diffie-Hellman Key Exchange

## Diffie-Hellman密钥交换——例子

- A/B双方已知素数q=353和本原根α=3
- A/B双方随机选择各自私钥
  - x_A=97， x_B=233
- A/B双方计算各自公钥
  - y_A=3^97 mod 353 = 40 (Alice)
  - y_B=3^233 mod 353 = 248 (Bob)
- A/B双方计算共享的会话密钥
  - K_AB = y_B^x_A mod 353 = 248^97 = 160 (Alice)
  - K_AB = y_A^x_B mod 353 = 40^233 = 160 (Bob)

## 椭圆曲线算术

- 目前公钥密码产品和标准主要是RSA
  - 为了提高安全性，密钥长度一直在增加

- 椭圆曲线密码学（ECC）对RSA提出挑战
  - 用更短的密钥长度，获得相同的安全性
  - 比RSA和Diffie-Hellman更难于描述

## 椭圆曲线算术——实数域上的椭圆曲线

- 选择椭圆曲线方程
  > y²=x³+ax+b
  其中a和b是实数，且满足4a³+27b² ≠ 0

- 椭圆曲线上点E(a,b)定义Abel群

- 加法运算定义

  1. O是加法单位元，若P∈E(a,b)，则P+O=P
  2. 若P=(x,y)，则逆元-P=(x,-y)
  3. 若P≠Q，过P和Q作一条直线交椭圆曲线于第三点S，则定义P+Q+S=O，即P+Q=-S
  4. 若P=Q，过P点画一条切线交椭圆曲线于第三点S，则定义P+Q=2P=2Q=-S

![](img/0935d481fd26d10140d7cd560b6540ba_332_0.png)

Figure 10.9 Example of Elliptic Curves

- • 椭圆曲线E(-1,0)和E(1,1)

- 加法运算

- 设

• P=(x_p, y_p) 和Q=(x_Q, y_Q)，则连接这两点的直线1的斜率 Δ=(y_Q-y_P)/(x_Q-x_P)

• R=P+Q

- 若P≠Q，则

```
x_R = Δ² - x_P - x_Q
```

```
y_R = -y_P + Δ(x_P - x_R)
```

- 若P=Q，则

```
x_R = ((3x_p² + a)/(2y_p))² - 2x_p
```

```
y_R = ((3x_p² + a)/(2y_p))(x_P - x_R) - y_P
```

## 椭圆曲线算术——Zp上的椭圆曲线

- 选择椭圆曲线方程
  - $y^2=x^3+ax+b\pmod p$
    - 其中$a$和$b$是$Z_p$中的元素，且满足$4a^3+27b^2\ne0\pmod p$

- 椭圆曲线上点$E_p(a,b)$定义Abel群

- 加法运算

$x_R=(\lambda^2-x_P-x_Q)\pmod p$

$y_R=(\lambda(x_P-x_R)-y_P)\pmod p$

其中

$\lambda=\begin{cases}
\left(\dfrac{y_Q-y_P}{x_Q-x_P}\right)\pmod p, & \text{若 }P\ne Q\\[6pt]
\left(\dfrac{3x_P^2+a}{2y_P}\right)\pmod p, & \text{若 }P=Q
\end{cases}$
