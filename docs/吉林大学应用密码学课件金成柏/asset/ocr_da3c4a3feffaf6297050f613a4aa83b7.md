# 简单的秘密密钥分配——安全性分析

中间人攻击（主动攻击）

+   1. A生成密钥对{KUa, KRa}，并将含有KUa和其标识IDA的消息发送给B
+   2. E截获该消息，生成其密钥对{KUe, KRe}，并将Kue||IDA发送给B
+   3. B生成秘密密钥（会话密钥）Ks，并发送EKue[Ks]
+   4. E截获该消息，并计算DKRe[EKue[Ks]]获得秘密密钥Ks
+   5. E发送EKUa[Ks]给A
+   6. A计算DKRa[EKUa[Ks]]获得秘密密钥Ks

这时，E获得了A、B通信的秘密密钥Ks