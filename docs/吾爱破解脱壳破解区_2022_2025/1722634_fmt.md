
# [CTF] pwnable.kr题目笔记

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1722634
- 抓取时间: 2026-11-01T23:04:20

## pwnable.kr笔记

本人是新学pwn的一个小菜鸟，最近在边学习边做[pwnable.kr](https://www.52pojie.cn/pwnable.kr)网站中的习题.现将一些菜鸟的解题方法与心得整理如下,欢迎各位大佬批评指正,谢谢!!!

本文环境:

- Windows11家庭中文版,版本21H2(OS 内部版本 22000.978)
- WSL 2
- https://www.microsoft.com/store/apps/9N9TNGVNDL3Q

### 1. Toddler's Bottle

#### 1.1 fd

![](https://attach.52pojie.cn/forum/202212/05/001913yoviug2u2huuebug.png)

2022-12-04-22-58-15.png (70.47 KB, 下载次数: 1)

第一题是一道关于Linux系统中的文件描述符(File Descriptor)的比较简单的题目.

> 

预备知识: 文件描述符是系统用于唯一标识文件的一串无符号整型数字,它们由小到大依次分配.其中,每个进程开始时都会有三个打开的文件,为:标准输入(描述符0),标准输出(描述符1),标准错误(描述符2).在中定义了成了常量`STDIN_FILENO`,`STDOUT_FILENO`和`STDERR_FILENO`.

首先我们使用题目给的命令进入远程服务器,括号内的guest为登录密码(注:输入密码时不可见):

```
ssh fd@pwnable.kr -p2222
# ssh(Secure Shell)是登录远程服务器的命令
# 其中,fd@pwnable.kr表示远程服务器的域名,-p2222表示端口(port)2222
# 使用`exit`命令退出连接
# 相关参数及用法可参考https://cloud.tencent.com/developer/article/1594881
```

之后我们就得到这样的界面

![](https://attach.52pojie.cn/forum/202212/05/002012dmpqn3equpesqqjq.png)

2022-12-04-23-12-48.png (65.99 KB, 下载次数: 2)

使用命令`ls`查看目录下文件或使用`ls -la`查看目录下文件以及相关权限

![](https://attach.52pojie.cn/forum/202212/05/002021acbgz1t38zwgt1x1.png)

2022-12-04-23-14-27.png (56.4 KB, 下载次数: 4)

我们发现目标文件`fd`,`fd.c`和`flag`.但是在这个过程中,我们发现该服务器访问较慢,于是使用如下命令下载本地.但在此之前,为了方便管理,可先使用命令`mkdir namefile`创建以namefile为名文件夹,再进入该文件夹,将文件下载到该文件夹下:

```
mkdir fd
# 本文创建文件夹名为fd
cd fd
# 进入fd文件夹中
scp -P2222 fd@pwnable.kr:fd .
scp -P2222 fd@pwnable.kr:fd.c .
# scp(Secure Copy)用于复制文件和目录
# 其中,fd@pwnable.kr表示远程服务器的域名,-P2222表示端口2222(注意是大写P),
# .表示本地当前目录
# 相关参数及用法可参考https://www.runoob.com/linux/linux-comm-scp.html
```

![](https://attach.52pojie.cn/forum/202212/05/002026ngoh0ad838idl82o.png)

2022-12-04-23-28-19.png (82.66 KB, 下载次数: 3)

随后我们可以使用`vim`命令查看`fd.c`的内容(注释为本人后加之):

```
#include 
#include 
#include 
char buf[32];
// argc表示文件参数的的个数,argv表示文件参数的地址,envp表示环境变量
int main(int argc, char* argv[], char* envp[]){
    // 判断是否有参数,在命令行,可执行文件的文件名本身算作一个参数,其地址为argv[0],所以argc始终大于等于1
    if(argc
    // 它将参数所指的字符串转化成int整型
    // 可参考: https://www.runoob.com/cprogramming/c-function-atoi.html
    int fd = atoi( argv[1] ) - 0x1234;
    int len = 0;
    // read()函数为系统调用函数,它从文件描述符为fd的拷贝32个字符到buf中
    // 返回值为真正拷贝的字符的个数.这是EOF的存在导致的,因为它可能没有那么多的字符供拷贝
    // 可参考: https://blog.csdn.net/zjhkobe/article/details/6633446
    len = read(fd, buf, 32);
    // strcmp用于比较两个字符的大小,如果两字符相等,则返回0
    if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

由上述代码及注释可知,本题需要传递一个合适参数,使得`fd`标识的文件中的32位字符为`LETMEWIN\n`.根据预备知识可知,我们可以使用`fd=0`的标准输入,来输入所需的字符.我们的输入值应为十进制,`0x1234`的十进制为4660.如下图所示:

![](https://attach.52pojie.cn/forum/202212/05/002029n5jzuouzmq1oiiib.png)

2022-12-05-00-12-43.png (110.96 KB, 下载次数: 5)
