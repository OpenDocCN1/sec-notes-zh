# 4、Hill 密码（矩阵密码）

Leste S Hill， 分组加密， 矩阵乘法

密钥 K = (kij) m×m ， K mod26 可逆

明文分成长度为 m 的分组—— 写成矩阵 P r×m

加密方法： 矩阵乘法

C = P * K mod 26

解密方法： P = C * K⁻¹ mod 26

m = 4

明文： code is ready 排成3x4 矩阵

给定4x4密钥矩阵： K =

| c | o | d | e |
| i | s | r | e |
| a | d | y | z |
| . | . | . | . |

| J | H | L | N |
| E | H | F | G |
| C | V | O | J |
| D | X | V | I |