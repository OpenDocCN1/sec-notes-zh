
```
#define V_EAX 0
#define V_EBX 1
#define V_ECX 2
#define V_EDX 3
#define V_ESI 4
#define V_EDI 5
#define V_EBP 6
#define V_ESP 7

UINT8 GetRegisterIndex(ZydisDecodedOperand& op) {
    switch (op.reg.value)
    {
        case ZYDIS_REGISTER_EAX:
            return V_EAX;
        case ZYDIS_REGISTER_EBX:
            return V_EBX;
        case ZYDIS_REGISTER_ECX:
            return V_ECX;
        case ZYDIS_REGISTER_EDX:
            return V_EDX;
        case ZYDIS_REGISTER_ESI:
            return V_ESI;
        case ZYDIS_REGISTER_EDI:
            return V_EDI;
        case ZYDIS_REGISTER_EBP:
            return V_EBP;
        case ZYDIS_REGISTER_ESP:
            return V_ESP;
    default:
        return 0xff;
    }
}

UINT8 GetMemRegisterIndex(ZydisDecodedOperand& op) {
    switch (op.mem.base)
    {
    case ZYDIS_REGISTER_EAX:
        return V_EAX;
    case ZYDIS_REGISTER_EBX:
        return V_EBX;
    case ZYDIS_REGISTER_ECX:
        return V_ECX;
    case ZYDIS_REGISTER_EDX:
        return V_EDX;
    case ZYDIS_REGISTER_ESI:
        return V_ESI;
    case ZYDIS_REGISTER_EDI:
        return V_EDI;
    case ZYDIS_REGISTER_EBP:
        return V_EBP;
    case ZYDIS_REGISTER_ESP:
        return V_ESP;
    default:
        return 0xff;
    }
}

DWORD WINAPI iat_search(LPVOID lpParam) {

    unsigned char data[16] = {0};

    EMULATOR_CPU_CONTEXT cpu_context = { 0 };

    UINT8 code[16] = { 0 };
    char lable[MAX_LABEL_SIZE] = { 0 };
    ZyanStatus status;
    ZydisDecoder decoder;
    UINT32 address = 0;
    duint count = 0;

    iatinfo iat_info;
    memset(&iat_info, 0, sizeof(iatinfo));
    duint current = vminfo.codebase;

    int callCount = 0;
    reset_module_map();
    g_call_map.clear();

    ZydisDecoderInit(&decoder, ZYDIS_MACHINE_MODE_LONG_COMPAT_32, ZYDIS_STACK_WIDTH_32);
    // 初始化格式化器
    ZydisFormatter formatter;
    ZydisFormatterInit(&formatter, ZYDIS_FORMATTER_STYLE_INTEL);
    // 反汇编上下文
    ZydisDecodedInstruction instruction;
    ZydisDecodedOperand operands[ZYDIS_MAX_OPERAND_COUNT];

    while (current (current)] = address;//另一个map
                            _plugin_logprintf("call address:%x %s\n", current, lable);

                        }
                    }
                    else if (operands[0].type == ZYDIS_OPERAND_TYPE_IMMEDIATE) {
                        if (DbgMemRead(current + 5 + operands[0].mem.base, data, sizeof(data)) && data[0] == 0x90) { //iat
                            Emulator(current, cpu_context, iat_info, vminfo);//模拟执行
                            g_iat_map[static_cast(current)] = iat_info;
                            callCount++;
                        }

                    }

                    break;
                }
                case ZYDIS_MNEMONIC_MOV: {
                    if (operands[0].type == ZYDIS_OPERAND_TYPE_REGISTER) {
                        UINT8 reg = GetRegisterIndex(operands[0]);
                        if (reg != 0xff && operands[1].type == ZYDIS_OPERAND_TYPE_MEMORY && \
                            operands[1].mem.base == ZYDIS_REGISTER_NONE) {
                            //mov reg32,[0x1001234]这种情况
                            memset(lable, 0, sizeof(lable));
                            if (DbgMemRead(operands[1].mem.disp.value, &address, sizeof(address)) && \
                                DbgGetLabelAt(address, SEG_DEFAULT, lable)) {
                                count++;
                                add_to_module_map(address);
                                g_mov_map[static_cast(current)] = address;
                                _plugin_logprintf("mov address:%x %s %x\n", current, lable, address);

                            }

                        }
                    }

                    break;
                }
                default:
                    break;
                }
                current += instruction.length;
            }
            else {
                current++;
            }
        }
    }
    _plugin_logprintf("iat number:%d\n", callCount);
    print_and_fill_iat(iataddress);
    return 0;
}
```

# 解释 `call [0x1001234]`（举例地址）这种情况

