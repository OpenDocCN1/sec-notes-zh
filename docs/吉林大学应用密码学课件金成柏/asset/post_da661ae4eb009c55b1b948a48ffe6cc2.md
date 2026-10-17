
# hash函数——分组链接技术

- 分组链接技术
  - 假设将消息M分组为M1 , M2 , ..., MN
  - 计算hash码G
    - H₀=0
    - Hᵢ = Eₘᵢ [Hᵢ₋₁]
    - G = Hₙ
  - 类似于不使用密钥的CBC

- 产生的hash码长度太小 (64-bit)
  - 易受到生日攻击
  - 中间相遇攻击

# 分组链接技术——中间相遇攻击

- 已知
  - 消息hash码G
- 构造
  - 消息Q，使其hash码为G
- 过程
  1. 构造任何形为Q1, Q2, ..., QN-2
  2. 对1<=i<=(N-2)，计算Hi=E Qi [Hi-1]
  3. 产生2^m/2随机分组，对每一分组x计算Ex[H N-2]
  4. 产生2^m/2随机分组，对每一分组y计算Dy[G]
  5. 在上述两个随机分组中，寻找计算值相同的一对分组x和Y
  6. 构造消息Q1, Q2, ..., QN-2, X, Y，该消息的hash码也为G

# Hash函数和MAC的安全性

- 穷举攻击
  > 要求抗强碰撞能力的hash其计算代价为2^m/2
    - MD5使用128位hash码，安全强度不足
    - SHA-1和RIPEMD-160的hash码都是160位，安全强度相对较高
  > MAC穷举攻击的代价为min(2^k,2^n),其中，k为密钥位数，n为MAC位数
    - 可以攻击密钥空间，也可以攻击消息或MAC值
    - 一般认为min(k,n)>128是安全的

# Hash函数和MAC的安全性

- 密码分析
  - hash函数的密码分析攻击取得了一定的成果
  - MAC函数的密码分析攻击研究较少
- 安全的hash码一般结构
  - MD5、SHA-1、RIPEMD-160等大多数hash函数都具有这种结构
  - 假设
    - IV=初始n位向量，CV=链接变量，Yi=第i个输入分组
    - f=压缩算法，L=输入分组数，n=hash码长度，b=输入分组长度
  - hash函数归纳为
    - CV0=IV
    - CVi=f(CVi-1,Yi-1)
    - H(M)=CVL
  - 压缩函数f的抗碰撞能力决定hash函数的抗碰撞能力

# 安全的hash算法一般结构

![](img/0935d481fd26d10140d7cd560b6540ba_369_0.png)

- IV = Initial value
- CV = chaining variable
- Yi = ith input block
- f = compression algorithm
- L = number of input blocks
- n = length of hash code
- b = length of input block

Figure 11.10 General Structure of Secure Hash Code

# 【密码编码学与网络安全】

（第三版）[美]William Stallings

# 第12章 hash算法

吉林大学

金成柏 jincb@sohu.com

2008年4月20日

# 内容提要

- MD5——消息摘要算法
- SHA-1——安全hash算法
- RIPEMD-160
- HMAC

# hash算法

- hash算法与对称分组密码有类似发展过程
  - 随着穷举攻击能力和密码分析能力的不断增强，人们设计出具有更长hash码且能抗密码分析的新hash算法。
    - 分组密码：从DES到AES
    - hash算法：从MD4 & MD5到SHA-1 & RIPEMD-160
  - 遵循已证明的结构
    - 分组密码：Feistel
    - hash算法：安全的hash算法一般结构

# 安全的hash算法一般结构

![](img/0935d481fd26d10140d7cd560b6540ba_373_0.png)

- IV = Initial value
- CV = chaining variable
- Yi = ith input block
- f = compression algorithm
- L = number of input blocks
- n = length of hash code
- b = length of input block

Figure 11.10 General Structure of Secure Hash Code

# MD5

- MIT的Ronald Rivest (RSA中的R) 提出
- MD2， MD4系列中最新版本
- 产生128-bit的hash值
- 作为Internet standard RFC1321
- 到目前为止仍为广泛使用的hash算法
- 安全强度受到威胁
  - 生日攻击代价为2^64
  - 抗密码分析攻击能力较弱

# MD5——算法步骤

Figure 12.1 Message Digest Generation Using MD5

# MD5——算法步骤

1. 生成512位消息分组

- 增加填充位
  - 填充消息使其长度与448模512同余
  - 消息长度恰好满足上述条件时，仍需要填充

- 填充长度
  - 用64位表示填充前消息的长度
  - 若消息长度大于2^64，则取模2^64的余
  - 将64位的消息长度附加到增加填充位后的消息后面

- 表示
  - Y_0,Y_1,...,Y_{L-1}表示填充后的消息，其中L是分组长度，Y_i为512位的分组
  - M[0..N-1]数组表示填充后的消息，其中N表示整个消息的32位字的个数，数组元素为32位。N=L×16

# MD5——算法步骤

2. 初始化MD缓冲区

> 缓冲区用4个32位的寄存器（A,B,C,D）表示
  - 主要用于保存hash函数的中间结果
> 初始化4个寄存器
  - A=67452301
  - B=EFCDAB89
  - C=98BADCFE
  - D=10325476
> 以低端格式存储：字的最低有效字节存储在低地址字节位置
> 表示
  - 缓冲区初值也表示为IV

# MD5——算法步骤

- 3. 处理每个512位消息分组
  ➢ 第q个分组处理表示为： CVq+1 = HMD5 (CVq , Yq)
  • 其中：
    • HMD5是压缩函数
    • Yq是第q个分组
    • CVq、CVq+1是压缩函数HMD5的输入、输出，CV0=IV
    • CVq也称之为链接变量
  • 每次处理前，将CVq输入到寄存器ABCD中，处理后将寄存器ABCD中内容输出到CVq+1

4. 第L个分组的压缩函数HMD5处理的输出CVL 即是128位的消息摘要

# MD5——压缩函数Hₘₔ₅

- 压缩函数由4轮运算组成

- 每轮运算用轮函数RFₓ表示
  - x表示每个轮函数使用的不同的基本逻辑函数，表示为F,G,H和I
  - 每轮有16步迭代

- 压缩函数Hₘₔ₅表示为
  CV₀=IV
  CV_q+1=SUM₃₂[CV_q,
  RF_I(Y_q,
  RF_H(Y_q,
  RF_G(Y_q,
  RF_F(Y_q,CV_q)))))]
  MD=CV_L
  其中SUM32=模2³²加法

![](img/0935d481fd26d10140d7cd560b6540ba_379_0.png)

# MD5——轮函数RFx

![](img/0935d481fd26d10140d7cd560b6540ba_380_0.png)

Figure 12.3 Elementary MD5 Operation (single step)

# MD5——轮函数RFx

- 每轮需要16步迭代

- 每步迭代可表示为： ABCD k,s,i
  = A←d,
  B←b+((a+g(b,c,d)+X[k]+T[i])<<<s),
  C←b,
  D←c

其中：

- a,b,c,d=缓冲区对应的4个寄存器中的字

- g =基本逻辑函数F,G,H,I之一

- <<<s =32位的变量循环左移s位，也表示为CLSs

- X[k] =M[q×16+k]=第q个分组的第k个32位字

- T[i] =矩阵T中第i个32位字=2³²×abs(sin(i))的整数部分

- + =模2³²加法
