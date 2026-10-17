# # Three-Way Authentication

+   - 使用三条消息(A->B, B->A, A->B)建立认证过程而不需要同步时钟
+   - 最后从A发往B的消息中包含签了名的临时交互号r<sub>b</sub>，这样，其中的时间戳就不用检查了