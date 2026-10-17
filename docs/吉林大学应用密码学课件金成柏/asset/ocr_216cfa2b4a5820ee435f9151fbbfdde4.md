# 消息认证码——基本用法

消息认证

A→B:
M | C_K (M)

消息认证和保密性：与明文有关的认证

A→B:
E_K2 [M | C_K1 (M) ]

消息认证和保密性：与密文有关的认证

A→B:
E_K2 [M] | | C_K1 (E_K2 (M) )

Figure 11.4 Basic Uses of Message Authentication Code (MAC)

![](bbox=[0.27, 0.2, 0.97, 0.45])

![](bbox=[0.27, 0.45, 0.97, 0.7])

![](bbox=[0.27, 0.7, 0.97, 0.88])