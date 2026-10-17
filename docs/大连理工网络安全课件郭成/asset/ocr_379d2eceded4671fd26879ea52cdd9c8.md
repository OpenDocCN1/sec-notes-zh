# # 扩展的Euclid算法求逆

如果a和b互素，则b有模a的乘法逆元。也就是说，如果gcd(a,b)=1,那么b有模a的乘法逆元。即对于正整数b<a,存在b<sup>-1</sup><a使bb<sup>-1</sup>=1mod a. 如果a是素数并且b<a，则显然a和b互素，且最大公因子为1.

我们已经证明过该式可以用扩展Euclid算法来解：
ax+by=d=gcd(a, b)
如果gcd(a, b)=1,则有ax+by=1.
[(ax mod a)+(by mod a)]mod a = 1 mod a
0+(by mod a)=1
如果by mod a=1，则y=b<sup>-1</sup>.