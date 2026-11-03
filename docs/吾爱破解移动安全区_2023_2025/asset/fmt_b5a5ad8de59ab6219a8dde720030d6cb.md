
# [Android 原创] 某手游xqtd的简单分析

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1792058
- 抓取时间：2026-11-01T21:42:49

闲着没事，看到有新游出来，于是简单的分析学习一下。

由于时间和水平有限，本文会存在诸多不足，欢迎补充指正。

样本：xqtd

加固：mtp

工具：`ida`、010、`jadx`、`frida`

## `libtprt.so`简单处理

### 寄存器间接跳转

- 第一种，流程很清晰，比较好处理

![0.png](https://attach.52pojie.cn/forum/202305/31/181926qftxb6qtzmzl4l4u.png)

`0.png`（144.89 KB，下载次数：0） 2023-5-31 18:19 上传 [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODY3NnwwNjBmZDMxM3wxNzkzNTY5MzY3fDE2ODA4NnwxNzkyMDU4&nothumb=yes)

- `ida` 脚本，如遇到报错需要修改脚本

```
import idautils
import idc 
import idaapi
import ida_ua
from keystone import *

g_reg = [0] * 40
reg_base = 129
g_cond_info = list()
ldr_reg = -1
add_reg = -1

ks = keystone.Ks(keystone.KS_ARCH_ARM64, keystone.KS_MODE_LITTLE_ENDIAN)

def get_opcode(ea):
    opcode = None
    disasm = idc.GetDisasm(ea)
    if disasm.find('LT') != -1:
        opcode = 'blt'
    elif disasm.find('EQ') != -1:
        opcode = 'beq'
    elif disasm.find('CC') != -1:
        opcode = 'bcc'
    elif disasm.find('GT') != -1:
        opcode = 'bgt'
    elif disasm.find('NE') != -1:
        opcode = 'bne'  
    elif disasm.find('GE') != -1:
        opcode = 'bge'  
    elif disasm.find('HI') != -1:
        opcode = 'bhi'  
    return opcode

def do_patch(patch_1, patch_2, opcode, cond_jmp_addr, uncond_jmp_addr):
    print("patch_1=0x%x patch_1=0x%x opcode=%s cond_jmp_addr=0x%x uncond_jmp_addr=0x%x" % (patch_1, patch_2, opcode, cond_jmp_addr, uncond_jmp_addr))

    jump_offset = " ({:d})".format(cond_jmp_addr - patch_1)
    repair_opcode = opcode + jump_offset
    encoding, count = ks.asm(repair_opcode)
    idaapi.patch_byte(patch_1, encoding[0])
    idaapi.patch_byte(patch_1 + 1, encoding[1])
    idaapi.patch_byte(patch_1 + 2, encoding[2])
    idaapi.patch_byte(patch_1 + 3, encoding[3])

    jump_offset = " ({:d})".format(uncond_jmp_addr - patch_2)
    repair_opcode = 'b' + jump_offset
    encoding, count = ks.asm(repair_opcode)
    idaapi.patch_byte(patch_2, encoding[0])
    idaapi.patch_byte(patch_2 + 1, encoding[1])
    idaapi.patch_byte(patch_2 + 2, encoding[2])
    idaapi.patch_byte(patch_2 + 3, encoding[3])

def do_deobf(ea):
    # 获取跳转条件
    opcode = get_opcode(ea)
    if opcode is None:
        print("opcode:unknown opcode 0x%x" % ea)
        return ea

    # 获取跳转信息
    cond_reg = -1
    uncond_reg = -1
    cond_data = -1
    uncond_data = -1
    mnem = idc.ida_ua.ua_mnem(ea)
    if mnem == 'CSEL':
        cond_reg = idc.get_operand_value(ea, 1)
        uncond_reg = idc.get_operand_value(ea, 2)
    elif mnem == 'CSET':
        cond_data = 1
        uncond_data = 0
    ea = idc.next_head(ea)

    # 获取LDR寄存器
    ldr_reg = -1
    lsl_value = -1
    mnem = idc.ida_ua.ua_mnem(ea)
    if mnem == 'LSL':
        lsl_value = idc.get_operand_value(ea, 2)
        ea = idc.next_head(ea)
        mnem = idc.ida_ua.ua_mnem(ea)

    if mnem != 'LDR':
        print("LDR:0x%x -> %s" % (ea, mnem))
        return ea

    operand_type = idc.get_operand_type(ea, 1)
    if operand_type == idc.o_phrase:
        insn = ida_ua.insn_t()
        ida_ua.decode_insn(insn, ea)
        ldr_reg = insn.Op2.reg
        if lsl_value == -1:
            lsl_value = insn.Op2.value
    else:
        return ea
    ea = idc.next_head(ea)

    # 获取ADD寄存器
    mnem = idc.ida_ua.ua_mnem(ea)
    if mnem == 'MOV':
        ea = idc.next_head(ea)
        mnem = idc.ida_ua.ua_mnem(ea)

    if mnem != 'ADD':
        print("ADD:0x%x -> %s" % (ea, mnem))
        return ea
    op_3 = idc.print_operand(ea, 2)
    op_3 = op_3[1:]
    ea = idc.next_head(ea)

    # 进行patch
    mnem = idc.ida_ua.ua_mnem(ea)
    if mnem != 'BR':
        print("BR:0x%x -> %s" % (ea, mnem))
        return ea

    #print('1 = %d 2 = %d 3 = 0x%x' % (g_reg[ldr_reg - reg_base],  g_reg[cond_reg - reg_base], g_reg[int(op_3)]))
    if cond_data != -1 and uncond_data != -1:
        print(lsl_value)
        cond_jmp_addr  = (idc.get_qword(g_reg[ldr_reg - reg_base] + (cond_data 
```

- 处理前

![1.png](https://attach.52pojie.cn/forum/202305/31/181943ttt6nvwn6zjptwmj.png)

`1.png`（42.65 KB，下载次数：1） 2023-5-31 18:19 上传 [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODY3N3w0MDQ1N2YwZXwxNzkzNTY5MzY3fDE2ODA4NnwxNzkyMDU4&nothumb=yes)

- 处理后

![2.png](https://attach.52pojie.cn/forum/202305/31/181948bq6vsyivp3c2wqi3.png)

`2.png`（192.45 KB，下载次数：1） 2023-5-31 18:19 上传 [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODY3OHxhNDg5MDNiMXwxNzkzNTY5MzY3fDE2ODA4NnwxNzkyMDU4&nothumb=yes)

- 第二种，有点像做了流水线优化，不太好处理

- 观察发现存在这种间接跳转的方法，其方法的流程并不复杂，将各种跳转`patch`掉即可

![3.png](https://attach.52pojie.cn/forum/202305/31/181954lz8m8hemjxwidm2e.png)

`3.png`（277.15 KB，下载次数：0） 2023-5-31 18:19 上传 [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODY3OXwzMDkwM2E3YXwxNzkzNTY5MzY3fDE2ODA4NnwxNzkyMDU4&nothumb=yes)

### 字符串解密

- `libtprt.so`中，部分字符串的解密，都是同一个算法，写了个简单脚本
- `libtersafe2.so`中其实也是一样的

```
import idautils

import idc 

import idaapi

decrypt_fun_list = dict()

def get_code_refs_to_list(addr):

    result = list(idautils.CodeRefsTo(addr, True))

    return result

def do_decrypt(src_addr, dst_addr, offset, eor_data, add_data, call_addr):

    flag = idc.get_wide_byte(src_addr)

    str_len = flag ^ (idc.get_wide_byte(src_addr + 1))

    final_str = ''

    for i in range(str_len):

        v4 = idc.get_wide_byte(src_addr + 2 + i)

        v5 = (flag + i) ^ eor_data

        final_str += chr((v4 ^ flag) & 0xff)

        flag = v5 + add_data

    decrypt_fun_list[call_addr] = final_str + "  " + str(hex(offset))

    print('decrypt_addr=0x%x offset=0x%x, final_str=%s' % (call_addr, offset, final_str))

def decrpt_str():

    decrypt_fun_list.clear()

    tmp_list = [7, 6, 5, 4, 3, 2, 1]

    fun_list = get_code_refs_to_list(0x106C54)

    if not fun_list:

        return

    for i in range(len(fun_list)):

        call_addr = fun_list[i]

        call_addr_start = idc.get_func_attr(call_addr, idc.FUNCATTR_START)

        caller_fun_list = get_code_refs_to_list(call_addr_start)

        for j in range(len(caller_fun_list)):

            call_decryptStr_addr = caller_fun_list[j]

            arg_addr = idc.prev_head(call_decryptStr_addr) 

            mnem = idc.ida_ua.ua_mnem(arg_addr)

            register = idc.get_operand_value(arg_addr, 0)

            while True:

                if (register == 129) and (mnem == 'MOV'):

                    break

                arg_addr = idc.prev_head(arg_addr)

                mnem = idc.ida_ua.ua_mnem(arg_addr)

                register = idc.get_operand_value(arg_addr, 0)     

            offset = idc.get_operand_value(arg_addr, 1)

            if offset == 0xa0:

                offset = 0

            src_addr = 0x1250D8 + offset

            dst_addr = 0x15E778 + offset

            index = offset % 100

            eor_data = index

            add_data = tmp_list[(index % 7)]

            do_decrypt(src_addr, dst_addr, offset, eor_data, add_data, call_decryptStr_addr)

    pass

def main():

    decrpt_str()

if __name__ == "__main__":

    main()
```

- 解密结果，还是暴露了不少关键信息的

![4.png](https://attach.52pojie.cn/forum/202305/31/182000fju9vvjgvt99ctk2.png)

`4.png`（241.71 KB，下载次数：0） 2023-5-31 18:20 上传 [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYxODY4MHwyYzQ3YjE3YnwxNzkzNTY5MzY3fDE2ODA4NnwxNzkyMDU4&nothumb=yes)

### ollvm混淆

- 不算太严重，不是很影响分析，也就没有处理，将就着看了

## `libtprt.so`检测

### hook检测
