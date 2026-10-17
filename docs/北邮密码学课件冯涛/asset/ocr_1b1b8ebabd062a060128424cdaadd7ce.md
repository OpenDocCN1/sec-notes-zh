# Blum Blum Shub 发生器

+   > 基于公钥算法
+   > use least significant bit from iterative equation:
+   >   - x_i = x_{i-1}^2 mod n
+   >   - 其中,n=p.q,并且素数 p,q=3 mod 4
+   > 是不可预测的,可通过续位测试
+   > 安全性基于N的因子分解困难性
+   > is unpredictable given any run of bits
+   > 会很慢,因为必须使用非常大的数
+   > 用来加密显得太慢,用来产生密钥是很好的选择