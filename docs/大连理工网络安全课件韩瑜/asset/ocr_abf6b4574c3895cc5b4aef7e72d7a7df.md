# # Kerberos管辖范围与多重服务

+   - 一个完整的Kerberos环境包括一个Kerberos服务器，一组工作站，和一组应用服务器，满足下列要求：
+     - Kerberos服务器必须在其数据库中拥有所有参与用户的ID(UID)和口令散列表。所有用户均在Kerberos服务器上注册。
+     - Kerberos服务器必须与每一个服务之间共享一个保密密钥。所有服务器均在Kerberos服务器上注册。

+   - 这样的环境被视为一个领域(realm)。
+     - 每一个辖区的Kerberos服务器与其它辖区内的Kerberos服务器之间共享一个保密密钥。两个Kerberos服务器互相注册。

![](bbox=[0.922, 0.925, 1.0, 1.0])