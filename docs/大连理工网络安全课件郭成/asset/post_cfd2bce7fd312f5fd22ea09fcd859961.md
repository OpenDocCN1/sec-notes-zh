

## 序列密码体制和分组密码体制

- 序列密码
  - 如果密文不仅与最初给定的算法和密钥有关，同时也与明文位置有关(是所处位置的函数)，则称为序列密码体制。加密以明文比特为单位，以伪随机序列与明文序列模2加后，作为密文序列。

- 分组密码
  - 如果经过加密所得到的密文仅与给定的密码算法和密钥有关，与被处理的明文数据在整个明文中的位置无关，则称为分组密码体制。通常以大于等于64位的数据块为单位，加密得相同长度的密文。

## 其他加密体制

- 确定型密码体制和概率密码体制
  - 确定型：当明文和密钥确定后，密文也就唯一地确定了。
  - 概率型：当明文和密钥确定后，密文通过客观随机因素从一个密文集合中产生，密文形式不确定，称为概率型密码体制。

- 单向函数型密码体制和双向变换型密码体制
  - 单向函数型密码体制适用于不需要解密的场合，容易将明文加密成密文，如哈希函数；
  - 双向变换型密码体制可以进行可逆的加密、解密变换。

## 现代密码学基本原则

- 现代密码学的基本原则
- 设计加密系统时，总是假定密码算法是可以公开的，需要保密的是密钥。一个密码系统的安全性不在算法的保密，而在于密钥，即Kerckhoff原则。
- 对加密系统的要求
- 系统应该是实际上安全的(practical secure)，截获密文或已知明文—密文对时，要决定密钥或任意明文在计算上是不可行的。
- 加密解密算法适用于密钥空间中的所有元素。
- 系统易于实现，使用方便。
- 系统的安全性不依赖于对加密体制或加密算法的保密，而依赖于密钥。
- 系统的使用不应使通信网络的效率过分降低。

# 传统密码的简化模型

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_341_0.png)

Figure 2.1 Simplified Model of Symmetric Encryption

# 传统密码体制的模型

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_342_0.png)

Figure 2.2 Model of Symmetric Cryptosystem

# 网络信息安全
Chapter 3
Block Cipher and Data Encryption Standard

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_343_0.png)

# 第3章 分组密码和数据加密标准

- 分组密码是一种加密解密算法，将输入明文分组当做一个整体处理，输出一个等长的密文分组。
- 许多分组密码都采用Feistel结构，这样的结构由许多相同的轮函数组成。每一轮里，对输入数据的一半进行代换，接着用一个置换来交换数据的两个部分，扩展初始的密钥使得每一轮使用不同的子密钥。
- DES是应用最为广泛的分组密码，它扩展了经典的Feistel结构。DES的分组和密钥分别是64位和56位的。
- 差分分析和线性分析是两种重要的密码分析方法。DES对这两种攻击有一定的免疫性。

# 乘积密码的设计思想

- Claude Shannon and Substitution-Permutation Ciphers
  - 1949年，Claude Shannon 引进了substitution-permutation (S-P) networks的思想，即现代的乘积加密器，形成了现代分组加密的基础。S-P Networks 是基于替代和置换这两个基本操作的。
- 提供了对明文信息处理所做的 *confusion* 和 *diffusion*。
  - Shannon认为，为了对付基于统计分析的密码破译，必须对明文作confusion(混淆)和diffusion(扩散)处理，以减少密文的统计特性，为统计分析制造障碍。
- **diffusion** —明文统计结构扩散消失到大批密文统计特性中，使明文和密文之间统计关系尽量复杂；
- **confusion** —混淆，使密文和加密密钥之间的关系尽量复杂。

# Figure 3.5 Single Round of DES Algorithm

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_346_0.png)

# 网络信息安全
## Chapter 4
## Finite Fields

![](img/4c4bffc5085ec4e6ce37d2fe335fbad5_347_0.png)

# 4.1群，环和域 Groups, Rings, and Fields

