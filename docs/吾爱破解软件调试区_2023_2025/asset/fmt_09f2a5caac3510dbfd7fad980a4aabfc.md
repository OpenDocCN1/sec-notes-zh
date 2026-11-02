
```
text:0803DCF9                               ; __unwind { // loc_8184A54
.text:0803DCF9 55                            push    ebp
.text:0803DCFA 81 EC 04 01 00 00             sub     esp, 104h                       ; esp开拓栈空间0x104
.text:0803DD00 8D 6C 24 FC                   lea     ebp, [esp-4]
.text:0803DD04 A1 B8 0F 23 08                mov     eax, ___security_cookie
.text:0803DD09 33 C5                         xor     eax, ebp
.text:0803DD0B 89 85 04 01 00 00             mov     [ebp+108h+var_4], eax
.text:0803DD11 6A 4C                         push    4Ch
.text:0803DD13 B8 54 4A 18 08                mov     eax, offset loc_8184A54
.text:0803DD18 E8 B4 A4 00 00                call    __EH_prolog3_catch
.text:0803DD18
.text:0803DD1D 8B 85 1C 01 00 00             mov     eax, [ebp+108h+arg_C]
.text:0803DD23 8B BD 10 01 00 00             mov     edi, [ebp+108h+arg_0]
.text:0803DD29 8B 9D 14 01 00 00             mov     ebx, [ebp+108h+arg_4]
.text:0803DD2F 89 7D D8                      mov     [ebp+108h+var_130], edi
.text:0803DD32 89 45 D0                      mov     [ebp+108h+var_138], eax
.text:0803DD35 E8 F2 39 00 00                call    sub_804172C
.text:0803DD35
.text:0803DD3A 33 F6                         xor     esi, esi                        ; esi清0，之后用于判断值是否为空
.text:0803DD3C 83 7F 08 03                   cmp     dword ptr [edi+8], 3
.text:0803DD3C
.text:0803DD40                               ;   try {
.text:0803DD40 89 75 FC                      mov     [ebp+108h+var_10C], esi
.text:0803DD43 0F 84 B7 01 00 00             jz      loc_803DF00
.text:0803DD43
.text:0803DD49 89 75 E4                      mov     [ebp+108h+var_124], esi
.text:0803DD4C 89 75 E8                      mov     [ebp+108h+var_120], esi
.text:0803DD4F 83 7F 0C 01                   cmp     dword ptr [edi+0Ch], 1
.text:0803DD4F                               ;   } // starts at 803DD40
.text:0803DD4F
.text:0803DD53                               ;   try {
.text:0803DD53 C6 45 FC 01                   mov     byte ptr [ebp+108h+var_10C], 1
.text:0803DD57 0F 85 4C 01 00 00             jnz     loc_803DEA9
.text:0803DD57
.text:0803DD5D 68 2C DB 19 08                push    offset aName                    ; "name"
.text:0803DD62 57                            push    edi                             ; int
.text:0803DD63 8D 4D E4                      lea     ecx, [ebp+108h+var_124]
.text:0803DD66 C6 45 EF 00                   mov     [ebp+108h+var_119], 0
.text:0803DD6A E8 68 3A FE FF                call    sub_80217D7
.text:0803DD6A
.text:0803DD6F 39 75 E4                      cmp     [ebp+108h+var_124], esi
.text:0803DD72 75 69                         jnz     short loc_803DDDD
.text:0803DD72
.text:0803DD74 68 4C DB 19 08                push    offset aSing                    ; "SING"
.text:0803DD79 57                            push    edi                             ; int
.text:0803DD7A 8D 4D DC                      lea     ecx, [ebp+108h+var_12C]         ; 指向SING表入口
.text:0803DD7D E8 84 3D FE FF                call    sub_8021B06                     ; 处理SING表
.text:0803DD7D
.text:0803DD82 8B 45 DC                      mov     eax, [ebp+108h+var_12C]         ; SING表入口赋值给eax
.text:0803DD85 3B C6                         cmp     eax, esi                        ; 判断表入口是否位空
.text:0803DD85                               ;   } // starts at 803DD53
.text:0803DD85
.text:0803DD87                               ;   try {
.text:0803DD87 C6 45 FC 02                   mov     byte ptr [ebp+108h+var_10C], 2
.text:0803DD8B 74 37                         jz      short loc_803DDC4               ; 若我们处理的SING表不出差错，这里是不会进行跳转的
.text:0803DD8B
.text:0803DD8D 8B 08                         mov     ecx, [eax]                      ; 这里传入的是SING表的第一个四字节，这里是1.0版本，也就是00 01 00 00
.text:0803DD8F 81 E1 FF FF 00 00             and     ecx, 0FFFFh                     ; 这里进行判断想与，会设置对应eflags标志位
.text:0803DD95 74 08                         jz      short loc_803DD9F               ; 由于上一步设置了相应标志位，因此在这里跳转
.text:0803DD95
.text:0803DD97 81 F9 00 01 00 00             cmp     ecx, 100h
.text:0803DD9D 75 21                         jnz     short loc_803DDC0
.text:0803DD9D
.text:0803DD9F
.text:0803DD9F                               loc_803DD9F:                            ; CODE XREF: sub_803DCF9+9C↑j
.text:0803DD9F 83 C0 10                      add     eax, 10h                        ; eax本来是存放SING表首地址，这里加上0x10偏移指向uniqueName
.text:0803DD9F                                                                       ; uniqueName域
.text:0803DDA2 50                            push    eax                             ; Source
.text:0803DDA3 8D 45 00                      lea     eax, [ebp+108h+Destination]     ; Destination是-0x108,所以这里应该就是普通的lea eax,[ebp]
.text:0803DDA6 50                            push    eax                             ; Destination
.text:0803DDA7 C6 45 00 00                   mov     [ebp+108h+Destination], 0
.text:0803DDAB E8 48 3D 13 00                call    strcat                          ; 漏洞点
```

