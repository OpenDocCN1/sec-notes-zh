# 密码分析学——穷举攻击的成本

Table 2.2 Average Time Required for Exhaustive Key Search

| Key Size (bits) | Number of Alternative Keys | Time required at 1 encryption/μs | Time required at 10⁶ encryptions/μs |
|---|---|---|---|
| 32 | 2³² = 4.3 × 10⁹ | 2³¹ μs = 35.8 minutes | 2.15 milliseconds |
| 56 | 2⁵⁶ = 7.2 × 10¹⁶ | 2⁵⁵ μs = 1142 years | 10.01 hours |
| 128 | 2¹²⁸ = 3.4 × 10³⁸ | 2¹²⁷ μs = 5.4 × 10²⁴ years | 5.4 × 10¹⁸ years |
| 168 | 2¹⁶⁸ = 3.7 × 10⁵⁰ | 2¹⁶⁷ μs = 5.9 × 10³⁶ years | 5.9 × 10³⁰ years |
| 26 characters (permutation) | 26! = 4 × 10²⁶ | 2 × 10²⁶ μs = 6.4 × 10¹² years | 6.4 × 10⁶ years |

+   > DES（数据加密标准）：56位密钥（已经不安全）
+   > 3DES：168位密钥
+   > AES（高级加密标准）：>=128位密钥
+   > （注：1μs = 0.001ms = 0.000001 s）