# 单向认证——公钥加密方法

+   - 保密性
+     A→B: E_KUb [Ks] || E_Ks [M]
+   - 认证
+     A→B: M || E_KRa [H(M)] || E_KRauth [T || ID_A || KU_a]