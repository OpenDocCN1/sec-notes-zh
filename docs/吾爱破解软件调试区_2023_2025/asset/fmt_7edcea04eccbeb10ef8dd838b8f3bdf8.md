
2023-6-1 11:41 上传

## 3.3

然而直接走到 `D3D9.DLL+7C47` 肯定不会经过 `NtUserHwndQueryRedirectionInfo`

最简单的解决方式肯定是直接把 `je` `D3D9.DLL+7C47` `nop`

测试结果如下

![](https://attach.52pojie.cn/forum/202306/01/114244orzhirjlfnmmfmmw.png)

image.png (253.51 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxODg4NXxiMDY1ZmRiZnwxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODg4NXxiMDY1ZmRiZnwxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes)

有效果

2023-6-1 11:42 上传

很明显

改了之后加载驱动可以直接绘制上这个 基佬色 的方块

然而还原之后

![](https://attach.52pojie.cn/forum/202306/01/114522io5iisleo4u5yygg.png)

image.png (291.53 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxODg4NnxjNjYxNWQwZHwxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODg4NnxjNjYxNWQwZHwxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes)

无效果

2023-6-1 11:45 上传

绘制效果消失了

而且在 `USER32.HungWindowFromGhostWindow+20`处

下断点是不会命中的（这不是废话吗）

## 4.0

梳理流程

![](https://attach.52pojie.cn/forum/202306/01/114613w5xbru908xi5109r.png)

image.png (42.51 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxODg4N3w5MDU5NjU1M3wxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODg4N3w5MDU5NjU1M3wxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes)

流程图1

2023-6-1 11:46 上传

判断函数主要判断 `[esi+760]` 是否为0

如果为0，直接走 `D3D9.DLL+7C47`，否则走 `NtUserHwndQueryRedirectionInfo`

这里我有必要提一下，不要想着锁定 `[esi+760]` 的值，这样做会导致`D3D`无限初始化`Surface`

虽然进了 `NtUserHwndQueryRedirectionInfo`，但是会一直在这里死循环，不再进行任何绘制。

而且修改代码段的内存来强行让他走 `NtUserHwndQueryRedirectionInfo` 恐怕也不是各位看官想要的结果.

### 4.1

摆在面前的有两条路

1: 在创建窗口->创建`D3D`设备 这个流程中仔细分析创建需要的参数，找出其原因.
2: 在这个绘制流程中，进入判断函数前 改掉 `[esi+760]` 的值，进入 `NtUserHwndQueryRedirectionInfo` 之后再改回来

由于时间有限，我姑且只讲一下第二种方式。

笔者重新理了一下执行流程

![](https://attach.52pojie.cn/forum/202306/01/114833go377k7y988bk8k7.png)

image.png (49.77 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjYxODg4OXw0ZjM5NWY3OXwxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODg4OXw0ZjM5NWY3OXwxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes)

流程图2

2023-6-1 11:48 上传

如图所示，我们只需要在 `D3DKMTPresent` 这个函数进入内核之后

改变其 `[esp+760]` 的值为1

在进入判断代码时，就会顺理成章的进入 `NtUserHwndQueryRedirectionInfo`

然后我们在 `NtUserHwndQueryRedirectionInfo`里把 `[esp+760]` 的值改成 0

这样走到后面的检测代码中，就不会无限获取`D3D9`主`Surface`导致死循环。

还有个很重要的问题 `esp+760` 如何获取，这个比较简单了，经过笔者指针扫描+瞎猜

这个值可以通过如下方式获得。

![](https://attach.52pojie.cn/forum/202306/01/114937b9x0xkwwhl6gx7gx.png)

image.png (88.49 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjYxODg5MXw3NGNkMmExY3wxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODg5MXw3NGNkMmExY3wxNzkzNTcwMTg0fDE2ODA4NnwxNzkyMTYz&nothumb=yes)

CE指针

2023-6-1 11:49 上传

最后一级偏移为760，所得地址便是 `esp+760` ,所得值便是 `[esp+760]`。

## 结语：

来自未来的时间(2023/06/01)：

以后有空再对这类型内容做研究，届时依然在52更新
