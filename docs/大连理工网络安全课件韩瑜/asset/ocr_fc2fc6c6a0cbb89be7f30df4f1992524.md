# # 教师

# # 迟到者

知道： y, p, q  
不知道： x(姓名/学号)

秘密： x(姓名/学号)  
公开： y=g^x, p, q|p-1, g(g^q=1 mod p)

随机数r  
s = g^r mod p

随机数t  
p

v = r + tx mod q

验证g^v = s × y^t?

Schnorr零知识证明方案