# # RSA密码体制基本原理

定理：给定ed mod φ(n) = 1, m∈[0, n-1], gcd(m, n)=1，则：

(mᵉ mod n)ᵈ mod n = mᵉᵈ mod n = m

证明：∵ ed mod φ(n) = 1

∴ ed = kφ(n)+1，对某些整数k

∴ mᵉᵈ mod n = mᵏφ(n)+1 mod n

= m(mᵏφ(n) mod n) mod n

∴ mᵏφ(n) mod n = (mφ(n) mod n)ᵏ mod n

= 1ᵏ mod n = 1

∴ mᵉᵈ mod n = (m* 1) mod n = m