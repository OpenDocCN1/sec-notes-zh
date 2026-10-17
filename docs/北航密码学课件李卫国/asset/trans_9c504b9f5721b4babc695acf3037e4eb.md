

### 例子：

计算 \(16^{15} \bmod 4731\)。

- (1) \(a \leftarrow 16, b \leftarrow 15, c \leftarrow 1\)
- (2) \(b \leftarrow 14, c \leftarrow 16\)
- (3) \(b \leftarrow 7, 16^2 = 256 \pmod{4731}, a \leftarrow 256\)
- (4) \(b \leftarrow 6, 256 \times 16 = 4096 \pmod{4731}, c \leftarrow 4096\)
- (5) \(b \leftarrow 3, 256^2 = 4033 \pmod{4731}, a \leftarrow 4033\)
- (6) \(b \leftarrow 2, 4033 \times 4096 = 3247 \pmod{4731}, c \leftarrow 3247\)
- (7) \(b \leftarrow 1, 4033^2 = 4642 \pmod{4731}, a \leftarrow 4642\)
- (8) \(b \leftarrow 0, 4642 \times 3247 = 4339 \pmod{4731}, c \leftarrow 4339\)
- (9) \(b = 0, 16^{15} = 4339 \pmod{4731}\)

### 例1 计算 \(16^{15} \bmod 4731\)

\[
16^{15} = 16^{8+4+2+1} = 16^8 \cdot 16^4 \cdot 16^2 \cdot 16 \pmod{4731}
\]

\[
16^2 = 256
\]

\[
16^4 = 256^2 \equiv 4033 \pmod{4731}
\]

\[
16^8 = (16^4)^2 \equiv 4033^2 \equiv 4642 \pmod{4731}
\]

所以：

\[
16^{15} \equiv 4642 \cdot 4033 \cdot 256 \cdot 16 \pmod{4731}
\equiv 4339 \pmod{4731}
\]

### 2、利用中国剩余定理加快RSA解密

理由：RSA 的解密运算要作大数的模幂运算

\[
m \equiv c^d \pmod{pq}
\]

关键：利用 Fermat 定理和中国剩余定理

- 1. 由 Fermat 定理：\(c^{p-1} \equiv 1 \pmod p\) 得
  \[
  m_1 \equiv c^d \equiv c^{d \bmod (p-1)} \pmod p
  \]
  \[
  m_2 \equiv c^d \equiv c^{d \bmod (q-1)} \pmod q
  \]
- 2. 由中国剩余定理先求解：
  \[
  p y_1 \equiv 1 \pmod q
  \]
  \[
  q y_2 \equiv 1 \pmod p
  \]

节省 \(1/4\) 的资源。

- 3. 计算：
  \[
  m \equiv q m_1 y_1 + p m_2 y_2 \pmod{pq}
  \]

## 四、RSA的使用

### 1、错误传输的影响

传输中的一位错误将会导致 RSA 的崩溃，对明文会有重大影响。

解决：密文传输要有纠错机制。

### 2、加解密的运算速度

加解密的运算速度慢，适合短信息加密。

通常用于数字签名、身份认证，或用于对密钥的加密——用于对称密钥的传输。

## 习题

- 1. 应用 RSA 算法对下列情况实现加密和解密：
  - a. \(p=11,\ q=13,\ e=11,\ M=7\);
  - b. \(p=3,\ q=13,\ d=7,\ C=5\).
- 2. 在 RSA 公钥加密方案中，每个用户有一个公开密钥 \(e\) 和一个私有密钥 \(d\). 假设他已泄露了 \(d\) 和 \(e\)，他不是产生一个新的模数，而是产生一对新的 \(d\) 和 \(e\)，这样做安全吗？为什么？

## 习题

- 3. 计算 \(15^{17} \bmod 5207\).
- 4. 利用 Fermat 定理和中国剩余定理计算
  \[
  m \equiv 13^{102} \pmod{43 \times 53}
  \]
- 5. 一 RSA 密码算法中：\(p=13,\ q=17,\ d=11,\ C=7\)，请解密求明文 \(M\).

在 RSA 公钥加密方案中，每个用户有一个公开密钥 \(e\) 和一个私有密钥 \(d\). 假设他已泄露了 \(d\) 和 \(e\)，他不是产生一个新的模数，而是产生一对新的 \(d\) 和 \(e\)，这样做安全吗？

### 不安全的理由：

设新的公开密钥为 \(e_1\)，则 \((e_1,\varphi(n))=1\)。

- 1) 计算 \(g=(e_1, ed-1)\);
- 2) 计算 \(h=(ed-1)/g\);
- 3) 求数 \(r, d_1\) 使得 \(rh + d_1 e_1 = 1\).

注意到 \(g \mid e_1\)，则 \((g,\varphi(n))=1\)。由于 \(\varphi(n) \mid (ed-1)\)，从而 \(\varphi(n) \mid h\)。故得：

\[
d_1 e_1 \equiv 1 \pmod{\varphi(n)}
\]

## § 10.4 Rabin 加密算法

密钥生成：

取素数 \(p, q\) 作为私钥，要满足

\[
p \equiv 3 \pmod 4,\quad q \equiv 3 \pmod 4
\]

即 \(p,q=4k+3\)。

取 \(n=pq\) 作为公钥。

加密：\(c \equiv x^2 \pmod n\)

解密：模开方运算：\(x^2 \equiv c \pmod n\) 的解。