- 群G，记作{G，⋅}，定义一个二元运算⋅的集合，G中每一个序偶(a, b)通过运算生成G中元素(a⋅b)，满足下列公理：
  - (A1) 封闭性Closure：如果a和b都属于G，则a⋅b也属于G。
  - (A2) 结合律Associative：对于G中任意元素a，b，c，都有a⋅(b⋅c)=(a⋅b)⋅c成立
  - (A3) 单位元Identity element：G中存在一个元素e，对于G中任意元素a，都有a⋅e=e⋅a=a成立
  - (A4) 逆元Inverse element：对于G中任意元素a，G中都存在一个元素a'，使得a⋅a'=a'⋅a=e成立

# 交换群和循环群

- 交换群Abelian Group：还满足以下条件的群称为交换群(又称阿贝尔群)
  - (A5) 交换律Commutative：对于G中任意的元素a，b，都有a·b=b·a成立

- 当群中的运算符是加法时，其单位元是0；a的逆元是-a，并且减法用以下的规则定义：
  - a − b = a + (-b)

- 循环群Cyclic Group
  - 如果群中的每一个元素都是一个固定的元素a（a ∈ G）的幂aᵏ(k为整数)，则称群G为循环群。元素a生成了群G，或者说a是群G的生成元。

# 环 (Rings)

- 环 R，由 {R, +, ×} 表示，是具有加法和乘法两个二元运算的元素的集合，对于环中的所有 a, b, c，都服从以下公理：
  - (A1-A5)，单位元是 0，a 的逆是 -a。
  - (M1)，乘法封闭性，如果 a 和 b 属于 R，则 ab 也属于 R。
  - (M2)，乘法结合律，对于 R 中任意 a、b、c，有 a(bc)=(ab)c。
  - (M3)，乘法分配律，a(b+c)=ab+ac or (a+b)c=ac+bc。
  - (M4)，乘法交换律，ab=ba，交换环。
  - (M5)，乘法单位元，R 中存在元素 1 使得所有 a 有 a1=1a。
  - (M6)，无零因子，如果 R 中有 a、b 且 ab=0，则 a=0 or b=0。
- 满足 M4 的是交换环；满足 M5 和 M6 的交换环是整环。

# 域 (Fields)

- 域F，可以记为{F, +, x}，是有加法和乘法的两个二元运算的元素的集合，对于F中的任意元素a, b, c，满足以下公理：
  - (A1-M6)，F是一个整环
  - (M7)，乘法逆元，对于F中的任意元素a(除0以外)，F中都存在一个元素a⁻¹，使得aa⁻¹=(a⁻¹)a=1。
  - 域就是一个集合，在其上进行加减乘除而不脱离该集合，除法按以下规则定义：a/b=a(b⁻¹)。
- 有理数集合，实数集合和复数集合都是域；整数集合不是域，因为除了1和-1有乘法逆元，其他元素都无乘法逆元

# 模算术运算

(a₁ op a₂) mod n =[(a₁ mod n )] op (a₂ mod n)] mod n

- ①反身性： a=a mod n
- ②对称性： 若a=b mod n， 则b=a mod n
- ③传递性：若a=b mod n 且b=c mod n， 则a=c mod n
- ④如果 a=b mod n且 c=d mod n， 则
  - a+c=(b+d) mod n
  - a-c=(b-d) mod n
  - a•c=(b•d) mod n
- ⑤ (a+b) mod n = (a mod n + b mod n) mod n
  - (a-b) mod n = (a mod n - b mod n) mod n
  - (a•b) mod n = (a mod n • b mod n) mod n

# 模算术运算

(a+b) mod n = (a mod n + b mod n) mod n

证明：定义 (a mod n)=ra, (b mod n)=rb
于是存在整数 j, k 使得 a=ra+jn, b=rb+kn.
那么
(a+b) mod n = (ra+jn+rb+kn) mod n
= (ra+rb+(k+j)n) mod n
= (ra+rb) modn
= [(a mod n)+(b mod n)] mod n