![](https://attach.52pojie.cn/forum/202507/08/193742xbsruruabfbaksr3.png)

796ea6f7-1191-4a63-918a-90e8c3b3b40a.png (203.36 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=Mjc4Nzk2MHxlYmRhNjA4ZnwxNzkzNTcwODM0fDE2ODA4NnwyMDQ0MzM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4Nzk2MHxlYmRhNjA4ZnwxNzkzNTcwODM0fDE2ODA4NnwyMDQ0MzM5&nothumb=yes)

## `0x1001240`地址处内容

2025-7-8 19:37 上传

注意看`0x1001240`处的内容,发现这是个api函数地址啊，查看引用

![](https://attach.52pojie.cn/forum/202507/08/194004zu1qceukkgl7qpnn.png)

43da471e-d6e5-4d03-aa98-26d38e7ce283.png (200.09 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=Mjc4Nzk2MXw0NWIxNDI3NnwxNzkzNTcwODM0fDE2ODA4NnwyMDQ0MzM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4Nzk2MXw0NWIxNDI3NnwxNzkzNTcwODM0fDE2ODA4NnwyMDQ0MzM5&nothumb=yes)

### `0x1001240`引用

2025-7-8 19:40 上传

有这么多处地方引用了,要么是`mov reg,[0x1001240]`, `call reg`,要么就是直接`call [0x1001240]`,那么我们直接`dump`放到其他机器肯定是不能运行的,因为函数地址不一样啊,所以这种情况要修复,将`0x1001240`修改成其他地址,即重建的`iat`地址

修复在后面,现在不提,这种情况我猜测是vmp没有把所有的导入表全部加密,留了几个在壳中事先解密出函数地址,回填到特定内存地址处

## 被加密的 API 函数

接下来看被加密的api函数 即`call xxxxxxxx`这种情况,基本逻辑如下(大致逻辑，并不准确)：

```
push reg
xchg [esp],reg
mov reg,imm32
mov reg,[reg+imm32]
lea reg,[reg+imm32]
ret
```

其中还夹杂一些`jmp`跳转以及一些垃圾指令,大概逻辑就是这么一个样子,我的方法是模拟执行,模拟这几种相关指令,遇到`ret`指令停止模拟,此时读取模拟的`esp`栈顶值,这个值可能是api函数地址,也可能不是,

为啥可能不是（还有可能被hook，对资源保护之后,解密出来是vm虚拟机,在这里不考虑这种情况）,看下面

```
0x10001233                                                push edi

0x10001234        mov edi,[0x1005678]                        call xxxxxxxx        此时模拟的esp栈顶值就是0x1001239而不是api函数地址 真正的api函数地址已经解密到edi寄存器中

                                        ------------>

0x10001239        call edi                                call edi
```

## 模拟执行代码

看模拟执行的代码，当然写的比较粗糙,只模拟重要指令,最终得到模拟栈顶`esp`存储的值

```
typedef struct _EMULATOR_CPU_CONTEXT {

    DWORD reg[8];

    DWORD eip; // 指令指针

    //DWORD eflags; 

    // 模拟内存块 (包含代码、数据、栈)

    //BYTE* mem_base;      // 分配的模拟内存起始地址

    //SIZE_T mem_size;      // 分配的模拟内存大小

} EMULATOR_CPU_CONTEXT,*PEMULATOR_CPU_CONTEXT;

UINT32 Emulator(UINT32 emulataddress, EMULATOR_CPU_CONTEXT& cpu_context, iatinfo& iat_info, vmpinfo& vminfo) {

    UINT32 emulatstack[64] = {0}; //模拟所需堆栈 模拟指令较少 因此栈不需要很大

    UINT8 code[16] = {0};

    UINT8 JmpFlag =  0 ;

    char buffer[256] = { 0 };

    UINT32 imm32 = 0;

    ZyanStatus status;

    UINT32 initaddress = emulataddress;

    cpu_context.reg[V_ESP] = (UINT32)&emulatstack[32];

    UINT32 init_vesp = cpu_context.reg[V_ESP];

    cpu_context.eip = emulataddress;

    // 初始化反汇编器（x86模式）

    ZydisDecoder decoder;

    ZydisDecoderInit(&decoder, ZYDIS_MACHINE_MODE_LONG_COMPAT_32, ZYDIS_STACK_WIDTH_32);

    // 初始化格式化器

    ZydisFormatter formatter;

    ZydisFormatterInit(&formatter, ZYDIS_FORMATTER_STYLE_INTEL);

    // 反汇编上下文

    ZydisDecodedInstruction instruction;

    ZydisDecodedOperand operands[ZYDIS_MAX_OPERAND_COUNT];

    while (1) {

        if (DbgMemRead(emulataddress, code, sizeof(code))) {

            status = ZydisDecoderDecodeFull(

                &decoder,

                code,

                sizeof(code),

                &instruction,

                operands

            );

            if (ZYAN_SUCCESS(status)) {

                ZydisFormatterFormatInstruction(&formatter, &instruction, operands, instruction.operand_count_visible, \

                    buffer, sizeof(buffer), ZYDIS_RUNTIME_ADDRESS_NONE, ZYAN_NULL);

                switch (instruction.mnemonic)

                {

                case ZYDIS_MNEMONIC_CALL: { //模拟call xxxxxxxx指令

                    if (operands[0].type == ZYDIS_OPERAND_TYPE_IMMEDIATE) {

                        cpu_context.reg[V_ESP] -= 4; //esp-4

                        *(UINT32*)cpu_context.reg[V_ESP] = emulataddress + instruction.length; //写入返回地址

                        cpu_context.eip += operands[0].imm.value.s + instruction.length; //修正eip
```
