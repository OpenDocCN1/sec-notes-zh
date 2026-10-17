# # ElGamal Cryptosystem

例：P = 17, α= 3, xA = 2, xB = 5, m = 11，  
从A发送到B, A选择k = 7.

求：密文(c1, c2)并解密

加密：YA = αxA mod P = 32 mod 17 = 9  
YB = αxB mod P = 35 mod 17 = 5  
K = (YB)k mod P = 57 mod 17 = 10  
c1 = αk mod P = 37 mod 17 = 11  
c2 = mK mod P = 10x11 mod 17 = 8  
所以， 密文C = (c1, c2) = (11, 8)

解密：K = c1xB mod P = 115 mod 17 = 10  
c2 = mK mod P = 10m mod 17 = 8  
m = c2/K mod P = c2K-1 mod P  
K K-1 mod P = 1， 即10 K-1 mod 17 = 1， 得K-1 = 12  
所以， 明文m = c2K-1 mod P = 8x12 mod 17 = 11