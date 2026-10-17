明文（Plaintext）： 原信息的初始形式 P
密文（CypherText）： 编码后的加密信息 C
加密算法（Encryption Algorithm）： C=E (P)
解密算法 (Decryption Algorithm)： P=D(C)
密码系统满足： P=D(E(P))

密钥（Secret Key）:
需要使用密钥的加密算法，记为： C=E(K,P),
加密与解密的密钥相同，即： P=D(K,E(K,P))
加密与解密的密钥不同，则： P=D(K_D,E(K_E,P))