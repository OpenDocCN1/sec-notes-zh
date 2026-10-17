# § 10.5　Elgamal 密码

密钥生成：

+   1、取大素数 $p$，取 $d\in\mathbb{Z}_p^*$，且 $1\leq d\leq n-2$  
+   　　取 $e_1$ 为 $\mathbb{Z}_p^*$ 的一个生成元（本原根）
+   
+   2、计算 $e_2=e_1^d\bmod p$

公钥（$e_1,e_2,p$）

私钥 $d$