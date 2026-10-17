# RIPEMD-160——算法步骤

+   3. 处理每个512位消息分组
+     > 第q个分组处理表示为： CV_{q+1}=H_{RIPEMD}(CV_q, Y_q)
+     • 其中：
+       • H_{RIPEMD}是压缩函数
+       • Y_q是第q个分组
+       • CV_q、CV_{q+1}是压缩函数H_{RIPEMD}的输入、输出，CV_0=IV
+       • CV_q也称之为链接变量
+     • 每次处理前，将CV_q输入到寄存器ABCDE和A′B′C′D′E′中，处理后将寄存器ABCDE和A′B′C′D′E′中内容计算后输出到CV_{q+1}

+   4. 第L个分组的压缩函数H_{RIPEMD}处理的输出 CV_L即是160位的消息摘要