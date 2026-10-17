# # ElGamal数字签名的验证

+   - 验证：给定m, r, 和s，容易计算  
+     \( \alpha^m \mod p = Y_A^r \cdot r^s \mod p \)，看其是否一致，k不能重复使用。

+   - 例：\( p = 17, \alpha = 3, x_A = 2, x_B = 5, m = 11, k = 5 \)，求签名及验证。  
+     签名：\( r = \alpha^k \mod p = 3^5 \mod 17 = 5 \)，  
+     \( 11 = (2x5 + 5s) \mod 16 = (10 + 5s) \mod 16 \)，  
+     \( 5s \mod 16 = 1, s = 13 \)。  
+     所以，签名为\( (5, 13) \)。  
+     验证：\( \alpha^m \mod p = 3^{11} \mod 17 = 10^2 \times 10 \times 9 \mod 17 = 7 \)，  
+     \( Y_A^r \cdot r^s \mod p = (3^2)^5 \times 5^{13} \mod 17 = 7 \)