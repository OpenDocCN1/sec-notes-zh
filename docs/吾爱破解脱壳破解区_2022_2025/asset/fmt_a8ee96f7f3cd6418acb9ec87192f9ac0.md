
```
bool __fastcall RPluginBase::isTrial(RPluginBase *this)
{
  return *((_BYTE *)this + 40) == 0;
}
```

![](https://attach.52pojie.cn/forum/202310/26/232432oc7nko7nd67hezqk.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkxOHxjMTE2YzM4OXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983314609989.jpg` (106.37 KB, 下载次数: 3) 2023-10-26 23:24 上传

将这个函数头开始直接覆盖为 `6a 00 58 c3`，保存修改到文件，重新运行试试：

![](https://attach.52pojie.cn/forum/202310/26/232436lcc5ffrq51c52cgk.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkxOXxjMzM2YzQ0MnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983315510926.jpg` (205.27 KB, 下载次数: 1) 2023-10-26 23:24 上传

神奇的一幕出现了，虽然还有激活弹窗，但是这里已经成功授权！

![](https://attach.52pojie.cn/forum/202310/26/232439gn2swbhy12oecm2b.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyMHw3NjY0NGU4YnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983315832529.jpg` (165.56 KB, 下载次数: 1) 2023-10-26 23:24 上传

包括这里也变成了授权状态，和上面的试用版本截然不同！

于是，落叶记住了两个函数：`checkLicense` 和 `isTrial`，让其他的文件中的这两个函数也统统消失吧！

对照

![](https://attach.52pojie.cn/forum/202310/26/232442ayvxvmzwu4r99yvs.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyMXwxMzNlYzJiYXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983316875164.jpg` (39.14 KB, 下载次数: 1) 2023-10-26 23:24 上传

有激活字样的文件名，一一修改这里不再赘述。

![](https://attach.52pojie.cn/forum/202310/26/232445fzocoqqhtqzc9c3p.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyMnxjNGY3M2IwYXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983325262561.jpg` (148.16 KB, 下载次数: 3) 2023-10-26 23:24 上传

总共需要修改这 9 个文件，同样的 `checkLicense` 强制：

```
mov     ebx, 0Ah
nop
```

`isTrial` 变成 `6a 00 58 c3`。

却说那奄奄一息的 QCAD 看着缓缓走来的魔子落叶，惊恐道：“不...！不能杀我！我背后的迦南世家不会放过你的！”

落叶毫无反应，淡漠的看着挣扎的 QCAD 冷冷道：“三年前，曾经你们迦南家族从我手里抢走的，我要亲手一步步夺回来！”

目光如电，看向迦南二世隐匿的方向，迦南二世心头一惊，连忙低喝道：“发现我们了，快走！此子绝不是区区魔导师实力！暂且避其锋芒！”

临走前仇恨的看着魔子落叶：“早晚要除掉你这个孽障！”，一咬牙，化为一阵青烟遁走。

落叶看着求饶的 QCAD，一脚踏下，只听得喀喳一声，世间一片寂静，再无声响。

重新打开 QCAD 看下：

![](https://attach.52pojie.cn/forum/202310/26/232449n9xrpzbrlfzs55ds.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyM3w4NTE1YmQwY3wxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983326261295.jpg` (783.12 KB, 下载次数: 3) 2023-10-26 23:24 上传

成了激活所有功能，导出 pdf 也没有水印，弹窗也没有了，非常完美。

## 3. 结束

这篇教程不涉及到 `LLDB` / 补丁注入破解，尽管我是 `lldb` + 注入 `hook` 一路调试过来的，本篇教程仅用于给 X1a0He 同学以及其他同学用于入门逆向学习用，所以讲的非常精炼，省略了很多繁杂的步骤。

最后留了一个 30 分钟强制关闭的漏洞，请各位同学根据上面的思路自己处理掉吧！

## 4. `LLDB` 解决破解不完全依然提示弹框

我们 `lldb /Applications/QCAD.app/Contents/MacOS/QCAD` 启动后，可以看到：

```
23:46:18: Debug:    loading plugins...
23:46:19: Debug:    RDwgPlugin::init: trial
23:46:19: Debug:    RProScriptsPlugin::init: trial
23:46:20: Debug:    RTracePlugin::init: trial
23:46:20: Debug:    loading static plugins...
Warning:  Populating font family aliases took 632 ms. Replace uses of missing font family "Sans" with one that exists to avoid this cost.
Warning:  Cannot read file ':/scripts/Pro/Block/CreateLibraryItem/CreateLibraryItem-inverse.svg', because: Expected '?', '!', or '[a-zA-Z]', but got '
```

![](https://attach.52pojie.cn/forum/202310/26/235901d110ouluelc402dz.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyNnwxMjNjZDgyMHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983352372312.jpg` (392.58 KB, 下载次数: 1) 2023-10-26 23:59 上传

这三个依然是 `trial` 阶段，所以我们要 `patch` 掉：

```
RDwgPlugin::init: trial
RProScriptsPlugin::init: trial
RTracePlugin::init: trial
```

![](https://attach.52pojie.cn/forum/202310/26/235907or3quqibiebuibjr.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyN3xiZjA4YzMwZXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983354154562.jpg` (222.61 KB, 下载次数: 0) 2023-10-26 23:59 上传

可以看到结果是这样子，所以我们 `ida` 打开这个文件直接搜：

![](https://attach.52pojie.cn/forum/202310/26/235911vorr149e1og62z9e.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyOHxjODlhZDk3MXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983354801911.jpg` (223.86 KB, 下载次数: 2) 2023-10-26 23:59 上传

可以看到这里存在字符串

![](https://attach.52pojie.cn/forum/202310/26/235914lvzfri1zyff0445w.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkyOXxkNjFjOGQzZXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983356494770.jpg` (113.71 KB, 下载次数: 4) 2023-10-26 23:59 上传

仔细一看，这里 `checkLicense` 里面的修改根本没起作用，那么我们只好暴力点，直接全部强制条转：

我们将其 `jz` 跳转改为 `nop` 即可：

![](https://attach.52pojie.cn/forum/202310/26/235921i1g61yokg0mgkpbz.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkzMHxkODQ0NTA2M3wxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983357647038.jpg` (450.22 KB, 下载次数: 2) 2023-10-26 23:59 上传

改后：

![](https://attach.52pojie.cn/forum/202310/26/235925pzomqiqiyzqhmo4l.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkzMXxmMDFjNzRmZnwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983358029671.jpg` (258.24 KB, 下载次数: 3) 2023-10-26 23:59 上传

把指令所在位置强制改为 `9090`，即可 `nop`。

![](https://attach.52pojie.cn/forum/202310/26/235928pr00znqitqdzh4gt.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkzMnwwOWU2MDZjMHwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983358273253.jpg` (174.95 KB, 下载次数: 1) 2023-10-26 23:59 上传

保存到文件，启动：

![](https://attach.52pojie.cn/forum/202310/26/235932jdu1c55afqcsuvsq.jpg)
[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTkzM3w5YTI2YWM3YXwxNzkzNTczMDQ1fDE2ODA4NnwxODQ5MTcy&nothumb=yes) `16983358701809.jpg` (233.41 KB, 下载次数: 1) 2023-10-26 23:59 上传

成功解决第一个，那么第二个第三个我就不多说了，大家去试试吧！

注入库补丁开源地址：

[https://github.com/QiuChenlyOpenSource](https://github.com/QiuChenlyOpenSource)

注入补丁参考项目：

[https://github.com/QiuChenlyOpenSource/InjectLib](https://github.com/QiuChenlyOpenSource/InjectLib)
