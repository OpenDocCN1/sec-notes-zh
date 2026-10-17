
## AES的密钥扩展

![](img/09e6564ace8d10cc99172928c514fc45_176_0.png)

(b) Key and expanded key

### 密钥扩展的伪代码

```
KeyExpansion(byte key[4 * Nk], word w[Nb * (Nr + 1)], Nk)
begin
    i = 0
    while (i < Nk)
        w[i] = word[key[4*i], key[4*i+1], key[4*i+2], key[4*i+3]]
        i = i + 1
    end while

    i = Nk
    while (i < Nb * (Nr + 1))
        word temp = w[i - 1]
        if (i mod Nk = 0)
            temp = SubWord(RotWord(temp)) xor Rcon[i / Nk]
        else if (Nk = 8 and i mod Nk = 4)
            temp = SubWord(temp)
        end if
        w[i] = w[i - Nk] xor temp
        i = i + 1
    end while
end
```

Note that Nk=4, 6, and 8 do not all have to be implemented; they are all included in the conditional statement above for conciseness. Specific implementation requirements for the Cipher Key are presented in Sec. 6.1.

### 伪代码之二

```c
KeyExpansion(byte key[16], word w[44])
{
    word temp;

    for (i = 0; i < 4; i++)
        w[i] = (key[4*i], key[4*i+1], key[4*i+2], key[4*i+3]);

    for (i = 4; i < 44; i++)
    {
        temp = w[i-1];
        if (i mod 4 = 0)
            temp = SubWord(RotWord(temp)) ⊕ Rcon[i/4];
        w[i] = w[i-4] ⊕ temp;
    }
}
```

### Figure 5.6 AES Key Expansion

### G变换

- RotWord 执行字节循环左移：\([b_0,b_1,b_2,b_3] \to [b_1,b_2,b_3,b_0]\)
- SubWord 执行使用 S-盒进行字节替换
- 前两步的结果与轮常数 \(Rcon[j]\) 异或

| \(j\) | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| \(RC[j]\) | 01 | 02 | 04 | 08 | 10 | 20 | 40 | 80 | 1B | 36 |

### example

For example, suppose that the round key for round 8 is

```
EA D2 73 21 B5 8D BA D2 31 2B F5 60 7F 8D 29 2F
```

Then the first 4 bytes (first column) of the round key for round 9 are calculated as follows:

![](img/09e6564ace8d10cc99172928c514fc45_181_0.png)

## 数论入门

### 完全剩余系 Complete Set of Residues

\(Z_n=\{0,1,2,\ldots,n-1\}\)

定义：由同余概念，所有整数在模 \(n\) 中分成 \(n\) 类，以 \(n\) 除所得余数为 \(0,1,2,\ldots,n-1\) 的数各为一类。将每一类中取一个数作为代表形成的集合。

### 素数

素数又叫质数，除了 1 和本身，不能被其他数整除的数。

### 公因子

设 \(a,b\) 为整数，整除 \(a\) 的数组成集合 \(A\)，整除 \(b\) 的数组成集合 \(B\)，\(C=A\cap B\)，\(C\) 中的元素称为 \(a\) 和 \(b\) 的公因子。\((a,b)\) 表示 \(a\) 和 \(b\) 的最大公因子（gcd，Greatest Common Divisor），\((a,b)=1\) 表示 \(a\) 与 \(b\) 互素。

### 既约剩余系 Reduced Set of Residues \(Z_n^*\)

在模 \(n\) 的完全剩余系中，将所有与 \(n\) 互素的数取出形成的集合。

如：\(n=10\)

完全剩余系：\(\{0,1,2,3,4,5,6,7,8,9\}\)

既约剩余系：\(\{1,3,7,9\}\)

乘法逆元：在模 \(n\) 的既约剩余系中取一整数 \(a\)，则必存在另一个整数 \(b\)（也属于此既约剩余系），使得 \(ab\equiv 1 \pmod n\)，且此解唯一。此时，称 \(b\) 为 \(a\) 在模 \(n\) 的乘法逆元，\(b\) 可表示为 \(a^{-1}\)。

### 求乘法逆元

