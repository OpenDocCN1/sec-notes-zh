
# 远程调试与栈溢出分析

## 启动 gdbserver 并连接目标

启用`gdb`开始调试进行分析，在本地运行的话太过麻烦，这个时候我们之前上传的`gdbserver`就起到作用了，在目标机器上

```
./gdbserver-static 0.0.0.0:8010 --attach $(netstat -anptl | grep 443 | awk '{print $7}' | cut -d'/' -f1 | grep -v "-")
```

在自己机上启用我们的`gdb`连接目标来进行调试

`target remote 192.168.201.169:8010`

## 设置断点监控 strcpy 溢出

连接到我们的目标，在`strcpy`函数返回后打一个断点,设置一个自动监控的脚本

```
break *$eip+100

commands
> echo === After strcpy === 
> x/32xw $esp
>info registers eip ebp esp 
>backtrace   
>continue
>end
```

设置完之后再次使用我们的`openconnect`打一次payload，此时可以发现溢出的数据。

![](https://attach.52pojie.cn/forum/202601/25/201916yueajjavzokxvkj3.png)

Pasted image 20260125200504.png (1021.29 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTUwMXxlNjk1MGFjYXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTUwMXxlNjk1MGFjYXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图21

2026-1-25 20:19 上传

## 栈布局与虚函数劫持

只有栈溢出还不够，还需要劫持控制流来构造完整的攻击链，在`sub_E4AD0`函数溢出发生后还有一个指针

![](https://attach.52pojie.cn/forum/202601/25/201913accwnr8wvykozyk1.png)

Pasted image 20260125200444.png (122.33 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTUwMHwxZTIyMjY0YnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTUwMHwxZTIyMjY0YnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图20

2026-1-25 20:19 上传

这里的栈空间分布大致为：

![](https://attach.52pojie.cn/forum/202601/25/201910jkcvuyk5n1cgshdk.png)

Pasted image 20260125200433.png (74.12 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ5OXw2NjU3NzVhNnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ5OXw2NjU3NzVhNnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图19

2026-1-25 20:19 上传

对应的汇编代码

```
 loc_E51C3:             ; CODE XREF: sub_E4AD0+6EC↑j

 mov     edx, [esp+0A0Ch+var_9E0] 

 //将栈上偏移量为[esp+0A0Ch+var_9E0]的内存地址处存储的值加载到edx寄存器中。

 mov     eax, [esp+0A0Ch+arg_0]

 //获取a1指针

 mov     eax, [eax]  

 //通过a1获取vtable地址

 mov     [esp+0A0Ch+src], edx

 //将edx寄存器中的值（之前从[esp+0A0Ch+var_9E0]加载的，可能是源字符串地址或长度）保存到栈上src的位置

 mov     edx, [esp+0A0Ch+arg_0]

 //再次将栈上第一个参数的值加载到edx寄存器中

 mov     [esp+0A0Ch+n], 2Eh ; '.' ; int

 // 将数值2Eh（即字符.）保存到栈上n的位置

 mov     [esp+0A0Ch+var_A0C], edx

 //将edx寄存器中的值（之前从[esp+0A0Ch+arg_0]加载的）保存到栈上var_A0C的位置。

 call    dword ptr [eax+48h]

 //调用了一个函数。函数的地址是从eax指向的结构体中的偏移48h处获取的。虚函数调用,我们要做的就是在这里劫持。
```

到这里我们只需要伪造虚函数表并将地址传递给`eax`让整个程序到这里执行我们的`ROP`来获取到`shell`。

>

## ROP 链构造

`ROP`主要是用来绕过内存保护机制，也就是我们在最开始说的数据不能在内存中直接执行。`ROP`就是通过串联程序中本来就存在的代码段（`gadget`），将这些代码段串联起来构造出我们要的执行流程。其中最主要的就是通过控制栈上的返回地址使其能够让其按照我们的想法来运行。

此时`esp`指向的区域为用户不可控区域，我们需要找到一个`gadget`，将`esp`指向可控的部分同时为了后续获取`shell`，`gadget`还要能够设置`ebx`寄存器，程序使用了大量的`lib`库,这里我们从所有引用的`lib`库中进行查找，最后在`libdsplibs.so`的`0x93849C`地址找到了合适的`gadget`。

![](https://attach.52pojie.cn/forum/202601/25/201906pvvi0niqv00ikv1b.png)

Pasted image 20260125200406.png (148.82 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ5N3w1ZTcyNTAwMnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ5N3w1ZTcyNTAwMnwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图17

2026-1-25 20:19 上传

![](https://attach.52pojie.cn/forum/202601/25/201904yz0pshihzfh0ep8t.png)

Pasted image 20260125200357.png (138.73 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ5Nnw5MDM3NjYzOHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ5Nnw5MDM3NjYzOHwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图16

2026-1-25 20:19 上传

后面就是正常构造`ROP`链来获取`shell`即可。

最终效果图：

![](https://attach.52pojie.cn/forum/202601/25/201901nlpzpuqqcxcczuic.png)

Pasted image 20260125200249.png (174.4 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjgyOTQ5NXw2Yzg4NjU1NXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyOTQ5NXw2Yzg4NjU1NXwxNzkzNTY5ODcyfDE2ODA4NnwyMDY3NjEx&nothumb=yes)

图15

2026-1-25 20:19 上传

### 3> 写在最后

==本文仅用于安全研究和漏洞分析目的，请勿用于任何非法用途。==

![](https://s3.bmp.ovh/imgs/2025/10/23/414ac77ac9c4e73c.jpg)

漏洞环境下载，感兴趣的坛友可以尝试一下[Shell] 纯文本查看 复制代码

```
通过网盘分享的文件：ps-ics-vmware-isa-v-22.7r2.3-b3431-package.zip
链接: https://pan.baidu.com/s/1LMBFudpWcAiWQu2hqqmcNg 提取码: 1111 
```
