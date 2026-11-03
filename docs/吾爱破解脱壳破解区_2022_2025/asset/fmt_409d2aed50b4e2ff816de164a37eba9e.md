
![](https://attach.52pojie.cn/forum/202507/29/170733jummwu95ac95vcvx.png)
53.png (418.02 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc5MzA1NXw1ZjNkNzQ1MnwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA1NXw1ZjNkNzQ1MnwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

点击右侧筛选当前日志

![](https://attach.52pojie.cn/forum/202507/29/170735glpbfllbkbka6tbt.png)
54.png (146.51 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc5MzA1Nnw1YTU2MjNmMHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA1Nnw1YTU2MjNmMHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

我这里根据事件ID筛选

![](https://attach.52pojie.cn/forum/202507/29/170739i5z58z46biek515t.png)
55.png (231.2 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5MzA1N3w4ODI0NzYxN3wxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA1N3w4ODI0NzYxN3wxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

检索到一个创建用户的事件，由`Administrator`创建了`solar$`的账户，SID（安全ID）为`S-1-5-21-3845547894-970975367-1760185533-1000`记录时间为：2025/7/23 17:05:45。那么黑客将这个隐藏用户先后加入了哪几个用户组呢？

创建用户的时候，我们只能看到账户域和账户名，没有用户组的显示。那就根据本地组去检索，因为这个机器没有域控，只有在改变域控用户组的时候才是`4728`，本地组就是`4732`事件了。

![](https://attach.52pojie.cn/forum/202507/29/170741sc06l5eddanq0g42.png)
56.png (241.16 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=Mjc5MzA1OHxjOTk5OGE1ZHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA1OHxjOTk5OGE1ZHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

`4732`中有2个事件，第一个是17:05的时候，将`S-1-5-21-3845547894-970975367-1760185533-1000`加入了`Users`组（实际的用户名就是`solar$`这个用户）。

![](https://attach.52pojie.cn/forum/202507/29/170744xiuedobs4z7nq2si.png)
57.png (178.57 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MzA1OXw0YTE5Njc4YXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA1OXw0YTE5Njc4YXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

在17:06时，就将这个用户添加到了`Administrators`组，所以最终是在`Administrators`组中的。这个问题最终答案就是 `Users-Administrators`

那么还有最后一个问题，黑客通过远程桌面成功登陆系统管理员账号的网络地址及端口号是多少呢？

我们需要先筛选`4624`（登录成功）事件，然后寻找登录类型为10的事件。
`Logon Type = 10` 表示远程桌面登录（RDP）
`Logon Type = 3` 表示网络登录（如通过共享、远程管理等）

![](https://attach.52pojie.cn/forum/202507/29/170746abjbbziht6gb822h.png)
58.png (109.22 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc5MzA2MHw2ZjFiYWViNHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2MHw2ZjFiYWViNHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

根据网络信息得到，源网络地址和端口，这个任务的flag为：[49197" target="_blank" rel="noopener noreferrer nofollow"](http://192.168.186.139:49197:rel="nofollow noopener" href="http://192.168.186.139)

## 公交车系统攻击事件排查

### 背景
思而听公交系统被黑客攻击，黑客通过web进行了攻击并获取了数据，然后获取了其中一位驾校师傅在FTP服务中的私密文件，其后黑客找到了任意文件上传漏洞进行了GETshell，控制了主机权限并植入了挖矿网页挖矿病毒，接下来你需要逐步排查

注：流量中的21端口对应2121、80端口对应8090

### 1. 分析环境内的中间件日志，找到第一个漏洞(黑客获取数据的漏洞)，然后通过分析日志、流量，通过脚本解出黑客获取的用户密码数据，提交获取的前两个用户名，提交格式：flag{zhangsan-wangli}

按照应急响应思路，要排查：端口、进程、相关服务日志来确定

```bash
netstat -lntp
```

![](https://attach.52pojie.cn/forum/202507/29/170749mkpb18y566y63wbv.png)
59.png (209.42 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc5MzA2MXw5MGJkNjVmZHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2MXw5MGJkNjVmZHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

通过查看开放端口看到，目前是开放了：SSH、MySQL、WEB

```bash
cat -n /var/log/apache2/access.log
```

![](https://attach.52pojie.cn/forum/202507/29/170752utafynjidy99qnzy.png)
60.png (830.84 KB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc5MzA2MnxiZDRiZjFhOXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2MnxiZDRiZjFhOXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

在执行了查看`access.log`（中间件日志）后，可以看到足有近6000行日志

```bash
awk '{print $6, $7}' access.log
```

![](https://attach.52pojie.cn/forum/202507/29/170757iwa9v8w34hetxwzt.png)
61.png (1.33 MB, 下载次数: 5)
[href="forum.php?mod=attachment&aid=Mjc5MzA2NHxlMGQ1Njc1ZHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2NHxlMGQ1Njc1ZHwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

通过执行命令查看web请求的url不难看出，五千条之前几乎全部都是base64请求，然后就是名为`shell1.php`的文件请求

因为有pcap流量包，使用流量分析效果会更好一些

https://github.com/brimdata/zui

这里推荐一个十分好用的流量分析工具，工具会把流量拆分为五元组，可以极快的利用形似sql语句的方式进行筛选

将流量导入后执行以下语句

```zui
count() by _path|sort -r count
```

![](https://attach.52pojie.cn/forum/202507/29/170759ho5c8az8aum55w5a.png)
62.png (130.09 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5MzA2NXxkZjU5NTZlMXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2NXxkZjU5NTZlMXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:07 上传

可以看到服务类协议有FTP和HTTP的，其它都在OSI模型的其它层，可忽略

```zui
count () by id.orig_h,status_code,uri|status_code==200 | sort -r count
```

![](https://attach.52pojie.cn/forum/202507/29/170802b8dj54ggv3vgjojb.png)
63.png (277.11 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=Mjc5MzA2NnwwNmMyNGFmOXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2NnwwNmMyNGFmOXwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:08 上传

执行筛选HTTP协议语句后，看到请求`search.php`非常之多，那这里可以判断漏洞点可能和`search.php`有关联了

随便复制出其中一条日志，如下

```text
/search.php?query=JyBBTkQgKFNFTEVDVCAxNjQ5IEZST00gKFNFTEVDVChTTEVFUCg1KSkpR2NhRykgQU5EICdFQURnJz0nRUFEZw==
```

直接复制访问平台分配的WEB端口，比如我这里分配的端口为8090

![](https://attach.52pojie.cn/forum/202507/29/170804lqquqiz3q77zq6q9.png)
64.png (168.34 KB, 下载次数: 4)
[href="forum.php?mod=attachment&aid=Mjc5MzA2N3w5MzFlMDAxZnwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MzA2N3w5MzFlMDAxZnwxNzkzNTcwNzcyfDE2ODA4NnwyMDQ5NTU1&nothumb=yes)
2025-7-29 17:08 上传

直接复制这个url访问，发现一直加载，然后5秒后加载完毕，开始对于这个请求参数进行刨析

URL解码(如果存在==这种符合，会进行url编码)

`JyBBTkQgKFNFTEVDVCAxNjQ5IEZST00gKFNFTEVDVChTTEVFUCg1KSkpR2NhRykgQU5EICdFQURnJz0nRUFEZw%3D%3D`

|

`JyBBTkQgKFNFTEVDVCAxNjQ5IEZST00gKFNFTEVDVChTTEVFUCg1KSkpR2NhRykgQU5EICdFQURnJz0nRUFEZw==`

|(BASE64解码)

`' AND (SELECT 1649 FROM (SELECT(SLEEP(5)))GcaG) AND 'EADg'='EADg`

这就很明显的看到是sql注入中的时间盲注了，尤其看到`SLEEP(5)`就明白为什么在访问后加载5秒了，因为这个页面只会响应true或false，那就只能用盲注来判断数据到底是否存在

因为是时间注入，且被编码了两层(base64->url)，所以需要用脚本来进行解码并猜测用户获取了哪些数据

### 思路

正常情况下，使用工具或者人为手动注入，需要先获取数据库->然后表名->字段->数据，所以就需要写个脚本

URL解码->BASE64解码->判断注入了哪些库->判断注入了哪些表->判断注入了哪些字段->最终获取了哪些数据？

再因为是时间盲注的原因，这里推荐`access.log`结合流量包的方式去写脚本

假如你编写脚本底子差或者没思路怎么办？不要忘了我们现在有AI大模型啊，只要会表达，把需求说明白就可以了，这个表达的语言叫做"提示词"

```text
我现在网站受到了攻击，初步判断受到了SQL注入且为时间盲注，证明如下{access.log}你来分析一下，然后通过响应内容的大小
帮助我写一个python脚本，首先你需要提取出search.php所有请求，提取出来之后，因为有些特殊字符被url编码了，你需要先进行解码
然后再对url解码后的内容进行base64解码会得到如下的时间盲注payload
' AND (SELECT 1649 FROM (SELECT(SLEEP(5)))GcaG) AND 'EADg'='EADg
你需要依次进行提取出acccess.log中延时成功的请求，然后给我将攻击者获取成功的数据库、表、字段、数据依次转换给我
你需要观察哪些响应延时成功了，然后再写脚本，记住请求顺序不能乱，最终输出结果以形似与sqlmap注入完毕后的表格样式给到我
```

让大模型写脚本不是一次性成功的，需要不断的去写，执行测试，然后把响应和错误点告诉他，以及日志给他，让他继续优化，这是一个过程，比如下面是让他结合log文件提取出result1.pcap流量中的请求，最终提取出注入成功结果然后转为json文件的脚本

```python
import re
import json
import base64
import urllib.parse
from collections import defaultdict
from datetime import datetime

def decode_payload(encoded_payload):
    """解码payload（URL解码 + Base64解码）"""
    try:
        url_decoded = urllib.parse.unquote(encoded_payload)
        try:
            base64_decoded = base64.b64decode(url_decoded).decode('utf-8')
            return base64_decoded
        except:
            return url_decoded
    except Exception as e:
        return encoded_payload
```
