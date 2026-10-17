# # 算法的证明

# ## 加密算法：

假设 M 是明文（M<n），那么密文就是 C = M<sup>e</sup>mod n。

# ## 解密算法：

假设 C 是密文，那么明文就是 M = C<sup>d</sup> mod n。

证明：

(1) 由于 C<sup>d</sup> = M<sup>e*d</sup> = M<sup>k*Φ(n)+1</sup> (mod n)，

如果 M 和 n 是互素的，

显然直接由欧拉定理我们就能得到：

C<sup>d</sup> = M<sup>k*Φ(n)</sup>*M<sup>1</sup> = M (mod n) = M。

ed≡1(mod(p-1)(q-1))