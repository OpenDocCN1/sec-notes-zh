
# [原创] 第一次尝试粗浅的逆向破解golang程序

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1684092
- 抓取时间: 2026-11-01T23:10:12

## 前言

这几天在我无聊逛 `github` 的时候，发现了一个写着能导出微信数据库密码和聊天记录的仓库，就想下载运行看看效果如何。

![](https://img1.imgtp.com/2022/09/04/TFLDmIkr.png)

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[href="forum.php?mod=attachment&aid=MjU1MTUwN3xmNWIwOGMxZXwxNzkzNTc0NjEwfDE2ODA4NnwxNjg0MDky" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1MTUwN3xmNWIwOGMxZXwxNzkzNTc0NjEwfDE2ODA4NnwxNjg0MDky)

(2.05 MB, 下载次数: 37)

2022-9-4 17:36 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB

然而启动后且报错 `pass error`，显然是需要个正确密码。

![](https://img1.imgtp.com/2022/09/04/VjRr64qi.png)

之后试用了下确实能用，但我很好奇 dump 个微信密钥就这为什么还得需要传个密码才能启动，而且 git 上也没源码只放了 release，第一想法以为可能是暗藏了根据电脑时间来做运行限制之类的暗桩，比如超过某段时间后算法会计算要提供新的密码才能用，也可能是这程序本来是在 shell 上用的，为了防止被蓝队拿了然后去分析或者二次利用。（其实我也经常想给自己的程序加个启动密码。另外 `README` 本有写要传密码，刚开始没注意））

所以想看看是为什么要这么搞，能不能把密码限制给去掉。

## 初步分析

用 `exeinfope` 查了目标程序，无壳，64 位程序，用 `go` 语言写的。可以直接上 `IDA` 分析，或者用 `x64dbg`。

![](https://img1.imgtp.com/2022/09/04/VkKtm3eW.png)

之前没分析过编译后的 `go` 程序，大概知道 `golang` 是静态链接的，封装层次高，有多返回值特点，`gc`、协程实现，标准库第三方库等等都嵌入了，文件体积贼大。

载入 `IDA`，能看到一大坨的函数，不可能一个个都去看的。后面干啃去调试时，函数间的跳转比较饶，加上有异步，容易跑飞。

![](https://img1.imgtp.com/2022/09/04/VTTG1afW.png)

像这种应该就是被 `strip` 掉了符号，`go build` 本身有命令可以去掉

```bash
go build ldflags="-s -w" main.go
```

也没找到 `.gopclntab` 区段

### 开局，搜索字符串

按老套路，最简单的搜索字符串，定位关键代码

![](https://img1.imgtp.com/2022/09/04/uvHSYTrw.png)

然而可惜的是，这并没有什么用，都找不到。这时猜测要么就是故意加密了，要么就是 `go` 字符串结构比较特殊。只能采取其他方法。

### 还原符号

由于启动密码的判断都写在程序的开头，如果能找到入口点，分析起来就简单不少，虽然这个程序没有调试符号，但一般情况就算 `go` 编译时去掉了，也还会包含一定量的信息，能还原出一些函数符号和源码文件路径。

我借助了 `IDAPython` 运行 `github.com/0xjiayu/go_parser` 和 `IDAGolangHelper` 那些脚本来解析，希望能方便后续分析。

然而可惜的是，不是有报错就是解析不全面，字符串也没解析出来，没有看到 `main_init` `main_main` 入口函数，当然可能是 `go` 版本较高，`pclntab` 结构有变化，一些工具还不支持。

不过至少还原了部分 `runtime`，这总比干啃好。

![](https://img1.imgtp.com/2022/09/04/jWgWnbRn.png)

有些奇怪的是，解析之后不少函数名都是随机字符串。我也不确定这程序是经过了混淆保护了，还是 `golang` 本身有其他什么设置就能导致如此。

![](https://img1.imgtp.com/2022/09/04/ddI0fmiA.png)

额外说一下：有时候 `go` 版本号可以在 `ida` `string` 列表找到。（虽然这个找不到）

![](https://img1.imgtp.com/2022/09/04/63scSIy7.png)

`go` 的汇编代码底部会有个 `jmp` 到函数头，用这个特征可以识别出函数体，`IDA` 有时识别不出来，手动按 `c` 可以强制分析成代码

![](https://img1.imgtp.com/2022/09/04/EtiJgD5X.png)

`IDA` `rebase` 一下基址或导出 `map` 方便在调试时候对着看

![](https://img1.imgtp.com/2022/09/04/z60o9nDv.png)

### 动态调试 API 断点回溯

接着尝试从 `API` 下断入手，逼近关键代码，首先很容易想到获取启动参数的 `API` 会用到 `GetCommandLineW`，`go` 用 `fmt.println` 写出到控制台的 `API` 则可能是 `WriteFile`，`WriteConsoleW`（64 位一般直接断 `W`），载入程序后下断点运行。

![](https://img1.imgtp.com/2022/09/04/bp79epEC.png)

返回用户模块空间，接下来每一次返回提前在 `ret` 处下好断点，很容易跟飞

![](https://img1.imgtp.com/2022/09/04/1zjtt0oQ.png)

![](https://img1.imgtp.com/2022/09/04/nlJ8rcHJ.png)

中间遇到循环不用管，一直专注回溯

![](https://img1.imgtp.com/2022/09/04/mHpavuvK.png)

同时注意堆栈字符串的变化，`println` 是分成时间和 `pass error` 两段。

看到这里再往后一两次 `ret` 差不多就到了。

![](https://img1.imgtp.com/2022/09/04/Z7kTKF6G.png)

这地方就差不多到了，在函数头下断点，，如果不确定就多下 `F2`，重载运行，输入参数。

`0000000000DE5F60` | `4C:8D6424 E8` | `lea r12,qword ptr ss:[rsp-18]`

![](https://img1.imgtp.com/2022/09/04/o54ZgK4w.png)

这种方法还是比较绕的，尤其是没有符号和不熟悉的情况，下面还有其他方法。

### 跟踪数据读写

命令行参数值始终是要判断的，用 `go` 写个 `demo`，大概写法就是：

```go
package main

import (
        "fmt"
        "os"
)

func main() {
        if len(os.Args) > 2 {
                if os.Args[1] == "123456" {
                        fmt.Printf("os.Args[1]: %v\n", os.Args[1])
                } else {
                        fmt.Println("pass error")
                }
        } else {
                fmt.Println("error 1")
        }
}
```

在没有符号的情况下，可以通过对比自己写的 `demo` 汇编结构和 `go` 源码来人肉识别某些库函数。

```go
// These routines end in 'ln', do not take a format string,
// always add spaces between operands, and add a newline
// after the last operand.

// Fprintln formats using the default formats for its operands and writes to w.
// Spaces are always added between operands and a newline is appended.
// It returns the number of bytes written and any write error encountered.
func Fprintln(w io.Writer, a ...any) (n int, err error) {
        p := newPrinter()
        p.doPrintln(a)
        n, err = w.Write(p.buf)
        p.free()
        return
}
...
......
```

`os.Args` 是一个全局切面变量，找到这个也可以定位。`os.Args` 是在 `os.init` 中初始化的。

![](https://img1.imgtp.com/2022/09/04/swtw9pyG.png)

启动后对 `GetCommandLineW` 下段，单步往下，对 `rax` 的第一个参数下硬件访问断点

![](https://img1.imgtp.com/2022/09/04/k7tSFa2A.png)

`F9` 运行来到这里，看起来只是在计算长度，继续运行

![](https://img1.imgtp.com/2022/09/04/THwHUB19.png)

循环取值到 `edi`，然后赋值转移到 `[rax+rbx*4]`，那也在这里下个硬件断点

![](https://img1.imgtp.com/2022/09/04/PSbRJXKc.png)

接着运行

![](https://img1.imgtp.com/2022/09/04/JFwQADeB.png)

这些地方大概是在将 `unicode` 的 `string` 转换编码，单步往下走，末尾 `ret` 下断，`eax` 有有新生成的字符串地址

![](https://img1.imgtp.com/2022/09/04/2CxWv5mt.png)

继续单步下去，观察 `eax` 用到哪

![](https://img1.imgtp.com/2022/09/04/5YKmwxav.png)

可以发现赋给了一个值

![](https://img1.imgtp.com/2022/09/04/ypFg8ZQF.png)

![](https://img1.imgtp.com/2022/09/04/vO7SQQ0f.png)

`mov qword ptr ds:[0x0000000000F0F300], rax` 这里就是 `os.Args`，这应该是个通用特征，可以通过搜索常量引用或者硬件下断都可以定位到校验方法

`0000000000DE5F60` | `4C:8D6424 E8` | `lea r12,qword ptr ss:[rsp-18]`

![](https://img1.imgtp.com/2022/09/04/t7tDCauB.png)

### patch 代码

很简单了，有字符串比较，跟一遍，只要 `nop` 掉两处跳转即可

![](https://img1.imgtp.com/2022/09/04/5H9mkyJe.png)

另外字符串是用这些方法移位解密的，怪不得搜不到。

```
0000000000DE58C0  | 49:3B66 10               | cmp rsp,qword ptr ds:[r14+10]           |
0000000000DE58C4  | 0F86 EE010000            | jbe target.DE5AB8                       |
0000000000DE58CA  | 48:83EC 70               | sub rsp,70                              |
0000000000DE58CE  | 48:896C24 68             | mov qword ptr ss:[rsp+68],rbp           | [rsp+68]:"袩\r"
0000000000DE58D3  | 48:8D6C24 68             | lea rbp,qword ptr ss:[rsp+68]           | [rsp+68]:"袩\r"
0000000000DE58D8  | 48:BA E4E18554A2A332C3   | mov rdx,C332A3A25485E1E4                |
0000000000DE58E2  | 48:895424 54             | mov qword ptr ss:[rsp+54],rdx           |
0000000000DE58E7  | 48:BA A2A332C3BFD17EA1   | mov rdx,A17ED1BFC332A3A2                |
0000000000DE58F1  | 48:895424 58             | mov qword ptr ss:[rsp+58],rdx           |
0000000000DE58F6  | 48:BA C5CDCF5350E17134   | mov rdx,3471E15053CFCDC5                |
0000000000DE5900  | 48:895424 60             | mov qword ptr ss:[rsp+60],rdx           |
0000000000DE5905  | 0FB65424 55              | movzx edx,byte ptr ss:[rsp+55]          |
0000000000DE590A  | 885424 53                | mov byte ptr ss:[rsp+53],dl             |
0000000000DE5914  | 44:884424 52             | mov byte ptr ss:[rsp+52],r8b            |
0000000000DE5919  | 44:0FB64424 54           | movzx r9d,byte ptr ss:[rsp+54]          |
0000000000DE591F  | 44:884C24 51             | mov byte ptr ss:[rsp+51],r9b            |
0000000000DE5924  | 44:0FB65424 56           | movzx r10d,byte ptr ss:[rsp+56]         |
0000000000DE592A  | 44:885424 50             | mov byte ptr ss:[rsp+50],r10b           |
0000000000DE592F  | 44:0FB65C24 5B           | movzx r11d,byte ptr ss:[rsp+5B]         |
0000000000DE5935  | 44:885C24 4F             | mov byte ptr ss:[rsp+4F],r11b           |
0000000000DE593A  | 44:0FB66424 64           | movzx r12d,byte ptr ss:[rsp+64]         |
0000000000DE5940  | 44:886424 4E             | mov byte ptr ss:[rsp+4E],r12b           |
0000000000DE5945  | 44:0FB66C24 57           | movzx r13d,byte ptr ss:[rsp+57]         |
0000000000DE594B  | 44:886C24 4D             | mov byte ptr ss:[rsp+4D],r13b           |
0000000000DE5950  | 44:0FB67C24 65           | movzx r15d,byte ptr ss:[rsp+65]         |
0000000000DE5956  | 44:887C24 4C             | mov byte ptr ss:[rsp+4C],r15b           |
0000000000DE595B  | 44:0FB66C24 58           | movzx r13d,byte ptr ss:[rsp+58]         |
```