已知 \(a,n\)，且 \((a,n)=1\)，如何求 \(a^{-1}\)，使得 \(a \times a^{-1}\equiv 1 \pmod n\)？

1. 利用欧几里得算法

   欧几里得算法（Euclidean Algorithm，也叫辗转相除法），用于求 gcd。

2. 利用欧拉函数 \(\phi(n)\)（Euler's Totient Function）

   \(\phi(n)\) 为小于 \(n\)，且与 \(n\) 互素的所有整数的个数。即 \(\phi(n)\) 为模 \(n\) 的既约剩余系中所有元素的个数。

### 欧拉定理（Euler's Theorem）

- 若 \((a,n)=1\)，则 \(a^{\phi(n)}\equiv 1 \pmod n\)。

### 利用欧拉定理求乘法逆元 \(a^{-1}\)

若 \(\phi(n)\) 已知，则由欧拉定理可知：

\[
a \cdot a^{\phi(n)-1}\equiv 1 \pmod n
\]

因此：

\[
a^{-1}=a^{\phi(n)-1}
\]

假设 \(a=3,\ n=8\)，利用欧拉定理，求 \(a^{-1}\)。可得 \(a^{-1}=3\)。

### 欧拉定理的证明

**Thm:** Let \(s=\{r_1,\ldots,r_{\phi(n)}\}\) be a reduced residue system modulo \(n\). If \((a,n)=1\), \(a\in\mathbb{Z}^+\), then the set \(s'=\{ar_1,ar_2,\ldots,ar_{\phi(n)}\}\) is also a reduced residue system modulo \(n\).

\[
\prod_{i=1}^{\phi(n)}(ar_i)\bmod n=\prod_{i=1}^{\phi(n)}r_i\bmod n
\]

\[
\left(a^{\phi(n)}\bmod n\right)\left(\prod_{i=1}^{\phi(n)}r_i\bmod n\right)=\prod_{i=1}^{\phi(n)}r_i\bmod n
\]

\[
a^{\phi(n)}\bmod n=1
\]

### 费马定理（Fermat's Theorem）

- **Thm:** If \(p\) is prime and \((a,p)=1\), then
  \[
  a^{p-1}\equiv 1 \pmod p
  \]
  （其中 \((a,p)=1\)）。

### \(ax\equiv b\pmod n\) 的求解过程

1. 利用欧几里得算法求出 \(d=(a,n)\)。若 \(d\) 不能整除 \(b\)，则无解。
2. 若 \(d\) 能整除 \(b\)，则令：

\[
a'=\frac{a}{d},\quad b'=\frac{b}{d},\quad n'=\frac{n}{d}
\]

因为 \((a',n')=1\)，\(a'x'\equiv b'\pmod{n'}\) 有唯一解，假设此解为 \(x_0\)。

利用欧几里得算法求出 \(a'\) 模 \(n'\) 的乘法逆元 \((a')^{-1}\)，

\[
x'=(a')^{-1}b'\pmod{n'}
\]

\[
x_0=x'\pmod n
\]

令

\[
x=x_0+(n/d)t\pmod n,\quad t=1,2,\ldots,d-1
\]

即可求出剩下的 \(d-1\) 个解。

### 求解 \(9x\equiv 12\pmod{15}\)

(1) 求出 \(d=(a,n)=(9,15)=3\)。3 能整除 12，因此 \(x\) 有解，且有 3 个解。

(2)

\[
a'=\frac{a}{d}=3,\quad b'=\frac{b}{d}=4,\quad n'=\frac{n}{d}=5
\]

\[
3x'\equiv 4 \pmod 5
\]

\[
3\times2\equiv 1 \pmod 5
\]

\[
3^{-1}\equiv 2 \pmod 5
\]

\[
x'=(2\times4)\pmod 5=3
\]

\[
x_0=x'\pmod n=3\pmod{15}=3
\]

\[
x_1=x_0+\left(\frac{n}{d}\right)\times t_1=3+\left(\frac{15}{3}\right)\times1=8
\]

\[
x_2=x_0+\left(\frac{n}{d}\right)\times t_2=3+\left(\frac{15}{3}\right)\times2=13
\]
