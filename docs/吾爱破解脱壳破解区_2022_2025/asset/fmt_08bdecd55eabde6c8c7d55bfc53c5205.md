
[href="forum.php?mod=attachment&aid=MjYxNzM4MHwyYjY2NWQwM3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4MHwyYjY2NWQwM3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

代码参考：`idapython/deTvm.py` . `tvmToAsm.xxx_handle()`

## jcc

上面我们已经获取了 `jcc` 的类型，和它的两个跳转地址，虽然都是 `V_RIP`，

我们对这两个 `V_RIP` 往下找第一个 `workingTrace`，它所在的 `tvmToAsm` 就是对应的跳转地址，

我们可以给这个地址的的 `Asm` 打上跳转目的地标签，例如：

![](https://attach.52pojie.cn/forum/202305/26/010706hs96kp6pc99ibsl6.png)

36 举例用jcc.png (25.04 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxNzM4MXw1NTlhODdjYXwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4MXw1NTlhODdjYXwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

可以翻译为：

```
......
je lable1
jmp label2
......
label2:        ;(0x14003c0e6往下找第一个tvmToAsm)
mov rax,1      ;只是举个例子
......
label1:        ;(0x14003c13b往下找第一个tvmToAsm)
mov rcx,2      ;只是举个例子
......
```

参考代码：`idapython/deTvm.py` . `tvmToAsm.jcc_handle()`

## ASM 2 HEX

这部分略，比较简单，就是通过 `keystone` 库函数将 `Asm` 编译成十六进制机器码，然后再创建一个段，把内存写进去。详情参考：`idapython/deTvm.py` . `tvmToAsmAll.WriteHex()` 和 `main0` 函数。

值得一提的是，我发现了 `keystone` 的一个 `bug`，你们可以试一下：

```
import keystone

ASM2HEX = keystone.Ks(keystone.KS_ARCH_X86, keystone.KS_MODE_64)

asm = """

mov rcx,qword ptr ds:[0x14000d250]

"""

byte,con = ASM2HEX.asm(asm,addr=0x1400ef00a)

for by in byte:

    print("%02x"%by,end="")
```

输出的结果是：

```
48 8b 0d 50 d2 00 40
```

将其转换成 `ASM`，是

```
1400ef00a  mov rcx, qword ptr [rip + 0x4000d250]
```

这明显是错误的，0x1400ef00a+0x7+0x4000d250 != 0x14000d250

```
asm = """
mov qword ptr ds:[0x14000d250],rcx
"""
byte,con = ASM2HEX.asm(asm,addr=0x1400ef00a)
```

就能输出正确的字节码。

所以我采用的方法是一句一句将 `Asm` 转换成 `HEX`，如果遇到 `mov reg,qword ptr[xxx]`，就把格式改成 `mov reg,qword ptr[rip + yyy]`，详情参考代码：`idapython/deTvm.py` . `Asm.AsmToHex()`

## deTvm.py脚本玩法

`main0` 是对全部 `ida` 识别的函数进行特征分析，如果符合 `tvm` 函数特征，就对它进行还原。

可以算是一键还原全部 `tvm` 函数了，有可能有些 `tvm` 函数不符合特征，你也可以手动添加还原函数。

例如：

你知道一个函数 `0x140001250` 它是被 `vm` 的，那么你这么写，脚本就会自动特征识别 `V_RIP`。

```
testTrace = traceTask(0x140001250,tvm0base)          #tvm0base是tvm0段的起始地址

testTrace.track(0)                                                                #开始跟踪 得到traceCode

testTrace.traceOut()                                                        #输出原始traceCode
```

如果你这个被 `vm` 的函数不符合我写的特征，但它确实是 `tvm` 的函数，那么可以这么写，自己设置 `V_RIP`：

```
#上一个例子的 函数 0x140001250 它的 V_RIP 就是 0x140059dc2
testTrace = traceTask(0,tvm0base)
testTrace.VStart = 0x140059dc2                #自己找这个vm函数的起始地址 例如V_RIP = 0x140059dc2 
testTrace.track(0)                                        #开始跟踪 得到traceCode
testTrace.traceOut()                                #输出原始traceCode
```

`traceOut(0)` 输出的结果如下：（基本没做处理）

![](https://attach.52pojie.cn/forum/202305/26/010708jr6gatonry6d6d2r.png)

37 traceOut测试.png (147.1 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxNzM4Mnw5YmQwNmUwY3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4Mnw5YmQwNmUwY3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

如果想看 对标记 `working` 的 `traceCode` 进行变量溯源的结果，你可以这么写：

```
testTrace = traceTask(0x140001250,tvm0base)          #tvm0base是tvm0段的起始地址

testTrace.track(0)                                                                #开始跟踪 得到traceCode

testTrace.VRegRecord(True)                                                #如果是False就是不使用标记（上文说过）

testTrace.tvmToAsmAll.printAll()                                #输出
```

输出：

![](https://attach.52pojie.cn/forum/202305/26/010710hc6f6hplauahppcf.png)

38 VRegRecord.png (79.63 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxNzM4M3w5NjZiNWE2ZHwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4M3w5NjZiNWE2ZHwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

如果想进一步的进行变量传播优化还有 `push`、`pop` 优化，可以这么写：

```
testTrace = traceTask(0x140001250,tvm0base)          #tvm0base是tvm0段的起始地址
testTrace.track(0)                                                                #开始跟踪 得到traceCode
testTrace.VRegRecord(True)                                                #如果是False就是不使用标记（上文说过）
testTrace.tvmToAsmAll.optimizeAll()                                #变量传播优化,push、pop优化
testTrace.tvmToAsmAll.printAll()                                #输出
```

输出：

![](https://attach.52pojie.cn/forum/202305/26/010712xdytd55mz55xux7u.png)

39 传播优化.png (52 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjYxNzM4NHw3ZjY1ZDMzM3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4NHw3ZjY1ZDMzM3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

如果想看还原成 `ASM` 是什么样的，可以这样写：

```
testTrace = traceTask(0x140001250,tvm0base)          #tvm0base是tvm0段的起始地址

testTrace.track(0)                                                                #开始跟踪 得到traceCode

testTrace.VRegRecord(True)                                                #如果是False就是不使用标记（上文说过）

testTrace.tvmToAsmAll.optimizeAll()                                #变量传播优化,push、pop优化

testTrace.tvmToAsmAll.AllTvmAsmToAsm()                         #转换成ASM  注意，一定要VRegRecord + optimizeAll 后才可以调用

testTrace.tvmToAsmAll.printAsmAll()                                #输出ASM
```

输出：

![](https://attach.52pojie.cn/forum/202305/26/010714h153oosp1glos4a3.png)

40 输出ASM.png (13.56 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxNzM4NXwyYjUxZGQ3MHwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4NXwyYjUxZGQ3MHwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

如果想看 `tvmToAsm` 和 `Asm` 对应起来的输出，可以这样写：

```
testTrace = traceTask(0x140001250,tvm0base)          #tvm0base是tvm0段的起始地址
testTrace.track(0)                                                                #开始跟踪 得到traceCode
testTrace.VRegRecord(True)                                                #如果是False就是不使用标记（上文说过）
testTrace.tvmToAsmAll.optimizeAll()                                #变量传播优化,push、pop优化
testTrace.tvmToAsmAll.AllTvmAsmToAsm()                         #转换成ASM

tvmToAsm_P = testTrace.tvmToAsmAll.tvmToAsmHead #结构为tvmToAsm
while (tvmToAsm_P != None):
    tvmToAsm_P.printAsm()                       #输出Asm
    tvmToAsm_P.print()                          #输出traceCodeAll
    print("")                                   #隔开
    tvmToAsm_P = tvmToAsm_P.BLink               #下一个
```

输出：

![](https://attach.52pojie.cn/forum/202305/26/010717a1mt17e87trawae8.png)

41 对应输出.png (62.2 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxNzM4NnwzNTBiOWYxMHwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4NnwzNTBiOWYxMHwxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

## 还原例子

总所周知 `ACE-BASE.sys` 的 `DriverUnload` 函数是被 `vm` 了的，那么我们就用它来看看还原效果：

![](https://attach.52pojie.cn/forum/202305/26/010719k899jayipyrbtji2.png)

42 ACE-BASE测试.png (181.53 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxNzM4N3w4NGQ5ZWYxY3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxNzM4N3w4NGQ5ZWYxY3wxNzkzNTczNTM1fDE2ODA4NnwxNzg5ODky&nothumb=yes)

2023-5-26 01:07 上传

不错，很符合我对 `DriverUnload` 的想象。

左边 命名为 `icxxx` 的函数均为还原成功的函数。

还原脚本项目地址：[//pan.baidu.com/s/1gjmYQHXELELJwqhb-HJ9iA?pwd=ICEY" target="_blank" rel="noopener noreferrer nofollow"](https://pan.baidu.com/s/1gjmYQHXELELJwqhb-HJ9iA?pwd=ICEY:rel="nofollow noopener" href="https)

## 一些补充

这个脚本只适用于这个版本的 `tvm` (`ACE` 用的版本)。

新一点的 `tvm`，虽然用的是同一套虚拟化指令集，但它会对整数进行加密，读取时进行简单的 `xor` 解密，并且新增了一些 虚拟指令。

而且 进入虚拟机的特征也有点不一样，不过对脚本进行简单的修改即可兼容。

### 一些闲话

`ACE` 你快用新版的 `tvm`，你现在用的版本是有 `bug` 的(`JBE` `JNA` 的模拟是错的)。

