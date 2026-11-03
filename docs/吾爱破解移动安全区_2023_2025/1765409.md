
# [iOS 原创] 【iOS逆向与安全】利用IDAPython插件提高反汇编和逆向工程效率

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1765409
- 抓取时间: 2026-11-01T21:45:37

## 前言

IDA Python是一个用Python语言编写的插件，它为IDA Pro提供了可扩展性和自动化脚本支持。使用IDA Python，可以以更快、更简单的方式完成反汇编和逆向工程任务。

## 1、目标

快速了解和开始使用idapython。

## 2、开发环境和工具清单

- mac系统
- IDA Pro7.0

## 3、常用API

`idaapi`模块：该模块提供了许多核心的IDA API，包括：
- `idaapi.get_func(ea)`：获取给定地址处的函数对象
- `idaapi.get_segm_by_name(name)`：获取指定名称的段对象
- `idaapi.get_screen_ea()`：获取当前屏幕上显示的地址
- `idaapi.get_func_name(ea)`：获取给定地址处的函数名
- `idaapi.get_name_ea_simple(name)`：获取具有给定名称的地址
- `idaapi.asktext(defval, prompt)`：显示一个文本框，等待用户输入文本
- `idaapi.get_func_offset(ea)`：获取指定地址相对于函数起始地址的偏移量
- `idaapi.get_segment_name(ea)`：获取指定地址所在的段名称

`idaapi.idc`模块：该模块提供了一些旧版IDA API的Python封装，包括：
- `idc.GetDisasm(ea)`：获取给定地址处的反汇编指令
- `idc.Jump(ea)`：跳转到指定地址
- `idc.SetColor(ea, what, color)`：设置给定地址的颜色
- `idc.MakeCode(ea)`：将给定地址处的字节转换为指令
- `idc.MakeName(ea, name)`：将给定地址处的符号名称更改为给定名称
- `idc.get_inf_attr(idc.INF_MIN_EA)`：获取载入程序的最小的有效地址
- `idc.get_inf_attr(idc.INF_MAX_EA)`：获取载入程序的最大的有效地址
- `idc.print_insn_mnem(ea)`：获取给定地址处的助记符
- `idc.prev_head(ea)`：获取给定地址的上一条指令的地址
- `idc.print_operand(ea,index)`：获取给定地址中的操作数
- `idc.PatchByte(ea, 0x90)`：修改给定地址的第一个字节
- `idc.PatchWord(ea, 0x9090)`：修改给定地址的前两个字节
- `idc.PatchDword(ea, 0x90909090)`：修改给定地址的前四个字节
- `idc.PatchQword(ea,0x9090909090909090)`：修改给定地址的前八个字节

`idautils`模块：该模块提供了一些常用的IDA辅助函数，包括：
- `idautils.Functions()`：返回当前程序中所有函数的迭代器
- `idautils.Segments()`：返回当前程序中所有段的迭代器
- `idautils.Strings()`：返回当前程序中所有字符串的迭代器
- `idautils.XrefsFrom(ea)`：返回指向给定地址的所有交叉引用的迭代器
- `idautils.Heads(ea,ea)`：获取指定地址段的汇编指令

## 4、使用示例

移除SVC指令：

需要被移除的汇编指令如下：
```
MOV X0, #0x1F
MOV X1, #0
MOV X2, #0
MOV X3, #0
MOV W16, #0x1A
SVC 0x80
```

idapython脚本如下：
```
import idautils

import idc

# 获取二进制文件的起始地址和结束地址

start_addr = idc.get_inf_attr(idc.INF_MIN_EA)

end_addr = idc.get_inf_attr(idc.INF_MAX_EA)

print("Start")

# 遍历二进制文件中的所有指令

for addr in idautils.Heads(start_addr, end_addr):

    # 判断当前指令是否为svc指令

    mnem = idc.print_insn_mnem(addr)

    if mnem == "SVC":

        # 判断svc指令的前五行是否是mov指令

        prev1_addr = idc.prev_head(addr)

        prev2_addr = idc.prev_head(prev1_addr)

        prev3_addr = idc.prev_head(prev2_addr)

        prev4_addr = idc.prev_head(prev3_addr)

        prev5_addr = idc.prev_head(prev4_addr)

        if (

            idc.print_insn_mnem(prev1_addr) == "MOV"

            and idc.print_operand(prev1_addr, 0) == "W16"

            and idc.print_operand(prev1_addr, 1) == "#0x1A"

            and idc.print_insn_mnem(prev2_addr) == "MOV"

            and idc.print_operand(prev2_addr, 0) == "X3"

            and idc.print_operand(prev2_addr, 1) == "#0"

            and idc.print_insn_mnem(prev3_addr) == "MOV"

            and idc.print_operand(prev3_addr, 0) == "X2"

            and idc.print_operand(prev3_addr, 1) == "#0"

            and idc.print_insn_mnem(prev4_addr) == "MOV"

            and idc.print_operand(prev4_addr, 0) == "X1"

            and idc.print_operand(prev4_addr, 1) == "#0"

            and idc.print_insn_mnem(prev5_addr) == "MOV"

            and idc.print_operand(prev5_addr, 0) == "X0"

            and idc.print_operand(prev5_addr, 1) == "#0x1F"

        ):

            print '查找到一条MOV指令'

            # 将相关指令设为nop

            idc.PatchDword(addr, 0xd503201f)  # nop

            idc.PatchDword(prev1_addr, 0xd503201f)  # nop

            idc.PatchDword(prev2_addr, 0xd503201f)  # nop

            idc.PatchDword(prev3_addr, 0xd503201f)  # nop

            idc.PatchDword(prev4_addr, 0xd503201f)  # nop

            idc.PatchDword(prev5_addr, 0xd503201f)  # nop

print("End")
```

执行前：

![](https://raw.githubusercontent.com/witchan/pic/master/640.png)

按下图操作执行上边的脚本

![](https://raw.githubusercontent.com/witchan/pic/master/640-20230326190823740.png)

![](https://raw.githubusercontent.com/witchan/pic/master/640-20230326190844880.png)

执行后效果如下：

![](https://raw.githubusercontent.com/witchan/pic/master/640-20230326190910906.png)

## 总结

这篇文章小牛试刀的介绍如何在IDA中使用IDAPython插件，这只是一部分应用场景，期待大家继续探索更多用法。
