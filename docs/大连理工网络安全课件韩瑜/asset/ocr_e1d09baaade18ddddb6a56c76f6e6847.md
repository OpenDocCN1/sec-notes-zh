![](bbox=[0.045, 0.063, 0.978, 0.984])

IPv4 数据包结构：原始 IP 头部、TCP、Data；加入 AH 后，新 IP 头部、AH、原始 IP 头部、TCP、Data。其中认证范围为新 IP 头部中除可变字段外的字段，以及 TCP 和 Data。

IPv6 数据包结构：原始 IP 头部、扩展头部（如存在）、TCP、Data；加入 AH 后，新 IP 头部、扩展头部、AH、原始 IP 头部、扩展头部、TCP、Data。其中认证范围为新 IP 头部及其扩展头部中除可变字段外的字段，以及 TCP 和 Data。

# AH认证

# IPv4 Header

# IPv6 Header

| 可变域 | 不变域 |
|---|---|
| Type of service | Version |
| Flags | IHL |
| Fragment offset | Total Length |
| TTL | Identification |
| Header checksum | Protocol |
| Options | Source address |
|  | Destination address |
|  | Data |

| 可变域 | 不变域 |
|---|---|
| Priority | Version |
| Flow label | Payload Length |
| Hop limit | Next header |
|  | Source address |
|  | Destination address |