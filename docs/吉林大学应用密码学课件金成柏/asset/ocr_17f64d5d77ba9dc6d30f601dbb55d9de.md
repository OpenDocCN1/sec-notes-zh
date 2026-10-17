# 密钥管理——Oakley

+   - **Oakley密钥生成协议**
+     - 提供增值安全性
+     - 基于Diffie-Hellman算法的密钥交换协议，但进行了改进
+     - 增加新的属性
+       - Cookies：可以防止阻塞攻击
+       - 组（global params）
+         - 双方协商得到一个组，作为Diffie-Hellman密钥交换的全局参数。3种模幂运算和2种椭圆曲线运算
+       - Nonces：防止重放攻击
+       - 具有认证的DH，防止中间人攻击