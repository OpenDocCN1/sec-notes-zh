# PGP——消息的一般格式（从A到B）

说明

KeyID=密钥标识

Timestamp=时间戳

Content

Operation

Session key component

Signature

Message

Key ID of recipient's public key (KU_B)

Session key (K_s)

Timestamp

Key ID of sender's public key (KU_A)

Leading two octets of message digest

Message Digest

Filename

Timestamp

Data

E_KU_B

E_KR_A

ZIP

E_K_s

R64

Notation:
E_KU_B = encryption with user B's public key
E_KR_A = encryption with user A's private key
E_K_s = encryption with session key
ZIP = Zip compression function
R64 = Radix-64 conversion function

Figure 15.3 General Format of PGP Message (from A to B)