# # SA(Security Association)

+   - 基本概念
+     - 是发送者和接收者两个IPSec系统之间的一个简单的单向逻辑连接.
+     - 因为SA是单个方向的，一个双向通信需要两个SA
+   - SA与IPSec系统中实现的两个数据库有关
+     - 安全策略数据库(SPD)；安全关联数据库(SAD)
+   - 每个SA通过三个参数来标识
+     - SPI(Security Parameters Index)
+     - 目标地址IP
+     - 安全协议标识