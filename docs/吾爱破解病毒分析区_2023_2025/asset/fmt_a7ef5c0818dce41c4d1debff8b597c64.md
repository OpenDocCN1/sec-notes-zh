
# [PC样本分析] 记一次银狐恶意样本 `WindowEvent.exe` 分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=2075186
- 抓取时间：2026-11-01T23:43:24

## 背景

该样本为前一段时间在52破解论坛的病毒求助区找到的恶意样本，通过沙箱+手动调试分析学习该银狐样本的免杀技术和恶意代码。原帖：[https://www.52pojie.cn/thread-2063616-1-1.html](https://www.52pojie.cn/thread-2063616-1-1.html)

## 样本概述

木马样本通过反射加载Shellcode外联C2服务器。

![1.png](https://attach.52pojie.cn/forum/202511/24/105236wxb46g8hxap6dx44.png)

1.png (61.35 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM1MXw4OGE0MTRlZHwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:52 上传

![2.png](https://attach.52pojie.cn/forum/202511/24/105251os5jugj4gdvw3fhx.png)

2.png (58.16 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM1NXw4ZGNjNzIyYXwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:52 上传

## 分析

### 1、加解密部分

入口点

![3.png](https://attach.52pojie.cn/forum/202511/24/105310drpdwkfbxrvrp8bb.png)

3.png (33.99 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM1Nnw2YTc0NjZmMXwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:53 上传

该样本无导入表和敏感字符串，经过分析样本调用`sub_D802F5`函数解析需要的WinAPI。该函数内部调用`sub_D80488`函数和`sub_D80271`函数。

静态分析`sub_D80488`函数，`a1`参数用于存放`a2`数组异或`0x6A`解密的数据，`a2`参数存放需要解密数据的内存地址。

![4.png](https://attach.52pojie.cn/forum/202511/24/105720zxtetet7ldz3dlm4.png)

4.png (26.2 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2MHw5OWUwMDMwMXwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

![5.png](https://attach.52pojie.cn/forum/202511/24/105722bzqs6lpqwx325acj.png)

5.png (73.78 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2M3xiN2M4NzA0M3wxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

静态分析`sub_D80271`函数，参数`a1`为载入的`Kernel32.dll`的基地址，参数`a2`为需要解析的函数的函数名字符串。

![6.png](https://attach.52pojie.cn/forum/202511/24/105724mrmzdjawfxfba1an.png)

6.png (66.97 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2NHw3NzMxZmQyZnwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

解析结果如下：

![7.png](https://attach.52pojie.cn/forum/202511/24/105726glr3gqzllqsaqfrv.png)

7.png (171.49 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2NXwxNmZjZmExZnwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

样本调用`sub_D804B8`函数来处理模块路径。解密字符串`"Kernel32.dll"`和`"GetModuleFileNameA"`，通过函数`sub_D80271`解析`"GetModuleFileNameA"`真实地址，调用`GetMoudleFileNameA`返回当前模块路径并对路径进行处理。

![8.png](https://attach.52pojie.cn/forum/202511/24/105728f1oals3h7zhxo2f7.png)

8.png (75.99 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2NnwzNWFkMjdmZXwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

`sub_D80660`函数是反射加载并执行shellcode的部分，使用`VirtualAlloc`分配内存用于存放加密Shellcode，使用异或解密Shellcode，调用`VirtualProtect`修改内存的权限，内存执行解密的Shellcode。

![9.png](https://attach.52pojie.cn/forum/202511/24/105730khwpdfhhiy19iuyi.png)

9.png (62.81 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2N3wwN2Y0NjUwY3wxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

### 2、创建计划任务持久化部分

`sub_D80693`函数用于创建计划任务持久化攻击，调用`CreateProcess`创建进程执行`"schtasks /run /tn WinSafe"`，创建`"WinSafe"`计划任务，实际上计划任务没有被成功创建。

![10.png](https://attach.52pojie.cn/forum/202511/24/105733ugzcf22f2mtoggcb.png)

10.png (77.04 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2OHxmNzAyMGZlMnwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

通过执行计划任务查询命令和手动创建`"WinSafe"`计划任务，找寻创建失败原因（Win10没有`"WinSafe"`模块）。

```
schtasks /query
schtasks /run /tn WinSafe #尝试自己进行创建
```

![11.png](https://attach.52pojie.cn/forum/202511/24/105735zejlsjkf5j98gsea.png)

11.png (84.04 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM2OXxiZjNmZDQ1YXwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

### 3、shellcode部分

dump对应shellcode，dump的shellcode可以复制前六个字节的16进制数据去查询，一般被研究过的样本家族的shellcode会有记录，可以结合前人的分析报告进行快速分析。

dump的shellcode可以使用`Winhex`等工具查看程序的数据（也可以使用`strings` / `010 editor`等工具进行查看），这里使用`Winhex`查看样本发现存在明显的IP地址和域名（初步判断为外联的C2服务器IP地址和域名）。

```
156.234.119.138
syumineyt.top
```

![12.png](https://attach.52pojie.cn/forum/202511/24/105737a6gbbbjjszgkygcb.png)

12.png (206.74 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM3MHwzMDE4MDg0NnwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

Shellcode通过遍历模块加载的`FullName`是否为`Kernel32.dll`来确定`Kernel32.dll`基地址，代码通过将获取的模块名称Ascii码进行相加最后和`Kernel32.dll`的模块名的Ascii码和进行比较来判断当前模块。

![13.png](https://attach.52pojie.cn/forum/202511/24/105739tw2m0llqq2bhb62z.png)

13.png (100.77 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM3MXxiNDAwOTYzZnwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

![14.png](https://attach.52pojie.cn/forum/202511/24/105741s8toy8snmrsq1sez.png)

14.png (47.18 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM3Mnw1NmJlNjU2MHwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

样本对调用的WinAPI进行了混淆，调用`sub_747`函数根据`APIHash`来获取对应API函数的真实地址，以便躲避杀软的敏感函数查杀。以下是样本需要调用的WinAPI以及`APIHash`和WinAPI的对应关系。

![15.png](https://attach.52pojie.cn/forum/202511/24/105744esrdqddlzrrcf5dd.png)

15.png (78.32 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgxNjM3M3w1M2RlMTVkYXwxNzkzNTc2NjAyfDE2ODA4NnwyMDc1MTg2&nothumb=yes)
2025-11-24 10:57 上传

```
1AB9B854h->GetProcessAddress
7F201F78h->LoadLibrary
5E893462h->VirtualAlloc
6488073h->VirtualFree
0DB579CBh->RtlZeroMemory
1518E9C0h->RtlMoveMemory
33522634h->WSAStartup
26402D9Fh->socket
41539501h->getaddrinfo
785E7DD7h->FreeAddrInfoW
313A84C8h->htons
92B50DAh->connect
0F8387DCh->send
0F6134B2h->recv
14AC161Bh->closesocket
4FF42CCFh->WSACleanup
```

调试过程中获取反转的字符串，使用`python`脚本进行反转发现疑似配置信息。

`eax=00BE086B L"|0:cf|1:hb|0:pj|8 .7 .5202:zb|0.1:bb|认默:zf|1:lc|1:dd|1:3t|0808:3o|pot.tyegnimuys:3p|1:2t|08:2o|pot.tyenimuys:2p|1:1t|344:1o|831.911.432.651:1p|"`将反转的参数地址引入。
