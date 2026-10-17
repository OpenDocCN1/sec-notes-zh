

## 常用Hash算法

- SHA-1
- MD5
- RIPEMD-160
- HMAC

## MD5

- MD5即Message-Digest Algorithm 5，用于确保信息传输完整一致。是计算机广泛使用的散列算法之一（又译摘要算法、哈希算法），主流编程语言普遍已有MD5实现。

- 将数据（如汉字）运算为另一固定长度值，是散列算法的基础原理，MD5的前身有MD2、MD3和MD4。

- MD5一度被广泛应用于安全领域。但是由于MD5的弱点被不断发现以及计算机能力不断的提升，现在已经可以构造两个具有相同MD5的信息，使本算法不再适合当前的安全环境。目前，MD5计算广泛应用于错误检查。例如在一些BitTorrent下载中，软件通过计算MD5和检验下载到的碎片的完整性。

- MD5还广泛用于加密和解密技术上。

例如在UNIX系统中用户的密码就是以MD5（或其它类似算法）经加密后存储在文件系统中。

## MD5算法描述 - step 1

在MD5算法中，首先需要对信息进行填充，使其字节长度对512求余的结果等于448。因此，信息的字节长度（Bits Length）将被扩展至N*512+448，即N*64+56个字节（Bytes），N为一个正整数。填充的方法如下，在信息的后面填充一个1和无数个0，直到满足上面的条件时才停止用0对信息的填充。然后，在这个结果后面附加一个以64位二进制表示的填充前信息长度。经过这两步的处理，现在的信息字节长度=N*512+448+64=(N+1)*512，即长度恰好是512的整数倍。这样做的原因是为满足后面处理中对信息长度的要求。

## Padding of Message Digest

Any length message  
1-512 bits Padding  
64 bits length  

Total length : multiple of 512 bits

## MD5描述一step 2

- 初始化MD缓存

- MD为128bit，用于存放散列函数的中间及最终结果

- MD可表示为4个32bit的寄存器(A,B,C,D)，初始化如下

A = 01 23 45 67 (0x67452301)  
B = 89 AB CD EF (0xEFCDAB89)  
C = FE DC BA 98 (0x98BADCFE)  
D = 76 54 32 10 (0x10325476)

## One way hash of MD5

H<sub>i-1</sub> (128 bits)

M<sub>i</sub> (512 bits)

A  
B  
C  
D  
Round 1  
P×16  

A  
B  
C  
D  
Round 2  
G×16  

A  
B  
C  
D  
Round 3  
H×16  

A  
B  
C  
D  
Round 4  
I×16  

H<sub>i</sub> (128 bits)

## Elementary MD5 Operation

初值  
A=01234567  
B=89ABCDEF  
C=FEDCBA98  
D=76543210  

F(B, C, D)=(B∧C)∨(¬B∧D)  
G(B, C, D)=(B∧C)∨(B∧¬D)  
H(B, C, D)=B⊕C⊕D  
I(B, C, D)=C⊕(B∨¬D)  
分别使用於FF, GG, III及II  

512 bits \( M_i \) 分成16個32 bits \( M_{ik} \)  
(k有指定之順序)  

\( T_i \) 為64個常數值(\( 2^{32} \times |sin i| \))  

S 指定左旋之位元數  

## MD5的安全性

- Berson表明，对单循环MD5，使用不同的密码分析可能在合理的时间内找出能够产生相同摘要的两个报文，这个结果被证明对四个循环中的任意一个循环也成立，但作者没有能够提出如何攻击包含全部4个循环MD5的攻击  
- Boer和Bosselaers显示了即使缓存ABCD不同，MD5对单个512bit分组的执行将得到相同的输出(伪冲突)  
- Dobbertin的攻击技术：使MD5的压缩函数产生冲突，即寻找$x、y，x\ne y，但H(x)=H(y)$  
- MD5被认为是易受攻击的，逐渐被SHA-1和RIPEMD-160替代  

## One way hash of SHA-1

H<sub>i-1</sub>(160 bits)

M<sub>i</sub> (512 bits)

Round 0

Round 79

H<sub>i</sub>(160 bits)

## Elementary SHA-1 Operation

$f_t = (B \wedge C) \vee (\neg B \wedge D) \quad 0 \le t \le 19$  
$= B \oplus C \oplus D \quad 20 \le t \le 39$  
$= (B \wedge C) \vee (B \wedge D) \vee (C \wedge D) \quad 40 \le t \le 59$  
$= B \oplus C \oplus D \quad 60 \le t \le 79$  

$W_t = M_t, \quad 0 \le t \le 15$  
$(W_{t-3} \oplus W_{t-8} \oplus W_{t-14} \oplus W_{t-16}) <<< 1, \quad 16 \le t \le 79$  

$K_t = 5A827999 \quad 0 \le t \le 19$  
$= 6ED9EBA1 \quad 20 \le t \le 39$  
$= 8F1BBCDC \quad 40 \le t \le 59$  
$= CA62C1D6 \quad 60 \le t \le 79$  

## Comparison of MD5 & SHA-1

| 差異處 | MD5 | SHA-1 |
|---|---|---|
| 摘要長度 | 128 bits | 160 bits |
| 運算步驟 | 64 | 80 |
| 基本邏輯函數個數 | 4 | 4 |
| 常數個數 | 64 | 4 |

## 数字签名

1. Introduction  
2. ElGamal 数字签名方案  
3. Schnorr数字签名方案  
4. DSS（DSA）

## 消息认证码 （MAC）

MAC=C<sub>k</sub>(M); k为A和B共享的密钥， M为明文  
A → B:M || C<sub>k</sub>(M)

- 提供鉴别  
  - 仅A和B共享密钥k  

## 消息认证码的局限性

- 用于保护通信双方免受第三方攻击  
- 无法防止通信双方的相互攻击  
  - 信宿方伪造报文（转账额度增加）  
  - 信源方否认已发送的报文（股票交易争端）  
- 引入数字签名，是笔迹签名的模拟  

## 数字签名的性质

### 传统签名的基本特点

- 能与被签的文件在物理上不可分割  
- 签名者不能否认自己的签名  
- 签名不能被伪造  
- 容易被验证  

### 数字签名是传统签名的数字化

- 签名能与所签文件（消息）“绑定”  
- 签名使用发送方独有的信息，以防止伪造和否认  
- 产生数字签名比较容易  
- 容易被识别及验证  
- 签名的伪造在计算上是不可行的，无论是从给定的数字签名伪造消息，还是从给定的消息伪造数字签名。  
- 可以保存数字签名的副本  

## 数字签名的功能特征

- 必须能够验证作者及其签名的日期时间  
- 必须能够认证签名时刻的消息内容  
- 签名必须能够由第三方验证，以解决争议  

## 数字签名分类

### 签名方式

- 直接数字签名direct digital signature  
- 仲裁数字签名arbitrated digital signature  

### 安全性

- 无条件安全的数字签名  
- 计算上安全的数字签名  

### 可签名次数

- 一次性的数字签名  
- 多次性的数字签名  

