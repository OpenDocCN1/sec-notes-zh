情况2：如果$M$和$p$互素，由欧拉定理，$M^{\phi(p)}\ \mathrm{mod}\ p=1$。继续如下步骤：

$M^{k(p-1)(q-1)+1}\ \mathrm{mod}\ p=[(M)M^{k(p-1)(q-1)}]\ \mathrm{mod}\ p$

$=[(M)(M^{\phi(p)})^{k(q-1)}]\ \mathrm{mod}\ p$

$=(M\ \mathrm{mod}\ p)\times[(M^{\phi(p)})\ \mathrm{mod}\ p]^{k(q-1)}$

$=(M\ \mathrm{mod}\ p)\times(1)^{k(q-1)}$

$=M\ \mathrm{mod}\ p$

$[M^{k(p-1)(q-1)+1}-M]\ \mathrm{mod}\ p=[M^{k(p-1)(q-1)+1}\ \mathrm{mod}\ p]-[M\ \mathrm{mod}\ p]=0$

因此$p$整除$[M^{k(p-1)(q-1)+1}-M]$，同理$q$也整除$[M^{k(p-1)(q-1)+1}-M]$。

因为$p$和$q$为不同的素数，因此必定存在一个整数$r$满足

$[M^{k(p-1)(q-1)+1}-M]=(pq)r=nr$

因此，$n$整除$[M^{k(p-1)(q-1)+1}-M]$，所以$M^{k\phi(n)+1}\ \mathrm{mod}\ n=M^{k(p-1)(q-1)+1}\ \mathrm{mod}\ n=M$。