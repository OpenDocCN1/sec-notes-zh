
```
0041DDAC   .  E8 EF4E0000   call aspack.00422CA0

0041DDB1   .^ E9 79FEFFFF   jmp aspack.0041DC2F

;跳过去后的代码

0041DC2F   > /6A 58         push 0x58

0041DC31   . |68 D0C74300   push aspack.0043C7D0

0041DC36   . |E8 C12A0000   call aspack.004206FC

0041DC3B   . |33F6          xor esi,esi

0041DC3D   . |8975 FC       mov dword ptr ss:[ebp-0x4],esi

0041DC40   . |8D45 98       lea eax,dword ptr ss:[ebp-0x68]

0041DC43   . |50            push eax                                 ; /pStartupinfo = FE1C4DA1

0041DC44   . |FF15 74214300 call dword ptr ds:[0x432174]             ; \GetStartupInfoW

0041DC4A   . |6A FE         push -0x2

0041DC4C   . |5F            pop edi                                  ;  kernel32.7C817077

0041DC4D   . |897D FC       mov dword ptr ss:[ebp-0x4],edi           ;  ntdll.7C930208

0041DC50   . |B8 4D5A0000   mov eax,0x5A4D

0041DC55   . |66:3905 00004>cmp word ptr ds:[0x400000],ax
```

## 4. Aspack

### 4.2 特征搜索
不适用，大跳转附件的代码好像是壳运行后才解压出来的，在没运行的情况下根本搜不到。

### 4.3 直接往下翻代码法
大跳转在入口地址前面，而且是运行中解压出来的，这个方法不适用。

### 4.4 单步跟踪法
前面两个`call`需要`F7`单步跟进去，否则就飞了，其他一路`F8`也能找到`OEP`，需要跟几页代码才能到。

### 4.5 内存镜像法（二次断点法）
根据大佬的经验，`Aspack`这个壳也不适用二次断点法。

### 4.6 一步到达OEP
大跳转要运行后才会解压出来，因此这个方法应该不适用。

### 4.7 最后一次异常法
完全停不下来，不适用。

### 4.8 SFX大法
入口地址在代码段外面，应是可以用`SFX`大法的，但是太慢了，我没有等到……

### 4.9 Dump和IAT修复
只需要使用`ollydump`进行`dump`就可以直接运行了，不需要特别的`IAT`修复，具体操作步骤祥见3.9。

## 5. tElock V0.98

### 5.1 ESP定律
这个壳运行时，即时调试选项中将所有异常中断忽略都选上，在硬件断点断下后，取消硬件断点，之后`F8`跟踪还是会被其他异常中断，最终报错退出。在`StrongOD`中选择`skip some exception`后，硬件断点也断不下来。

### 5.2 特征搜索
由于大跳转位置的代码是壳运行之后才逐步解压出来的，因此该方法不适用。

### 5.3 直接往下翻代码法
不适用。

### 5.4 单步跟踪法
壳代码运行过程比较长，难度较大。

### 5.5 内存镜像法（二次断点法）
首先将调试选项中将所有异常中断忽略都选上，然后`Alt+M`打开内存界面，找到第一个`.rsrc`资源，按`F2`设下断点。按`F9`运行一次，再次打开内存界面，发现还没运行到设置的断点位置。再次按下`F9`，进入内存界面，发现断点状态已经被清除了，这是在`.rsrc`资源上面的`CODE`资源处按`F2`下断点，再按`F9`，断下的位置就是`OEP`了。

### 5.6 一步到达OEP
不适用。

### 5.7 最后一次异常法
先将调试选项中的异常忽略全部取消，如果有`StrongOD`，将`skip some exception`取消。按`Shif+F9`运行，记录到第17次时，程序已经启动。重新载入，`Shif+F9`运行16次，这时找到堆栈区的第一个`SE`处理程序对应的地址，然后在代码区按`Ctrl+G`跳转到`4657C7`，在这一行按`F2`设下断点，然后再按`Shif+F9`运行一次，接着就是`Shift+F8`单步跟踪。

