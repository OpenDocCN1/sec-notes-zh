# 2、 利用中国剩余定理加快RSA解密

理由： RSA的解密运算要作大数的模幂运算
$m \equiv c^{d} \mod pq$

关键： 利用Fermat 定理和中国剩余定理

+   1. 由Fermat 定理: $c^{p-1} \equiv 1 \pmod p$ 得
+   $m_1 \equiv c^{d} \equiv c^{d \mod (p-1)} \pmod p$
+   $m_2 \equiv c^{d} \equiv c^{d \mod (q-1)} \pmod q$

+   2. 由中国剩余定理先求解:
+   $py_1 \equiv 1 \pmod q$
+   $qy_2 \equiv 1 \pmod p$

节省1/4的资源.

+   3. 计算:
+   $m \equiv qm_1y_1 + pm_2y_2 \pmod{pq}$