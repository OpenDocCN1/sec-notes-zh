
# [原创] 某静态代码分析工具（Understxxd）过验证

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1611757
- 抓取时间: 2026-11-01T23:31:18

## 背景介绍

`Understxxd`是一个静态代码分析工具，用来生成类图和流程图很方便和强大，用来分析没有详细文档的大型的源代码库相当顺手。
网上也有破解注册的版本，但是在断网情况下会提示无法验证本地证书，导致弹窗验证无法进入主界面。

![](https://attach.52pojie.cn/forum/202203/27/155344pw53w6ssnvl545c3.png)

`2.png` (17.24 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjUwODQ1NXwwMTNmMmI1YnwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwODQ1NXwwMTNmMmI1YnwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes)

2022-3-27 15:53 上传

有遇到同样问题的可以参考一下如下步骤。

## 工具

`x64dbg`

## 过程

### 1. 首先查个壳

![](https://attach.52pojie.cn/forum/202203/27/161837nw9mvmeg8mp5gs7g.png)

`1.png` (31.01 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwODQ1NHwwYmNiZGYxYnwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwODQ1NHwwYmNiZGYxYnwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes)

2022-3-27 16:18 上传

ohh，不是有效的`PE`文件，直接拖入`OD`中提示可能不是一个32位`PE`文件，推测可能是64位的。
我也是第一次玩64位的破解，从论坛中下载`x64dbg`。

### 2. 载入分析

将执行文件拖入到 `x64dbg`中，直接右击搜索->查找所有模块->字符串，搜索关键词【`Unable to Validate`】。

![](https://attach.52pojie.cn/forum/202203/27/155346ojbhvi51jihbiqij.png)

`3.png` (27.57 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUwODQ1Nnw5ZjVmNTVmMHwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwODQ1Nnw5ZjVmNTVmMHwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes)

2022-3-27 15:53 上传

直接找到相关字符串了，看来是没有加密处理，那么接下来就简单了。

双击定位到代码，定位到【`0x7FF6CDD76C81`】，加断点。

让程序跑起来，点击按钮【`Use Legacy Licensing`】

![](https://attach.52pojie.cn/forum/202203/27/155347o348hk68i4h6fm8i.png)

`4.png` (22.92 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjUwODQ1N3w2NDdiYmE5Y3wxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwODQ1N3w2NDdiYmE5Y3wxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes)

2022-3-27 15:53 上传

成功截下断点。

![](https://attach.52pojie.cn/forum/202203/27/155350brh33nmtm9sh1jsm.png)

`5.png` (241.96 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwODQ1OHxhYjE0YzBiMHwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwODQ1OHxhYjE0YzBiMHwxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes)

2022-3-27 15:53 上传

`F8`反复调试，发现在弹窗代码上方不远处有个关键跳转能够跳过整段提示代码，可能是关键跳，因此爆破一下试试。

```
00007FF6CDD76BD7 | 8BCE                     | mov ecx,esi                                                           |
00007FF6CDD76BD9 | E8 E27B1202              | call                                     |
00007FF6CDD76BDE | 84C0                     | test al,al                                                            |
00007FF6CDD76BE0 | E9 07010000              | jmp understand.7FF6CDD76CEC               这里je改为jmp               | 可能是判断是否进行弹窗的，这里爆破后不进行验证了
00007FF6CDD76BE5 | 90                       | nop                                                                   |
00007FF6CDD76BE6 | 41:83FD 02               | cmp r13d,2                                                            |
00007FF6CDD76BEA | 0F85 FC000000            | jne understand.7FF6CDD76CEC                                           |
```

如图上方【`0x7FF6CDD76BE0`】将`je`改为`jmp`之后能够跳过弹窗。

改为`jmp`之后，继续调试代码，运行到退出。发现这个函数的返回值在`eax`寄存器中，返回值为1，未爆破前返回值为0。

![](https://attach.52pojie.cn/forum/202203/27/155352hll3n63ee11l1kll.png)

`6.png` (207.23 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUwODQ1OXw1MjEzMzEzY3wxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwODQ1OXw1MjEzMzEzY3wxNzkzNTc1ODc2fDE2ODA4NnwxNjExNzU3&nothumb=yes)

2022-3-27 15:53 上传

经过爆破，软件已经能够正常使用了。

下面两个红色箭头所指的`jmp`很奇怪，是往上跳转的，跟踪分析之后发现是进行重复验证的逻辑。当本地证书不通过的时候，程序会跳到上方，继续弹窗提示验证。爆破后直接跳过这两个`jmp`，跳过验证代码。

### 3. 保存文件

右击点击补丁，或者快捷键`Ctrl+P`，选择修补文件，另存文件即可保存。
