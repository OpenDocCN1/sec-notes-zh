
# [原创] 练习笔记之手撕常见压缩壳

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1646466
- 抓取时间：2026-11-01T23:22:44

## 手撕压缩壳练习

这个是对之前网友“关于压缩壳脱壳步骤详解！看这一篇足够。”（[//www.52pojie.cn/thread-1641368-1-1.html](https://www.52pojie.cn/thread-1641368-1-1.html)）的练习文件进行练习的一点记录，写的比较流水帐，非菜鸟建议直接看结论就够了。

### 1. UPX v3.91

#### 1.1 `ESP`定律

`F8`单步，运行到寄存器只有`ESP`和`EIP`为红色（有的程序运行一步就可以，有的需要多步），右键`ESP`的地址，选择在数据窗口跟随，然后在数据窗口右键该地址，设置硬件访问断点，然后`F9`运行。

![upx-01.png](https://attach.52pojie.cn/forum/202206/08/135351nxncawwwbw5zzqxo.png)

`upx-01.png` (14.82 KB, 下载次数: 1) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY0NHw1ZmU5YWE1NXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:53 上传

程序断在457348这行，单步运行到47355这个地址，后面都是0，看多了就知道这个是UPX的特征。

![upx-02.png](https://attach.52pojie.cn/forum/202206/08/135353dmr7dxvw4y6m4hya.png)

`upx-02.png` (11.71 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY0NXw0NjNkOTVhOXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:53 上传

这个跳转过去的地址就是到了41DDAC，再往下单步执行，41DDB1的跳转跳过去就到了OEP（OEP就是程序没加壳前的入口地址，不同编译软件编译出来的代码特征各不相同，但大部分都是`push XX`开头）。

```
0041DDAC    E8 EF4E0000     call UPX.00422CA0
0041DDB1  ^ E9 79FEFFFF     jmp UPX.0041DC2F        ;大跳转
;跳过去后的代码
0041DC2F    6A 58           push 0x58
0041DC31    68 D0C74300     push UPX.0043C7D0
0041DC36    E8 C12A0000     call UPX.004206FC
0041DC3B    33F6            xor esi,esi
0041DC3D    8975 FC         mov dword ptr ss:[ebp-0x4],esi
0041DC40    8D45 98         lea eax,dword ptr ss:[ebp-0x68]
0041DC43    50              push eax
0041DC44    FF15 74214300   call dword ptr ds:[0x432174]             ; kernel32.GetStartupInfoW
```

#### 1.2 特征搜索

这个方法针对UPX壳的特征（大跳转后面接着是大块的0），直接在代码区`Ctrl+B`搜索二进制字符串，输入一长串的0，然后搜索，找到的位置往上看，就是一个大跳转，单步跟踪两步就到了OEP。

![upx-03.png](https://attach.52pojie.cn/forum/202206/08/135355gohtfbb719kgofk4.png)

`upx-03.png` (12.63 KB, 下载次数: 1) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY0NnwyNDE1OWU1M3wxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:53 上传

#### 1.3 直接往下翻代码法

按大佬的说法，UPX壳太简单了，直接往下翻就是。OD打开程序后，直接停在`00457170 > $  60  pushad`位置（如果没有停在这个位置，打开调试选项按下图将第一次暂停设置在主模块入口点），然后往下翻大概5页代码，就到了457355这个大跳转（跳的距离比较远）位置，后面都是一堆的0。这个时候到41DDAC看的话，都是0，这个是正常的，因为壳程序还没运行，内容还没解压出来。在457355行按`F2`设下断点，`F9`运行后，再到41DDAC看，程序代码就都出来了，这个往下单步跟踪两步就到OEP了。

![upx-04.png](https://attach.52pojie.cn/forum/202206/08/135357a4ja4amrzvamy40a.png)

`upx-04.png` (20.54 KB, 下载次数: 2) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY0N3wyYjdjYzk0YXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:53 上传

![upx-05.png](https://attach.52pojie.cn/forum/202206/08/135359nk6htf6kv91yua2t.png)

`upx-05.png` (28.19 KB, 下载次数: 0) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY0OHw0ZGFiY2JjOHwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:53 上传

#### 1.4 单步跟踪法

按介绍“单步（`F8`）、单步进入（`F7`）和运行到（`F4`）功能，完整走过程序的自脱壳过程（不断使用`F8`向下运行，遇见向上跳转的就用`F4`），跳过一些循环恢复代码的片段，并用单步进入确保程序不会略过OEP。”，其中`F4`在下拉菜单没有看到，百度了一下才知道这个快捷键的作用是“运行到所选择的那一行。作用就是直接运行到光标所在位置处暂停。”。之前只会在循环往上跳的下一行`F2`下断点后执行，现在学会了用`F4`，效率更高。

试着跟了一次，飞了……为什么？明明前面就两个系统函数调用，函数调用执行完程序没有飞啊。

![upx-06.png](https://attach.52pojie.cn/forum/202206/08/135401bm442r5cbw4ofb5f.png)

`upx-06.png` (28.17 KB, 下载次数: 1) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY0OXw1OWZmOTA3YXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:54 上传

重新再来一次，发现4572DF这行有个向上跳转，选中它的下一行按`F4`就飞了……这对新手太不友好了吧～

只好仔细在看看循环部分的代码，发现里面有个跳转直接调到4572E7了，因此`Ctrl+F2`重来，直接选中4572E7按`F4`，终于没有飞，再往后`F8`就毫无波澜的来到了OEP。

#### 1.5 内存镜像法（二次断点法）

按攻略方法，试了n次，发现都不管用。百度了一下，发现大佬说UPX和ASPACK用这个方法不灵光，原因是这两种壳将`code`段解压完毕的时候马上就`JMP`到OEP去，所以第二次断点断不下来。具体原理可以再查查资料了解，对新手来说，知道这个方式不适用就行了，否则像我一样，一直怀疑是自己操作的姿势不对，反复操作，直到怀疑人生……

#### 1.6 一步到达OEP

`ctrl+f`查找`popad`，注意去掉整个块的勾选，第一个跳转就来到了457355附近。

![upx-07.png](https://attach.52pojie.cn/forum/202206/08/135403s8lvllyjmt818w1y.png)

`upx-07.png` (11.78 KB, 下载次数: 1) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY1MHwwZGRiYTM3OXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:54 上传

#### 1.7 最后一次异常法

OD打开文件，按下`F9`。呃……直接程序界面出来了。把调试选型中的忽视异常选项全部去掉，把插件Strong OD中的“忽视”也取消掉了，重启OD，再次按下`F9`，程序界面又直接出来了。猜测是不是有哪个插件也有忽视异常的功能，导致停不下来。

关掉OD，找到OD目录下的`plugin`文件夹，将里面的文件全部移走，然后再试。结果，这感觉是吃了炫迈啊，完全停不下来……导致我怀疑是不是我的OD坏了。还好换个其他的壳试了试，至少是有异常断点能停下来的，只能说是UPX的壳没有异常断点事件，所以这个方法不好使。

#### 1.8 SFX大法

由于SFX大法使用的条件是“当程序载入OD之后，入口点必须是代码段之外才可以用”，UPX壳的入口地址为457170，正好在程序代码段地址范围内，因此该方法不适用。

![upx-08.png](https://attach.52pojie.cn/forum/202206/08/135405n0znn577l0zqqzq0.png)

`upx-08.png` (23.08 KB, 下载次数: 0) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY1MXxmZGVmYmJlMXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:54 上传

#### 1.9 Dump和IAT修复

UPX的壳只需要使用`ollydump`进行dump就可以直接运行了，不需要特别的IAT修复，具体操作步骤祥见3.9。

### 2. PECompact v2.78a ~ 3.11

#### 2.1 `ESP`定律

设置硬件断点后，会停在以下位置。

```
005AA203    53              push ebx
```

接着`F8`单步跟踪，下面找到一个大跳转，大跳转后面全是0。这个跳转过去后就是OEP了。

![PECompact-01.png](https://attach.52pojie.cn/forum/202206/08/135448ozipo2les8es8ceo.png)

`PECompact-01.png` (20.38 KB, 下载次数: 0) [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY1M3xjYjYwMjljMnwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes) 2022-6-8 13:54 上传

#### 2.2 特征搜索

根据大佬的总结，PECompact壳的特征码为`03 CA 68 00 80 00 00 6A 00 57 FF 11 8B C6 5A 5E 5F 59 5B 5D`，看下面代码就可以看出，大跳转前的指令都是固定的。
