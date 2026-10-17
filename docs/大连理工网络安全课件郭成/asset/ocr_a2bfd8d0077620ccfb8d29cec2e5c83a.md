# 输出反馈模式Output FeedBack (OFB)

+   - 输出反馈模式Output FeedBack (OFB)
+     - 结构上类似CFB，但是OFB中加密函数输出被反馈回移位寄存器，CFB中是密文单元被反馈回移位寄存器。优点是传输中的比特差错不会传播，缺点是比CFB更容易受报文流篡改攻击。