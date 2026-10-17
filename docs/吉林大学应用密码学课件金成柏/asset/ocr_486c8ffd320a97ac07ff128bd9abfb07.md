# 流密码——考虑的因素

# 需考虑的主要因素

+    - 密钥流序列的周期要长
+    - 密钥流应该尽可能接近一个真正的随机数流的特征
+    - 密钥应该足够长

# 流密码和分组密码速度对比

Table 6.2 Speed Comparisons of Symmetric Ciphers on a Pentium II

| Cipher | Key Length | Speed (Mbps) |
|---|---|---|
| DES | 56 | 9 |
| 3DES | 168 | 3 |
| RC2 | variable | 0.9 |
| RC4 | variable | 45 |