# # 会话状态参数

| 会话标识符 | Session identifier | 由服务器选择的任意字节顺序，用来确定活动或可恢复的会话状态 |
| --- | --- | --- |
| 对等实体证书 | Peer certificate | 对等实体的 X509.v3 证书。该状态的元素可为空 |
| 压缩方法 | Compression method | 压缩数据的算法优于加密算法 |
| 加密规格 | Cipher spec | 指定批量数据加密算法和用于 MAC 计算的哈希算法。同时指定密码属性，如 hash_size |
| 主控密钥 | Master secret | 由客户机和服务器共享的 48 位密码 |
| 是否可恢复 | Is resumable | 用来确定会话是否可用于初始化新连接的标志 |