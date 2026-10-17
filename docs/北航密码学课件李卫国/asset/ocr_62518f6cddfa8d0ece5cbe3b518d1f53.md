# 2、有密钥换位

密钥 K 就是一个指定的置换。

例如：把明文分成5个字母一段。每段都按照密钥指定的置换方式进行换位。

| 密钥 K = | 3 | 1 | 4 | 5 | 2 |
|---|---:|---:|---:|---:|---:|
|  | 1 | 2 | 3 | 4 | 5 |

明文：**Enemy attacks tonight！**

| Enemy | attac | kston | ightz |
|---|---|---|---|
| EEMY N | TAACT | TKONS | HITZG |

密文：**EEMYN TAACT TKONS HITZG**

| 2 | 5 | 1 | 3 | 4 |
|---|---|---|---|---|
| 1 | 2 | 3 | 4 | 5 |

| EEMYN | TAACT | TKONS | HITZG |
|---|---|---|---|
| Enemy | attac | kston | ightz |

![](bbox=[0.546, 0.064, 0.971, 0.604])