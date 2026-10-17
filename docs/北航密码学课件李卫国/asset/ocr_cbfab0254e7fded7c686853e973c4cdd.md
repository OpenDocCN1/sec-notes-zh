# § 2 AES 算法详解

Rijndael 算法适用于128, 192, 256位的密钥长度。在此我们取定取定128位密钥进行讨论。

Rijndael 算法由10轮循环组成，每一轮的输入和输出均是128位 (再分成16个字节，每字节8bit) 的状态矩阵。

每一轮循环由4个基本步骤 (称为层-layer) 组成：

+   换字盒 （SubBytes）
+   行移位 （ShiftRows）
+   列混合 （MixColumns）
+   轮加密 （AddRoundKey）