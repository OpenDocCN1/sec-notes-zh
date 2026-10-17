# 三 客户机验证和密钥交换

![](bbox=[0.014, 0.012, 0.903, 0.228])

# The Client Certificate message is composed of

+   a. The server Identifier information
+   b. A Digital Certificate of the client information encrypted with the CAs Private Key

同时： 1) The Client Authenticates the Server with the CA.

客户利用服务器传过来的信息验证服务器的合法性，服务器的合法性包括：证书是否过期，发行服务器证书的 CA 是否可靠，发行者证书的公钥能否正确解开服务器证书的“发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配。如果合法性验证没有通过，通讯将断开；如果合法性验证通过，将继续进行第四步。。

+   2) Generates a session key (psuedo-random number) to use as a Pre-Master Key then
+   3) Encrypts the session key with the server's public key.