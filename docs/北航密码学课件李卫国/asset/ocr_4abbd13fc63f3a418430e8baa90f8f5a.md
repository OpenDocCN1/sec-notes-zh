# RSA算法证明：

p1 = c^d mod n = (p^e mod n)^d mod n

= p^{ed} mod n = p^{1 mod φ(n)} mod n

= p^{kφ(n)+1} mod n

= p mod n

所以，p1 = p mod n