![](https://attach.52pojie.cn/forum/202206/08/135830zpj44j1yzy894fdl.png)

Telock-04.png (12.3 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUyODY4MHwzYWZkYjYxMXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY4MHwzYWZkYjYxMXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes)

2022-6-8 13:58 上传

需要跟比较长的代码，大概有3～4页，然后就能到下图的位置，比较典型的，前面一个`popad`，后面跟一个大跳转，跳过去就是`OEP`，因此就是这个位置了。

![](https://attach.52pojie.cn/forum/202206/08/135828nszre8zpe8bhqtqp.png)

Telock-03.png (29.01 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUyODY3OXwwZTE4MjU5YXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY3OXwwZTE4MjU5YXwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes)

2022-6-8 13:58 上传

### 5.8 SFX大法
入口地址不在代码段范围内，应该是可以使用`SFX`大法，但是太慢了，我没能等到。

### 5.9 Dump和IAT修复
`OEP`找到后，使用`Ollydump`进行`dump`，发现提示“无法读取被调试进行（00400000..00466FFF）的内存”的错误。将内存界面里面能设置全部权限的项目都设置完后，还是同样的错误。换用`LordPE`还是`PETools`都显示无反问权限，不知道什么原因。

又尝试了其他几个版本的`OD`，发现`OD`吾爱论坛专用版、`ollyICE`吾爱扣扣专版这两个版本无法`dump`，而`ollyICE`原版可以用`ollydump`正常导出。

然后使用`Import REConstructor`进行`IAT`修复，按照`ximo`大佬的教程，进行`trace level3`的修复，但是提示“can't initialize the tracer”，修复后的程序点击运行后没有任何反应。

后来网上查找了半天，才搞清楚这个是`Import REC 1.7`版本的`bug`，`ximo`大佬使用的是`1.6`版的，没有这个问题。

`Import REC 1.7`版本在进行`trace level3`的修复时，需要手动在插件中选择`tELoxk V0.98`才能正确修复，然后再将无效引导删除，再进行`dump`修复就可以了。

![](https://attach.52pojie.cn/forum/202206/08/135824m1z9pvnnrhk2helu.png)

Telock-01.png (43.85 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUyODY3N3wwYjhlMTE5MnwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY3N3wwYjhlMTE5MnwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes)

2022-6-8 13:58 上传

这里还需要注意的一个小细节是，进行`trace level3`的修复，`Import REC`选择的进程不能是`OD`载入的程序进行，必须选择另外单独打开的程序进程，否则修复时会程序死机。

## 6. WinUpack

### 6.1 ESP定律
按`ESP`定律操作流程，设定硬件断点停下后，清除硬件断点，然后一路`F8`跟踪，一直到`672DFB`位置，`retn`执行后，就来到了熟悉的`OEP`。

```
00672DF4    50              push eax
00672DF5    53              push ebx                                 ; comdlg32.76320000
00672DF6    FFD5            call ebp                                 ; kernel32.GetProcAddress
00672DF8    AB              stos dword ptr es:[edi]
00672DF9  ^ EB E7           jmp short WinUpack.00672DE2
00672DFB    C3              retn

;retn执行后到达
0045BE98    55              push ebp                                 ; kernel32.GetProcAddress
0045BE99    8BEC            mov ebp,esp
0045BE9B    6A FF           push -0x1
0045BE9D    68 58C15600     push WinUpack.0056C158
0045BEA2    68 CC154600     push WinUpack.004615CC
0045BEA7    64:A1 00000000  mov eax,dword ptr fs:[0]
0045BEAD    50              push eax
0045BEAE    64:8925 0000000>mov dword ptr fs:[0],esp
0045BEB5    83EC 58         sub esp,0x58
0045BEB8    53              push ebx                                 ; comdlg32.76320000
0045BEB9    56              push esi                                 ; WinUpack.005A9796
0045BEBA    57              push edi                                 ; WinUpack.0047C6A8
```

### 6.2 特征搜索
跳转部分的代码需要壳程序解压后才有，因此该方法不适用。

### 6.3 直接往下翻代码法
跳转部分的代码需要壳程序解压后才有，因此该方法不适用。

### 6.4 单步跟踪法
单步也能比较容易的跟到`OEP`位置，比`ESP`定律法跟踪的代码稍微多一点。

### 6.5 内存镜像法（二次断点法）
不适用，第一个断点就直接飞了。

### 6.6 一步到达OEP
跳转部分的代码需要壳程序解压后才有，不适用。

### 6.7 最后一次异常法
完全停不下来，这个方法不适用。

### 6.8 SFX大法
入口地址在代码段范围内，不适用。

### 6.9 Dump和IAT修复
按3.9的步骤进行操作就可以完成。

## 7. nsPack ver.3.x-4.1

### 7.1 ESP定律
按`ESP`定律的操作，在`popad`指令执行完后，下硬件断点，直接就断在`5AB833`，下一行就是大跳转。`OEP`入口的代码默认被识别成数据，需要右键`[分析]`→`[从模块中删除分析]`后，才能看到代码。

![](https://attach.52pojie.cn/forum/202206/08/140100rqp337olh8takwak.png)

nspack-01.png (1.99 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjUyODY4MXxlMjg2ODUzYnwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUyODY4MXxlMjg2ODUzYnwxNzkzNTc1MzYyfDE2ODA4NnwxNjQ2NDY2&nothumb=yes)

2022-6-8 14:01 上传

### 7.2 特征搜索
直接`Ctrl+B`搜索`61 9D E9`，找到的第一个位置就是大跳转位置。

### 7.3 直接往下翻代码法
直接往下翻5页左右，找到的第一个大跳转位置就是。

### 7.4 单步跟踪法
过程也比较简单，能够比较简单的找到。

### 7.5 内存镜像法（二次断点法）
不适用，第一个断点断不下来。

### 7.6 一步到达OEP
`ctrl+f`找了一下`popad`，大概第23个位置就是大跳转的位置。

### 7.7 最后一次异常法
`F9`运行过程中没有异常产生，因此该方法不适用。

### 7.8 SFX大法
当程序载入`OD`之后，入口点的地址在代码段之外，因此这个壳应该是可以用`SFX`大法的。尝试以块方式进行跟踪，等待很久后停在程序认为的入口位置，但是实际并不是准确的位置，正确的`OEP`在该地址前面一点位置。以字节方式进行跟踪，速度太慢，放弃。

### 7.9 Dump和IAT修复
直接使用插件`ollydump`进行操作，并不需要额外的`IAT`修复，程序直接可以运行。

## 8. 总结

### 8.1 统计
通过上述练习，基本熟悉了一些常用的压缩壳的脱壳方法，虽然对于加密壳、`VM`壳来说，要进一步学习的东西还有很多，但相比使用专用脱壳软件来说，应该是一大进步。根据上述过程，统计各种壳可以使用的查找`OEP`方法如下表：

壳类型 | 
ESP定律 | 
特征搜索 | 
直接翻代码 | 
单步跟踪法 | 
二次断点法 | 
一步达到OEP | 
最后一次异常法 | 
SFX法 | 
IAT修复 | 

UPX v3.91
```
