
先读一下内存，发现是1，那么按正常逻辑我们会发现确实和 `if ( *(a1 + 153) != 1 )` 条件不成立，所以未激活。

那么我们就狠狠的 `writeInt(0)`！

保存，自动编译为 `_index.js`，然后运行 `python` 文件让他自动注入app：

![](https://attach.52pojie.cn/forum/202301/27/215414kwgdmt15529mgwa2.jpg)

`16748226984007.jpg` (151.12 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxMnxkOTg4NDJhYXwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

绿绿的，很喜欢。

是不是很简单？大家都学会了吧？下面我们用object-c写一遍去持久化注入App。

> 估计没几个人能认真看到这里，我偷偷喵两句：
> 这个软件真是不懂那个几十美元收的是什么钱，激活了也不知道新增了什么功能，完全不知道收费版和不收费有什么区别。难道是这个绿色和红色的区别？
> 大家都喜欢绿油油的？呃呃呃蝈楠那可真够下头的捏。
> @ 腾讯公司 过来学习先进皮肤经验。（一开始没想到论坛真有这个人 还艾特出来了。。。）

## 第三步 dylib持续化注入

毕竟不能靠Frida先启动然后再启动app，这并不酷，也不符合我对高科技的想象。

### 新建项目

![](https://attach.52pojie.cn/forum/202301/27/215417p8jf4xxbxmmf11oh.jpg)

`16748231731920.jpg` (130.04 KB, 下载次数: 3)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxM3w0YjU3YjNkN3wxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

然后随便起个名：

![](https://attach.52pojie.cn/forum/202301/27/215419u1abnxu1vl3anoba.jpg)

`16748231970122.jpg` (96.05 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxNHw5NTBjMmFhOHwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

点创建选择一个保存的路径即可。

然后我们打开 `.m` 文件开始编写代码。

![](https://attach.52pojie.cn/forum/202301/27/215421r424e336xte12atm.jpg)

`16748232399429.jpg` (12.12 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxNXxiYWQ4MzA5ZHwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

直接把 `rd_route` 复制两个文件到项目里即可。

![](https://attach.52pojie.cn/forum/202301/27/215424xf18ntyofckhhkc6.jpg)

`16748233192993.jpg` (105.64 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxNnw5NTU0ZGE2Y3wxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

如图所示。

然后 `import` 一下几个基本库：

UI控件库是注入CleanMyMac的，可以不写。

然后我们在 `load` 中写自己的ObjectC代码

![](https://attach.52pojie.cn/forum/202301/27/215426ydt4dnz2xghh141g.jpg)

`16748233952045.jpg` (125.08 KB, 下载次数: 1)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxN3xiZjI2NjQ3N3wxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

我们今天主要看AirBuddy，大家不要在意上面的一连串注入代码。

![](https://attach.52pojie.cn/forum/202301/27/215429hm0zf0su6esmt3f2.jpg)

`16748234566506.jpg` (243.71 KB, 下载次数: 2)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxOHxhYTQzNTMwOXwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

我们在frida里面直接能修改寄存器，OC里很遗憾不能直接操作。

但是我们有ASM！

直接写内联汇编啊铁子！（突然激动）

我们首先屏蔽掉 `activate` 函数，这没用。

![](https://attach.52pojie.cn/forum/202301/27/215431v8hq5qq3j8z0pc5w.jpg)

`16748235786385.jpg` (216.75 KB, 下载次数: 0)

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcxOXw2ODliYjE3YXwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

然后我们用 `_dyld_get_image_vmaddr_slide(0) + 0x100050480;` 加上函数偏移得到内存中这个函数的偏移地址。

为什么 `_dyld_get_image_vmaddr_slide` 参数选0捏？因为这是选择第一个镜像（也就是可执行文件本身的内存空间镜像基址），具体细节懒得说，百度搜一下大家就懂了。

记住写0就是表示App可执行文件本身，也就是你注入的那个进程本身。别乱写哦，因为app在运行时不止一个内存镜像。

然后我们伪造一个函数进行挂钩。

```
int _0x100050480New() {

    NSLog(@"==== _0x100050480New called");

    __asm

    {   //内联汇编直接修改寄存器的值

        //对比原始二进制反汇编 cmp byte ptr [r13+99h], 1

        mov byte ptr [r13+99h], 0

    }

    NSLog(@"==== _0x100050480New call end");

    return _0x100050480Ori();//调用原函数恢复执行

}
```

接下来声明一下原始函数保存地址，这里直接转换成函数结构体可以直接调用

`int (_0x100050480Ori)();`

定义原函数返回值 `int` 无参数，加上取该变量内存地址里的值以被下面挂钩函数赋新值：

`rd_route((void *) _0x100050480, _0x100050480New, (void ) &_0x100050480Ori);`

`(void ) &_0x100050480Ori` 这里用 `(void *)` 进行类型转换，`&` 取地址符传递地址进函数。

然后 `_0x100050480Ori` 保存的就是原函数地址了，注意如果不写 `*` 你会得到 `_0x100050480Ori` 的内存地址而不是这个内存地址里面的值。

然后使用经典的 `mov` 指令在函数进入时将寄存器的值赋予0，并恢复原函数执行，这样原函数读到的就是新的 `r13` 寄存器里的这个0值啦！
