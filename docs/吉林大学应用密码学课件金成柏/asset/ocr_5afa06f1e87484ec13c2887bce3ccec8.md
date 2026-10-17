# 认证头（AH）——格式

![](bbox=[0.16, 0.21, 0.88, 0.73])

+   - 邻接头（Next Header）：紧跟在此报头后面的头类型
+   - 载荷长度（Payload Length）：认证头长度减2（32 bit字为单位）
+   - 安全参数索引（SPI）
+   - 序列号（Sequence Number）：单调递增计数値
+   - 认证数据（Authentication Data）：包含完整性校验值（ICV）