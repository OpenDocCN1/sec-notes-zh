
## PGP——信任关系

- 信任处理操作
- 1. 所有者信任域（OWNER TRUST）
  - 当A向公钥环中插入一个新的公钥时
    - 如果所有者为A（自己），则该域自动设为ultimate trust
    - 否则，A必须键入相应的信任级别
- 2. 签名信任域（SIGTRUST）
  - 当新的公钥进入到环中，可能有一个或多个签名与它相联系。当插入一个签名时，PGP将搜索公钥环，看该签名的作者是否是已知的公钥所有者
    - 如果是，则该域设为签名作者的OwnerTrust值
    - 否则，该域设为unknown user
- 3. 密钥合法性域（KEYLEGIT）
  - 根据表项中签名信任域计算密钥合法性域。
    - 如果至少有一个签名的签名信任值为ultimate，则该域置为complete
    - 否则，PGP计算加权信任值

## PGP——信任模型示例

![](img/0935d481fd26d10140d7cd560b6540ba_498_0.png)

? = unknown signatory

X → Y = X is signed by Y

○ = key's owner is trusted by you to sign keys

◐ = key's owner is partly trusted by you to sign keys

● = key is deemed legitimate by you

Figure 15.7 PGP Trust Model Example

## S/MIME

- **Secure/Multipurpose Internet Mail Extensions**
  - 增强了互联网络电子邮件格式标准MIME的安全性
  - RFC为S/MIME定义了许多文档
    - 重要的有RFC 2630、RFC 2632、RFC 2633
- **许多现代的邮件代理都支持S/MIME**
  - MS Outlook、Netscape等
- **为了理解S/MIME，我们先介绍**
  - RFC 822
  - MIME

## S/MIME——RFC 822

- RFC 822定义了用电子邮件传输的文本消息格式
  - 消息格式包含若干行的报头和无限制的正文
  - 报头通常由关键字、冒号 “:”和参数组成
  - 报头和正文间用一个空行隔开

```
Date: Tue, 16 Jan 1998 10:37:17 (EST)
From: "William Stallings" <ws@shore.net>
Subject: The Syntax in RFC 822
To: Smith@Other-host.com
Cc: Jones@Yet-Another-Host.com

Hello. This section begins the actual
message body, which is delimited from the
message heading by a blank line.
```

## S/MIME——MIME

- MIME是对RFC 822框架的一个扩展
  - 期待解决使用SMTP、RFC 822电子邮件中存在的问题
  - 相关文档：RFC 2045-2049
- SMTP/RFC 822存在的限制
  - SMTP不能传输可执行文件或其它二进制对象，SMTP仅限于传输7位ASCII码
  - SMTP网关在ASCII码和EBCDIC码之间转换时没有使用一致的映射
  - 一些SMTP实现与定义在RFC 821中的SMTP标准不完全一致
    - 回车和换行的增加、删除、重排
    - 长于76个字符的截断和转行
    - 去掉空白字符（制表符和空格符）
    - 将消息的行填充为等长
    - 将制表符转换为若干空格符

## S/MIME——MIME

- 要素
  - 定义了五个新的报头域
    - MIME-Version（MIME版本）：必须为1.0
    - Content-Type（内容类型）
    - Content-Transfer-Encoding（内容转换编码）
    - Content-ID（内容标识）
    - Content-Description（内容描述）
  - 定义了若干个内容格式
    - 从而标准化支持多媒体的电子邮件
  - 定义了编码转换方式
    - 使得任何内容格式均可转化为邮件系统认可的格式

## S/MIME——MIME内容类型

### Table 15.3 MIME Content Types

| Type | Subtype | Description |
|---|---|---|
| Text | Plain | Unformatted text; may be ASCII or ISO 8859. |
|  | Enriched | Provides greater format flexibility. |
| Multipart | Mixed | The different parts are independent but are to be transmitted together. They should be presented to the receiver in the order that they appear in the mail message. |
|  | Parallel | Differs from Mixed only in that no order is defined for delivering the parts to the receiver. |
|  | Alternative | The different parts are alternative versions of the same information. They are ordered in increasing faithfulness to the original, and the recipient's mail system should display the "best" version to the user. |
|  | Digest | Similar to Mixed, but the default type/subtype of each part is message/rfc822. |
| Message | rfc822 | The body is itself an encapsulated message that conforms to RFC 822. |
|  | Partial | Used to allow fragmentation of large mail items, in a way that is transparent to the recipient. |
|  | External-body | Contains a pointer to an object that exists elsewhere. |
| Image | jpeg | The image is in JPEG format, JFIF encoding. |
|  | gif | The image is in GIF format. |
| Video | mpeg | MPEG format. |
| Audio | Basic | Single-channel 8-bit ISDN mu-law encoding at a sample rate of 8 kHz. |
| Application | PostScript | Adobe Postscript |
|  | octet-stream | General binary data consisting of 8-bit bytes. |

## S/MIME——MIME转换编码

定义了两种编码方式

- 但Content-Transfer-Encoding可以取6个值
  - 两种编码方式：quoted-printable、base64
  - 当值为7 bit、8 bit、binary时，不进行编码
  - 当值为x-token时，需提供该自定义方式的名称

### Table 15.4 MIME Transfer Encodings

|  |  |
|---|---|
| 7bit | The data are all represented by short lines of ASCII characters. |
| 8bit | The lines are short, but there may be non-ASCII characters (octets with the high-order bit set). |
| binary | Not only may non-ASCII characters be present but the lines are not necessarily short enough for SMTP transport. |
| quoted-printable | Encodes the data in such a way that if the data being encoded are mostly ASCII text, the encoded form of the data remains largely recognizable by humans. |
| base64 | Encodes data by mapping 6-bit blocks of input to 8-bit blocks of output, all of which are printable ASCII characters. |
| x-token | A named nonstandard encoding. |
