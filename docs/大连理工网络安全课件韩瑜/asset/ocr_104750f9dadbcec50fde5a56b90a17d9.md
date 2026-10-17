# # Diffie-Hellman密钥交换协议描述

当Alice和Bob要进行保密通信时，他们可以按如下步骤来做：

+   (1) Alice选取大的随机数x，并计算 X = g^x(mod P)
+   (2) Bob选取大的随机数x'，并计算 X' = g^x'(mod P)
+   (3) Alice将X传送给Bob； Bob将X'传送给Alice
+   (4) Alice计算K= (X')^x(mod P); Bob计算K' = (X)^x'(mod P)，

易见， K = K' = g^xx'(mod P)

+   由(4)知， Alice和Bob已获得了相同的秘密值K

+   双方以K作为加解密钥以传统对称密钥算法进行保密通信