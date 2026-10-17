# # 椭圆曲线用于加密

+   - 找到一个难题：
+     - 考虑等式Q=kP，其中Q、P属于E<sub>p</sub>(a,b)， k<p
+     - 已知k和P，计算Q，是容易的
+     - 已知Q和P，计算k，是困难的
+   - 选择E<sub>p</sub>(a,b)的元素G,使得G的阶n是一个大素数
+     - G的阶是指满足nG=O的最小n值
+   - 秘密选择整数r，计算P=rG，然后
+     - 公开(p,a,b,G,P)， P为公钥
+     - 保密r
+   - 加密M：先把消息M变换成为E<sub>p</sub>(a,b)中一个点P<sub>m</sub>
+     - 然后，选择随机数k，计算密文C<sub>m</sub>={kG，P<sub>m</sub>+kP}
+     - 如果k使得kG或者kP为O，则要重新选择k。
+   - 解密C<sub>m</sub>：(P<sub>m</sub>+kP)-r(kG)=P<sub>m</sub>+krG-rkG=P<sub>m</sub>
+   - 加密信息有扩张