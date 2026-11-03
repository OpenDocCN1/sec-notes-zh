
```asm
.text:0000000000002D84 060                 ADD             X9, X9, W11,SXTH ; X9 = Xn + imm16
```

## 执行

经过二次解码分析得出目前有效的真实寄存器有：`x9`是源寄存器加上立即的一个偏移量，`w8`是目标操作数索引值。

### 3318：偏移量的内存中取值

### 3A70：计算目标寄存器在`pContext`的偏移地址，`x8= x21 + w8 * 8`

### 3A74：将内存值放入目标寄存器

```asm
.text:0000000000003318 060                 LDR             X9, [X9] 
.text:0000000000003A70 060                 ADD             X8, X21, W8,UXTW#3
.text:0000000000003A74 060                 STR             X9, [X8,#8]
```

### 指令还原

加载指令：`LDR Xt, [Xn + imm16]`

当`imm`为0时：`LDR Xt, [Xn]`

### 2> 还原

### 3> 虚拟机执行框架

分析完取指、解码、执行后大体得到了一个模糊的虚拟机执行框架，为了方便记忆和理解使用c伪代码来描述。

![](https://attach.52pojie.cn/forum/202406/26/172744hr8jj4zlq44qgb8b.png)

### 3> 解码op1 & op2

使用脚本解析`opcode`的`op1`和`op2`打印所有需要分析的handler。

```python
            import struct

            pcode_start = 0xB090

            pcode_size = 0x2D8

            with open("libEncryptor.so", "rb") as f:

                            content = f.read()

            bytes_code = content[pcode_start : pcode_start + pcode_size]

            insts_op1_sets = {}

            insts_op2_sets = {}

            for pc in range(0, pcode_size, 4):

                    word = struct.unpack(" 0:

                            opcode = word[0]

                            op1 = opcode & 0x3F

                            if op1 == 11:

                                    op2 = (opcode >> 6) & 0x3F

                                    if insts_op2_sets.get(op2, None) == None:

                                            insts_op2_sets[op2] = 1

                                    else:

                                            insts_op2_sets[op2] += 1

                            else:

                                    if insts_op1_sets.get(op1, None) == None:

                                            insts_op1_sets[op1] = 1

                                    else:

                                            insts_op1_sets[op1] += 1

            print("op1指令统计:")

            op1_sorted_dict = dict(sorted(insts_op1_sets.items(), key=lambda item: item[1], reverse=True))

            for inst, count in op1_sorted_dict.items():

                     print(f'op1: {inst}, count: {count}')

            print("op2指令统计:")

            op2_sorted_dict = dict(sorted(insts_op2_sets.items(), key=lambda item: item[1], reverse=True))

            for inst, count in op2_sorted_dict.items():

                     print(f'op1: 11, op2: {inst}, count: {count}')
```

一共打印出15个handler，好在需要分析还原的指令不多：

```text
op1指令统计:
op1: 23, count: 56
op1: 40, count: 38
op1: 21, count: 27
op1: 24, count: 2
op1: 12, count: 2
op1: 17, count: 2
op1: 48, count: 2
op1: 52, count: 1
op1: 7, count: 1
op2指令统计:
op1: 11, op2: 7, count: 25
op1: 11, op2: 43, count: 14
op1: 11, op2: 12, count: 8
op1: 11, op2: 25, count: 2
op1: 11, op2: 39, count: 1
op1: 11, op2: 62, count: 1
```

### 3> 还原脚本

在15个handler并还原后，现在重新编写脚本`decode_opcode.py`解码`opcode`，将打印还原的指令打印出伪汇编代码并保存文件`ttencryptor.asm`和`generate_aes_key_iv.asm`。

```python
        def decode(pcode_start, pcode_size, liner_disasm = False):

                with open("libEncryptor.so", "rb") as f:

                        content = f.read()

                print(f"{'pc':^8} {'指令':^8}  {'op1':^8}  {'op2':^8}\t{'助记符':^30}")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tx0, #0\t\t\t;x0始终为0，XZR寄存器?")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tx4, pArgs\t\t;参数列表指针")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tx5, #0")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tx6, pfn_external_func_list\t\t;外部函数列表指针")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tx7, pCallRegisterTrampolineFunction\t;保存跳转函数地址")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tx29, pVirualStackBottom;\t\t;虚拟机堆栈栈底")

                print(f"{'-'*8}  {'-'*8}  {'-'*8}  {'-'*8}  MOV\tlr, #0\t\t\t;x31=0")

                # 已解析的指令

                known_insns_op1 = {}        # dcode_insns_status=1

                known_insns_op2 = {}        # dcode_insns_status=2

                # 未解析的指令

                unknown_insts_op1 = set()   # dcode_insns_status=3

                unknown_insts_op2 = set()   # dcode_insns_status=4

                bytes_code = content[pcode_start : pcode_start + pcode_size]

                for pc in range(0, pcode_size, 4):

                        word = struct.unpack(" 0:

                                opcode = word[0]            

                                op1 = get_op1(opcode)

                                Xt = get_openand1(opcode)   # x8/w8

                                Xn = get_operand2(opcode)   # x9/w9    

                                Xm = get_operand3(opcode)   # x10/w10

                                X4 = get_operand4(opcode)   # x11/w11

                                match op1:

                                        case 11:

                                                Xt = get_operand3(opcode)   # x10/w10

                                                Xn = get_openand1(opcode)   # x8/w8

                                                Xm = get_operand2(opcode)   # x9/w9                 

                                                op2 = get_op2(opcode)                    

                                                match op2:

                                                        case 7:

                                                                dcode_insns_status=2

                                                                print(f"{pc:08X}  {opcode:08X}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tORR\t{get_regsiter_name(Xt)}, {get_regsiter_name(Xn)}, {get_regsiter_name(Xm)}")

                                                        case 12:

                                                                dcode_insns_status=2

                                                                print(f"{pc:08X}  {opcode:08X}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tADD\t{get_regsiter_name(Xt)}, {get_regsiter_name(Xn)}, {get_regsiter_name(Xm)}")

                                                        case 25:

                                                                dcode_insns_status=2

                                                                if X4 == 0:

                                                                        print(f"{pc:08X}  {opcode:08X}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tNOP\t\t\t\t;LSL\t{get_regsiter_name(Xt)}, {get_regsiter_name(Xn)}, #{X4}")

                                                                else:

                                                                        print(f"{pc:08X}  {opcode:08X}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tLSL)\t{get_regsiter_name(Xt)}, {get_regsiter_name(Xn)}, #{X4}")

                                                        case 39:

                                                                dcode_insns_status=2

                                                                print(f"{pc:08X}  {opcode:08X}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tCMP\t{get_regsiter_name(Xm)}, {get_regsiter_name(Xn)}")

                                                                print(f"{' '*8}  {' '*8}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tCSET\t{get_regsiter_name(Xt)}, CC")

                                                        case 43:

                                                                dcode_insns_status=2

                                                                if liner_disasm:

                                                                        print(f"{pc:08X}  {opcode:08X}  {opcode & 0x3F:02d}(0x{opcode & 0x3F:02X})  {op2:02d}(0x{op2:02X})\tBR\t{get_regsiter_name(Xm)}\t\t\t;LR={get_regsiter_name(Xt)}")

                                                                else:
```