## 5. 样本分析（阶段一：栈溢出）

我们首先获取到对应样本，其为一个pdf，当我们打开此pdf会有个任意命令执行的功能，具体呈现出的效果为打开一个计算器，如下：

![](http://imgsrc.baidu.com/forum/pic/item/6609c93d70cf3bc7f8d2ec269400baa1cc112a2d.jpg)

然后执行他，会出现一个明显的pdf弹框，但不完全，接下来就会出现计算器

![](http://imgsrc.baidu.com/forum/pic/item/c9fcc3cec3fdfc03e1b7c272913f8794a5c22628.jpg)

当然也可以是别的命令，真正的恶意样本我们用来调试，在这里使用计算器来暂时说明我们可以任意控制了而已

首先我们打开`OD`，加载Adobe文件下的`AcroRd32`可执行文件

![](http://imgsrc.baidu.com/forum/pic/item/314e251f95cad1c8c771dded3a3e6709c83d5135.jpg)

然后我们按下`F9`来运行程序，这样可以用来加载我们需要的库

这里我们先配合之前的`IDA`静态分析的结果，我们利用`ctrl + g`进行地址跟踪，然后双击十六进制部分进行断点，至于下断点的位置，有以下三个点，
- 首先就是我们讲`SING`表赋值给`eax`的那个点(0x803dd82)；
- 然后就是执行`strcat`函数的点(0x803ddab)
- 最后就是一个关键漏洞点(0x808b308)，这个点我们之后详细介绍

此时我们使用`OD`打开的`Adobe Reader`打开样本`Exploit`，然后会断到我们的第三个断电处

![](http://imgsrc.baidu.com/forum/pic/item/6a63f6246b600c336800fe025f4c510fd8f9a1fb.jpg)

该指令是执行`eax`寄存器保存的地址指向的函数，这里我们可以从右上角的`Registers`窗口看到`eax`的值，发现他是在栈上的，此时我们右键点击`eax`，然后选择`Follow in Stack`，这样我们的栈窗口看到咱们`eax`中保存的值所指向的地址是0x80833EF,此刻我们返回`IDA`，查看该地址所在的函数

```
size_t *__cdecl sub_80833EF(int a1, int a2, void *a3, size_t *a4)

{

  size_t *result; // eax

  switch ( a2 )

  {

    case 0:

      return (size_t *)sub_8083119(a3, *a4);

    case 2:

      return (size_t *)sub_80830AE(*a4);

    case 3:

      result = (size_t *)sub_80828ED(*(_DWORD *)(a1 + 4), 0);

      *a4 = (size_t)result;

      break;

    default:

      result = a4;

      *a4 = 0;

      break;

  }

  return result;

}
```

观察这里是一个`switch`选择语句，看别的师傅博客是说明这里是处理`SING`表的时候会执行的函数，具体情况我们到下面再来讲解

接下来我们再次`F9`，类似于`gdb`当中的`c`，这里会到达我们之前下的第一个断点，这条指令会将我们`SING`表的首地址传入`eax`，我们`F8`单步执行到下一条指令来查看以下`EAX`的值

![](http://imgsrc.baidu.com/forum/pic/item/342ac65c10385343f388a7ebd613b07ecb8088b0.jpg)

此时我们查看`eax`指向地址的值，首先右键`eax`，然后选择`follow in dump`，接着会再二进制窗口显示我们值指向的地址，这里从我们之前了解到的`SING`表结构会知道`uniqueName`的地址应该是`SING`表偏移0x10字节的地方，也就是0x035529a0,可以看到样本中`uniqueName`字段十分长，然后我们单步执行到`push eax`，这里是将我们的`uniqueName`字段的地址压栈

![](http://imgsrc.baidu.com/forum/pic/item/b2de9c82d158ccbf231c95f05cd8bc3eb0354153.jpg)

然后我们再次`F9`运行到第二个断点处，这里即将调用`strcat`函数，可以看到其目的地址已经在栈上了，目的地址也是指向栈上的一个缓冲区地址，此时我们查看一下目前的目的地址附近的值，我们在栈窗口上`ctrl+g`，然后输入目的地址0x12e468

![](http://imgsrc.baidu.com/forum/pic/item/8b82b9014a90f603d830bc8d7c12b31bb151ed7c.jpg)

然后我们单步`F8`步过`call strcat`指令，由于我们没有对`uniquename`进行长度检查，这就导致了我们现如今会将其全部拷贝到栈上指定的地址，结果如下

![](http://imgsrc.baidu.com/forum/pic/item/00e93901213fb80e5f23411d73d12f2eb838940d.jpg)

可以看到我们左边`uniqueName`字段的值已经赋值到右边栈上0x12e4d8这里了，此时我们再来查看之前0x12E6D0,此时可以看到已经被覆盖为我们所伪造的地址了

![](http://imgsrc.baidu.com/forum/pic/item/8718367adab44aed9b8d9e2ef61c8701a08bfb3d.jpg)

此时我们可以看到旁边注释也是很清楚，返回到`icucnv36.4A80CB38`，此时我们跳转到该地址来查看一下函数是干啥的，以及我们为啥要修改之前的返回地址为他

![](http://imgsrc.baidu.com/forum/pic/item/6a600c338744ebf8ea5d47f59cf9d72a6159a7c8.jpg)

根据别的师傅的说法，这里是明显的利用`ROP`绕过`DEP`保护的手段，但是由于本人太菜不了解，所以下面来科普一下相关知识

### 1. GS保护

类似于`linux`上的`canary`,函数执行前存放在返回值与`ebp`上（低地址），然后当我们程序执行完毕之后会调用检查函数来判断该值是否与之前相同，因此我们此时就不能通过覆盖`ret`地址进行`ROP`链构造了，而是修改栈上保存的某一个函数指针来进行利用

`IDA`反汇编可以看到函数开始前会有如下指令：
