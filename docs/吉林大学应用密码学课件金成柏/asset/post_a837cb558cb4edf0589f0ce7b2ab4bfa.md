
## RC5密码——密钥扩展

![](img/0935d481fd26d10140d7cd560b6540ba_217_0.png)

Figure 6.5 RC5 Key Expansion

+   初始化
+     定义Pw和Qw
+     初始化S算法

| W | 16 | 32 | 64 |
| Pw | B7E1 | B7E15163 | B7E151628AED2A68 |
| Qw | 9E37 | 9E3779B9 | 9E3779B97F4A7C15 |

```
S[0] = Pw;
for i=1 to t-1 do
    S[i]=S[i-1]+Qw
```

+   - 混合
+     i=j=X=Y=0;
+     do 3×max(t,c) times:
+     S[i]=(S[i]+X+Y)<<<3;
+     X=S[i];
+     i=(i+1) mod t;
+     L[j]=(L[j]+X+Y)<<<(X+Y);
+     Y=L[j];
+     j=(j+1) mod c;

## RC5密码——加密/解密

![](img/0935d481fd26d10140d7cd560b6540ba_220_0.png)

Figure 6.6 RC5 Encryption and Decryption

## RC5密码——加密算法

+   - 将输入数据分组分成两个部分A和B
+   
+     `L₀ = A + S[0];`
+     `R₀ = B + S[1];`
+     `for i = 1 to r do`
+       `Lᵢ = ((Lᵢ₋₁ XOR Rᵢ₋₁) <<< Rᵢ₋₁) + S[2 x i];`
+       `Rᵢ = ((Rᵢ₋₁ XOR Lᵢ) <<< Lᵢ) + S[2 x i + 1];`

## RC5密码——运行模式

RFC2040为RC5定义了4种工作模式

+   - RC5分组密码，即ECB模式
+   - RC5-CBC，即CBC模式。要求消息长度为分组的整数倍。
+   - RC5-CBC-PAD，属于CBC模式。通过填充，可以处理任意长度的消息。
+   - RC5-CTS，属于CBC模式，称为密文挪用模式。可以处理任意长度的消息，并产生等长的密文。

![](img/0935d481fd26d10140d7cd560b6540ba_222_0.png)

## 高级对称分组密码特点

+   - 算法参数
+     - 密钥长度可变、分组长度可变、迭代次数可变
+   - 复杂的密钥生成算法
+   - 复合运算：使用多个算术和布尔运算使算法复杂化
+   - 与密钥相关的S盒
+   - 与密钥相关的循环位移
+   - 与数据相关的循环位移
+   - 可变的轮函数F
+   - 每一轮迭代对数据左右两半部分同时运算

## 流密码——典型结构

![](img/0935d481fd26d10140d7cd560b6540ba_224_0.png)

Figure 6.8 Stream Cipher Diagram

+   - 密钥K输入到一个伪随机发生器
+   - 发生器的输出称为密钥流
+   - 通过与同一时刻的明文流进行异或产生密文流

## 流密码——考虑的因素

## 需考虑的主要因素

+    - 密钥流序列的周期要长
+    - 密钥流应该尽可能接近一个真正的随机数流的特征
+    - 密钥应该足够长

## 流密码和分组密码速度对比

Table 6.2 Speed Comparisons of Symmetric Ciphers on a Pentium II

| Cipher | Key Length | Speed (Mbps) |
|---|---|---|
| DES | 56 | 9 |
| 3DES | 168 | 3 |
| RC2 | variable | 0.9 |
| RC4 | variable | 45 |

## RC4流密码

+   - 由Ron Rivest 设计，被RSA拥有，简单但高效
+     - 面向字节流
+     - 密钥长度可变（1到256个字节）
+     - 广泛应用
+       - web SSL/TLS, wireless WEP

## RC4流密码——密钥流生成

+   - 假设
+     - 状态数组S[0...255]，密钥数组K[0...keylen]，临时数组T[0...255]
+     - 数组元素为字节
+   - 初始化S
+     - 置S元素为0到255，即S[0]=0,S[1]=1,...,S[255]=255
+     - 将密钥K循环重复填充T
+     - 进行S的初始置换
+       - j=0
+       - for i=0 to 255 do
+         - j=(j+S[i]+T[i]) mod 256;
+         - swap (S[i],S[i]);
+   - 密钥流生成
+     - i = j = 0
+     - for each message byte Mᵢ
+       - i = (i + 1) (mod 256)
+       - j = (j + S[i]) (mod 256)
+       - swap(S[i], S[j])
+       - t = (S[i] + S[j]) (mod 256)
+       - Cᵢ = Mᵢ XOR S[t]

## RC4流密码——密钥流生成

(a) Initial state of S and T

(b) Initial permutation of S

(c) Stream Generation

Figure 6.9 RC4

![](img/0935d481fd26d10140d7cd560b6540ba_228_0.png)

## RC4流密码——安全性

+   - 能够抵御已知的攻击
+   - 变换是非常非线性的
+   - 由于RC4是流密码，密钥决不能重用
+     - 如果使用相同的密钥对两个明文流加密，则2个密文流的异或，就是两个明文流的异或

## 【密码编码学与网络安全】

（第三版）[美]William Stallings

## 第7章 用对称密码实现保密性

吉林大学

金成柏 jincb@sohu.com

2008年3月30日

## 内容提要

+   - 密码功能的配置
+   - 传输保密性
+   - 密钥分配
+   - 随机数的生成

## 密码功能的配置——安全隐患

![](img/0935d481fd26d10140d7cd560b6540ba_232_0.png)

Figure 7.1 Points of Vulnerability

## 密码功能的配置——安全隐患

+   - 考虑典型的场景
+     - 在局域网上
+       - 局域网通常是广播网络，数据是以帧的形式传输。
+     - 在电话网上
+       - 通过调制解调器与通信服务器相连
+     - 在数字网上（宽带）
+       - DDN、X.25、帧中继或ATM网，通过路由器与局域网相连
+     - 在无线网上
+       - 微波链路、卫星频道，通过路由器与局域网相连
+   - 在上述的通信链路上，攻击者都可能进行窃听和攻击
+     - 局域网上的攻击，技术难度低
+     - 无线网上的攻击，风险小

## 密码功能的配置——链路加密和端对端加密

![](img/0935d481fd26d10140d7cd560b6540ba_234_0.png)

● = end-to-end encryption device

○ = link encryption device

PSN = packet switching node

Packet-switching network

Figure 7.2 Encryption Across a Packet-Switching Network

## 密码功能的配置——链路加密和端对端加密

+   - 保证数据传输的安全性，有两种可选配置
+   - 链路加密（link encryption）
+     - 在每个链路两端都加上加密设备
+     - 优点
+       - 链路传输的保密性得到保证
+     - 缺点
+       - 在链路之间传输，都要重新加解密，通信节点的保密性无法控制
+       - 每条链路都需要一对加密设备和密钥，密钥数量可能很大
+   - 端对端加密（end-to-end encryption）
+     - 在信息传输的两端系统中进行加解密
+     - 优点
+       - 用户数据在传输的全过程中都是保密的
+     - 缺点
+       - 传输的过程存在不安全性。路由信息、密文信息
+       - 每对用户都需要一对密钥，密钥数量可能巨大
