
```
nop 2
returnhere:
[DISABLE]
//code from here till the end of the code will be used to disable the cheat
dealloc(newmem)
"PlantsVsZombies.exe"+88263:
db 2B 45 24 89 44 24 10
//sub eax,[ebp+24]
//mov [esp+10],eax
```

## 三、植物0元购

既然是0元购，那就不需要阳光值啦，找到阳光值，锁定为0吧。

![](https://attach.52pojie.cn/forum/202407/15/203539l0yg3yzgnr0030ig.png)

`3.1.png` (51.08 KB, 下载次数: 4)

[forum.php?mod=attachment&aid=MjcwODQzM3xjZTJiZWJhY3wxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQzM3xjZTJiZWJhY3wxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

思考：当我们有足够的阳光，图标是亮的，不够的话就是暗的。那说明，有个地方一直在窥视我们的阳光值，并与植物的花费在做比较，判断我们是否拥有足够的阳光值。

那就从刚刚找到的阳光值下手，右键查找是什么访问了这个地址，

![](https://attach.52pojie.cn/forum/202407/15/203542fg6acjbxjcpgla3o.png)

`3.2.png` (30.28 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQzNHxlZGYwYTNmN3wxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQzNHxlZGYwYTNmN3wxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

![](https://attach.52pojie.cn/forum/202407/15/203544f3czjq87y8d48qq0.png)

`3.3.png` (23.82 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQzNXw5MGE4Y2IxNnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQzNXw5MGE4Y2IxNnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

找到了两条代码，下断点，看看寄存器有什么变化。如果寄存器的值看着像地址，就右键选择以十六进制显示，在内存数据里看看有没有熟悉的数据。

![](https://attach.52pojie.cn/forum/202407/15/203546puta00nwzuok7k2u.png)

`3.4.png` (60.05 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQzNnxmYzY5MjExZHwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQzNnxmYzY5MjExZHwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

![](https://attach.52pojie.cn/forum/202407/15/203703ry2wlruislzfwwg5.png)

`3.5.png` (40.22 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQ0NnwwYzQyYjYxNnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQ0NnwwYzQyYjYxNnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:37 上传

两条代码都没有在寄存器或者地址里找到想要的数据，但是在第一条代码里还是得到了一些信息，而第二条代码可以直接pass掉了。回到第一条代码，下断点，把目光放在右下，如下图

![](https://attach.52pojie.cn/forum/202407/15/203550pk9bon6bnkofagau.png)

`3.6.png` (180.94 KB, 下载次数: 4)

[forum.php?mod=attachment&aid=MjcwODQzOHwwNTE3MDg2YnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQzOHwwNTE3MDg2YnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

![](https://attach.52pojie.cn/forum/202407/15/203553woepojdf5q9v6ppx.png)

`3.7.png` (48.78 KB, 下载次数: 5)

[forum.php?mod=attachment&aid=MjcwODQzOXxiZDVjMDBmZXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQzOXxiZDVjMDBmZXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

双击进入后，我们会来到 `test al,al` 这行代码，我们在上面 `call` 前面下一个断点

![](https://attach.52pojie.cn/forum/202407/15/203555p7xpfgqaqtaqp5gw.png)

`3.8.png` (26.48 KB, 下载次数: 5)

[forum.php?mod=attachment&aid=MjcwODQ0MHwzMWI0ZjAyNnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQ0MHwzMWI0ZjAyNnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

如上图，代码停在这里时，我们可以从寄存器发现，已经拿到植物购买需要的花费阳光值，我们可以尝试在这里写段代码让 `eax = 0`，看看能不能购买。

![](https://attach.52pojie.cn/forum/202407/15/203557mbuu3ru3dkpsu2kd.png)

`3.9.png` (65.06 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQ0MXw2MTMwNjkwYnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQ0MXw2MTMwNjkwYnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:35 上传

这是让 `eax = 0` 的结果，图标虽然亮了起来，但还是无法购买。OK，既然找到了想要的数据，就说明步骤没错，现在，需要在更早的时候去处理 `"7D"` 这个数据，第一时间闪过的就是源头，对，就是 `"7D"` 第一次出现的地方。

`PlantsVsZombies.exe+88478 - call PlantsVsZombies.exe+1BAB0`：这是我们当前的地址，我们在汇编代码窗口一直往上滑，在每个 `call` 前面都下断点，看看寄存器有没有看到 `"7D"` 值，有的话就继续往上找 `call`，直到那个 `call` 在被运行前，寄存器是不存在 `"7D"` 为止。

直到找到 `PlantsVsZombies.exe+883B4 - call PlantsVsZombies.exe+1DAE0`：这个 `call` 运行前并没有我们需要的 `"7D"`，那这个 `call` 就是获取我们植物购买需要的花费阳光值 `"7D"`。就在这里下断点，使用 `F7` 跟进这个 `call`，跟进以后在 `PlantsVsZombies.exe+1DAE4 - call PlantsVsZombies.exe+67B00`，这个 `call` 能拿到 `"7D"`，那就继续 `F7` 一步一步跟进。我们不需要看代码做了什么，只需要盯着寄存器看到 `"7D"` 的出现即可。

直到运行到这行代码：`PlantsVsZombies.exe+4A3C13 - mov eax,[eax*4+PlantsVsZombies.exe+3A2010]`：我们需要的 `"7D"` 终于第一次闪亮登场。

![](https://attach.52pojie.cn/forum/202407/15/203601qg0rvktuwrurb1uw.png)

`3.11.png` (43.57 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQ0M3w3YTEyNGExZnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQ0M3w3YTEyNGExZnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:36 上传

![](https://attach.52pojie.cn/forum/202407/15/203604lsesf6m17s1psefq.png)

`3.12.png` (73.53 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQ0NHwwOWE1ZjVkZXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQ0NHwwOWE1ZjVkZXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:36 上传

在内存区域跟进这个 `PlantsVsZombies.exe+3A2010` 地址看看。绿油油一片，还有一些很让人熟悉的数据，这里就是存放植物花费和冷却时间的基地址。

为了随用随关，我们就尽量不改这一块地址里面的值了。从汇编代码看，直接给 `eax` 赋予 `0`，就可以做到0元购了。

![](https://attach.52pojie.cn/forum/202407/15/203606ctf4ppft1eecfi8r.png)

`3.13.png` (104.84 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjcwODQ0NXw3ZWMyY2NjYnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwODQ0NXw3ZWMyY2NjYnwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-15 20:36 上传

### 最后CE脚本
