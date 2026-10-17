# DSA——验证签名

+   - 验证方收到消息M和签名 (r , s)
+   - 验证签名
+     - w = s⁻¹ (mod q)
+     - u1= (SHA(M) . w) (mod q)
+     - u2= (r . w) (mod q)
+     - v = (g^u1 . y^u2 (mod p)) (mod q)
+     - 如果v=r，则签名有效