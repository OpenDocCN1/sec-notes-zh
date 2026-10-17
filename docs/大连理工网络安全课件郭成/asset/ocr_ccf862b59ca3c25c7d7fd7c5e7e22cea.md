# 求多项式的最大公因式

$$\begin{aligned}
a(x)&=x^4+x^3+x^2+1\\
b(x)&=x^3+x^2+x+1
\end{aligned}$$

$$\frac{x+1}{x^3+x^2+1}=x+\frac{x^4+x^3+x}{x^3+x^2+1}$$

$$x^4+x^3+x=x(x^3+x^2+1)-1$$

从而获得 $r_2(x)=0,\ q_2(x)=x+1$。

因此，$\gcd[a(x),b(x)]=r_1(x)=x^3+x^2+1$。