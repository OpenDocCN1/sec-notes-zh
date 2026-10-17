# # 群、有限群和无限群

+   - 用 $N_n$ 表示 $n$ 个不同符号的集合，\{1,2,\ldots,n\}。$n$ 个不同符号的一个置换是一个 $N_n$ 到 $N_n$ 的一一映射。定义 $S_n$ 为 $n$ 个不同符号的所有置换组成的集合。$S_n$ 中的每一个元素都代表集合 $\{1,2,\ldots,n\}$ 的一个置换，容易验证 $S_n$ 是一个群：
+     - **A1：** 如果 $\boldsymbol{\pi},\boldsymbol{\rho}\in S_n$，则合成映射 $\boldsymbol{\pi}\bullet\boldsymbol{\rho}$ 根据置换 $\boldsymbol{\pi}$ 来改变 $\boldsymbol{\rho}$ 中元素的次序而形成，如，$\{3,2,1\}\bullet\{1,3,2\}=\{2,3,1\}$，虽然 $\boldsymbol{\pi}\bullet\boldsymbol{\rho}\in S_n$
+     - **A2：** 映射的合成显而易见满足结合律
+     - **A3：** 恒等映射就是不改变 $n$ 个元素位置的置换，对于 $S_n$，单位元是 $\{1,2,\ldots,n\}$
+     - **A4：** 对于任意 $\boldsymbol{\pi}\in S_n$，抵消由 $\boldsymbol{\pi}$ 定义置换的映射就是 $\boldsymbol{\pi}$ 的逆元，这个逆元总是存在，例如：$\{2,3,1\}\bullet\{3,1,2\}=\{1,2,3\}$

+   - 有限群Finite Group和无限群Infinite Group：如果一个群的元素是有限的，则该群称为有限群，且群的阶等于群中元素的个数；否则称为无限群