### 解密密文 \(c\)：

因为 \(c\) 是 \(\bmod p\) 的平方剩余（QR），可知

\[
c^{(p-1)/2} \equiv 1 \pmod p
\]

从而

\[
(c^{(p+1)/4})^2 \equiv c^{(p+1)/2} \equiv c^{(p-1)/2} c \equiv c \pmod p
\]

同样可得

\[
(c^{(q+1)/4})^2 \equiv c \pmod q
\]

所以

\[
x = \pm c^{(p+1)/4} \pmod p
\]

\[
x = \pm c^{(q+1)/4} \pmod q
\]

![](img/b96edad7944fa6b4b037429dcfe38a6f_399_0.png)

令

\[
r \equiv c^{(p+1)/4} \pmod p,\qquad s \equiv c^{(q+1)/4} \pmod q
\]

求 \(a\) 和 \(b\) 使得

\[
ap + bq = 1
\]

计算

\[
x \equiv aps + bqr \pmod n
\]

\[
y \equiv aps - bqr \pmod n
\]

则明文 \(P\) 为下述之一：

\[
x,\ -x,\ y,\ -y \pmod n
\]

**Bob：**

- 选出 \(p,\ q \equiv 3 \pmod 4\)
- \(n = p \times q\)
- 公钥 \(n\)
- 私钥 \(p,q\)

**Alice：**

- 明文 \(P\)
- \(C = P^2 \bmod n\)
- 密文 \(C\)
- 求 \(C\) 的平方根
- 明文 \(P\)

### Rabin 密码中对应于密文 \(C\) 有四个可能的明文。

如果明文是一个随机序列，则无法确定该选择哪一个。一个解决的办法是加已知的前缀。

### 例1

私钥 \(p=23,\ q=7\)，公钥 \(n=161\)。

明文 \(P = 24\)。

加密：

\[
C = 24^2 \bmod 161 = 93
\]

解密：

\[
P \equiv 93^{(23+1)/4} \pmod{23} \equiv 1^6 \pmod{23} \equiv 1 \pmod{23}
\]

\[
P \equiv 93^{(7+1)/4} \pmod{7} \equiv 2^2 \pmod{7} \equiv 4 \pmod{7}
\]

### 中国剩余定理求解

\[
x \equiv \pm 4 \pmod 7
\]

\[
x \equiv \pm 1 \pmod{23}
\]

四种组合：

- \(x \equiv 4 \pmod 7,\ x \equiv 1 \pmod{23}\)
- \(x \equiv 4 \pmod 7,\ x \equiv -1 \pmod{23}\)
- \(x \equiv -4 \pmod 7,\ x \equiv 1 \pmod{23}\)
- \(x \equiv -4 \pmod 7,\ x \equiv -1 \pmod{23}\)

得

\[
x = 116,\ 137,\ 24,\ 45 \pmod{161}
\]

### Rabin 密码的问题

已知明文的攻击：

窃密者选一随机数 \(x\)，并求出 \(x^2 \equiv c \pmod n\)。

窃密者将 \(c\) 送给 Bob 要求解密。

Bob 解密后的明文 \(m\) 被窃密者获取。

如果 \(m \not\equiv \pm x \pmod n\)（其概率为 \(1/2\)），则 Eve 可获得 \(c\) 的 4 个平方根的两个独立解，并据此可解 \(n\) 的因子，故此系统可被破解。

### Rabin 密码的安全性

求解方程

\[
x^2 \equiv a \pmod n
\]

的计算复杂度与对 \(n\) 进行因数分解等价。

只要 \(n\) 足够大，Rabin 密码与 RSA 同样安全。

## § 10.5 Elgamal 密码

密钥生成：

- 1. 取大素数 \(p\)，取 \(d \in \mathbb{Z}_p^*\)，且 \(1 \leq d \leq p-2\)。
  取 \(e_1\) 为 \(\mathbb{Z}_p^*\) 的一个生成元（本原根）。
- 2. 计算 \(e_2 = e_1^d \bmod p\)。

公钥：\((e_1, e_2, p)\)

私钥：\(d\)

加密：明文 \(P < p\)，任选随机 \(r\)。

\[
C_1 \equiv e_1^r \pmod p,\quad C_2 \equiv P \times e_2^r \pmod p
\]

密文 \(C = (C_1, C_2)\)。

解密：\(P = C_2 \times (C_1^d)^{-1} \bmod p\)。

证明：

\[
\begin{aligned}
P' &= C_2 \times (C_1^d)^{-1} \bmod p \\
&= (P \times e_2^r) \times (e_1^{rd})^{-1} \bmod p \\
&= (P \times e_1^{rd}) \times (e_1^{rd})^{-1} \bmod p \\
&= P
\end{aligned}
\]

![](img/b96edad7944fa6b4b037429dcfe38a6f_408_0.png)

**Alice：**

公钥 \((e_1, e_2, p)\)

\[
C_1 \equiv e_1^r \pmod p,\quad C_2 \equiv P \times e_2^r \pmod p
\]

明文 \(P\)

密文 \(C\)

**Bob：**

- 1. 选出 \(p\) 和生成元 \(e_1\)
- 2. 选出 \(d\)
- 3. 计算 \(e_2 = e_1^d \bmod p\)

私钥 \(d\)

\[
P = C_2 \times (C_1^d)^{-1} \pmod p
\]

明文 \(P\)

