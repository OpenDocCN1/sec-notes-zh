# 双证书机制中证书及配置文件下载

双证书用户只能通过专用的客户端软件下载签名证书、加密证书及加密私钥。具体的流程如下：

+   用户在双证书客户端软件中输入签名证书的用户识别码（RefNum）和授权码（AuthCode）；

+   客户端软件以安全方式将RefNum/AuthCode传给Authority；

+   Authority验证用户身份通过；

+   Authority向KMC请求相应加密证书的私钥；

+   Authority检索用户签名证书及对应的加密证书；

+   Authority检索用户加密证书的RefNum/AuthCode；

+   将用户信息安全传给客户端软件；

+   客户端软件根据用户要求，制作证书文件；