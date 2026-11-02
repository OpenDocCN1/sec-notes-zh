
# [PC样本分析] 窃密木马借 壁纸引擎 传播，Steam 再中招

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1912507
- 抓取时间: 2026-11-01T23:48:18

近期，火绒安全实验室收到用户反馈称《Wallpaper Engine:壁纸引擎》下载的壁纸打开之后造成 Steam 账号异常，火绒安全工程师第一时间为用户提供技术支持，提取样本并进行分析。分析过程中发现该程序为正常壁纸文件捆绑了恶意的 Steam 盗号木马，用户在点击该程序时只会看到壁纸界面，但盗号木马在后台程序也会“悄悄”运行。目前，火绒安全产品可对上述病毒进行拦截查杀，请广大用户及时更新病毒库以提高防御能力。

《Wallpaper Engine:壁纸引擎》曾被指责 "黄油" 与 "盗号木马”泛滥，在此，火绒工程师建议大家在下载壁纸时，尽量选择正规可信的壁纸程序，并保持火绒等安全软件的开启。

## 样本执行流程

![](https://attach.52pojie.cn/forum/202404/11/150433i3cacdat7z5socqt.png)

## 一、样本分析

### 文件伪装与自解压

文件打开后是一个以 HTML 后缀伪装的 exe 文件 "`删掉HTML后缀后打开.exe`" ，该 exe 实际上是一个自解压压缩包，并会在启动时执行 `wehelpe.exe` 和 `rE.exe`：

![](https://attach.52pojie.cn/forum/202404/11/150436nttcjc6tcfjrtruc.png)

其中 `rE.exe` 是《Wallpaper Engine:壁纸引擎》的正常壁纸程序，而 `wehelpe.exe` 则是由易语言编写的 Steam 恶意盗号木马：

![](https://attach.52pojie.cn/forum/202404/11/150438r6ej9qba90qjm8e3.png)

### 前期准备

样本在开始盗号前，会先设立一系列相关操作保证盗号流程顺利进行，其中包括提升权限、持久化、强制重新登录等操作，具体细节如下：

#### 提权操作
获取 `SE_DEBUG_NAME` 权限，该权限允许跨进程操作，用于后续读取 Steam 内存：

![](https://attach.52pojie.cn/forum/202404/11/150440eild93idpnb9zpww.png)

#### 驻留操作
对 `wehelpe.exe` 和 `wallpaper64` 设立开机自启动注册表来进行驻留：

![](https://attach.52pojie.cn/forum/202404/11/150443crsvmm2xuroou2o9.png)

#### 强制重新登录操作
设置 Steam 下 `AutoLoginUser` 字符为 0，这样会迫使用户重新登录，刷新 token 时效：

![](https://attach.52pojie.cn/forum/202404/11/150445irdx604dkh74zx4x.png)

### 内存遍历 Steam 搜寻 Token

内存遍历 Steam 搜寻 Token 是以线程方式进行的，首先遍历搜寻 `steam.exe` 的进程号，如果不存在则以 `steamchina.exe` 进行代替：

![](https://attach.52pojie.cn/forum/202404/11/150447bzcuoajzshsyubu6.png)

样本会根据前面获取到的 Steam 程序句柄，通过下图中的 `search_from_steam_and_get_address`（`sub_4041A9`）函数搜索 Steam 进程内存，获取 Token 标签出现的所有地址，存储在参数数组中并返回出现的次数：

![](https://attach.52pojie.cn/forum/202404/11/150449abe04qezl0lttllk.png)

![](https://attach.52pojie.cn/forum/202404/11/150451v4659rd6igzz9z92.png)

随后根据前面获取到的次数，循环遍历每个 Token 所在的地址，获取完整的字符串：

![](https://attach.52pojie.cn/forum/202404/11/150516oketvbcjjjnvlf1t.png)

### Steam 中 Token 解析

Steam 中的 Token 以标识符 `{ "typ": "JWT", "alg": "EdDSA" }` 开头，并以 `"."` (2E) 连接 Token 主体和其它内容，它们都以 base64 形式存在于 Steam 进程的内存中。

![](https://attach.52pojie.cn/forum/202404/11/150518yc44y4t4sqz9qfy8.png)

其中，对于 Token 字段的解析如下：
- "`iss`": 表示签发者（issuer），即颁发该令牌的实体。
- "`sub`": 表示主题（subject），即令牌所代表的用户或实体。
- "`aud`": 表示观众（audience），即令牌预期接收者。
- "`exp`": 表示过期时间（expiration time），即令牌过期时间戳。
- "`nbf`": 表示不生效时间（not before），即令牌生效时间戳，在此时间之前令牌无效。
- "`iat`": 表示发布时间（issued at），即令牌发布时间戳。
- "`jti`": 表示 JWT ID，即令牌唯一标识符。
- "`oat`": 表示令牌签发时间（original issued at），即令牌最初签发时间戳。
- "`per`": 表示权限（permissions），即令牌所具有的权限级别。
- "`ip_subject`": 表示主题的 IP 地址，即主体的 IP 地址。
- "`ip_confirmer`": 表示确认者的 IP 地址，即对令牌进行确认的实体IP 地址。

下图是本机测试 Steam 进程中提取的两种类型Token：
- 一种是由 Steam 签发的用于用户与系统之间通信的 Token，"`aud`" 字段值 "`client`"、"`web`"、"`renew`"、"`deriver`" 表示可用于客户端应用程序、Web 应用程序、续订令牌、生成其它令牌等操作。
- 另一种是用于机器之间通信的 Token，"`aud`" 字段值为 "`machine`"，常用于不同服务之间通信。

![](https://attach.52pojie.cn/forum/202404/11/150521lsyy1iyyyyyyy8oo.png)

### 数据上传

随后样本会解密出要接收数据的网址及一些分隔符，以 "`|xxxx|ey.txt|xxxx|----Token 标识……`" 的方式拼接以备上传：

![](https://attach.52pojie.cn/forum/202404/11/150523asmb4gs2bbkmld4n.png)
