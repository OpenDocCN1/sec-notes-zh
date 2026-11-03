
```asm
    push 0x402110
    push 0x92124021
    call 0x41542b

0x41542b:
    push ebx
    push ecx
    push ebp
    push eax
    push esi
    push edx
    push edi
    push eflags

    mov ecx,0
    push ecx
    mov esi,dword ptr ds:[esp+0x28]  //此时esi的值是0x92124021
    dec esi
    xor esi,0x7733592f
    inc esi
    xor esi,0x66bf7d8f
    rol esi,3
    sub esi,0x1cb30155   //上面操作对esi进行解密 此时esi=0x4023a7 
    lea esi,dword ptr ds:[esi+ecx]

    mov edi,esp         
    lea esp,dword ptr ss:[esp-0xc0]
    mov ebx,
    sub ebx,ecx
    lea ebp,dword ptr ds:[0x466596]
    sub esi,0x4
    mov eax,[esi]
    xor eax,ebx
    not eax
    dec eax
    ror eax,0x3
    not eax
    bswap eax
    lea eax, dword ptr ds:[eax-0x62320A2C]
    xor ebx,eax
    add ebp,eax
    jmp ebp   //此时ebp = 0x42ca8c  等价于jmp 0x42ca8c

0x42ca8c:
    mov eax,dword ptr ds:[edi]
    add edi,0x4
    lea esi,dword ptr ds:[esi-1]
    movzx ecx, byte ptr ds:[esi]
    xor cl,bl
    inc cl
    neg cl
    add cl,0xB
    ror cl,0x1
    sub cl,0x8e
    neg cl
    xor cl,0x54
    dec cl
    xor bl,cl

    mov dword ptr ss:[esp+ecx], eax     
    sub esi,0x4
    mov ecx, dword ptr ds:[esi]
    xor ecx, ebx
    sub ecx, 0x3E850F9
    rol ecx, 0x1
    sub ecx, 0x6BA107A2
    xor ecx, 0x53345A
    lea ecx, ds:[ecx+0x53205C24]
    neg ecx
    add ecx, 0x177B784E
    rol ecx, 0x1
    xor ebx, ecx
    add ebp,ecx
    push ebp   
    ret       //跳转到0x40d855处

0x40d855:
    mov eax, dword ptr ds:[edi] //[edi]的内容是初始时压栈的eflags寄存器的值 后续会保存到虚拟机栈中
    add edi,4                   
    lea esi, ds:[esi-0x1]
    movzx ecx, byte ptr ds:[esi]
    xor cl, bl
    inc cl
    neg cl
    add cl, 0xB
    ror cl, 0x1
    sub cl, 0x8E
    neg cl
    xor cl,0x54
    dec cl
    xor bl,cl
    mov dword ptr ss:[esp+ecx], eax //ecx=0x2c 保存真实的elags寄存器到虚拟机的elags寄存器，初始化虚拟机环境 vmcontext.eflags
    sub esi,4
    mov ecx, dword ptr ds:[esi]
    xor ecx, ebx
    sub ecx, 0x3E850F9
    rol ecx, 0x1
    lea ecx, ds:[ecx-0x6BA107A2]
    xor ecx, 0x53345A
    lea ecx, ds:[ecx+0x53205C24]
    neg ecx
    add ecx, 0x177B784E
    rol ecx, 0x1
    xor ebx, ecx
    add ebp,ecx
    jmp ebp //  此时ebp=0x0044FA3F

0x0044FA3F:
    mov eax, dword ptr ds:[edi]  //[edi]的内容是初始时压栈的edi寄存器的值 后续会保存到虚拟机栈中
    add edi,4
    sub esi, 0x1
    movzx ecx, byte ptr ds:[esi]
    xor cl, bl
    inc cl
    neg cl
    add cl,0xB
    ror cl,1
    sub cl,0x8e
    neg cl
    xor cl,0x54
    dec cl
    xor bl, cl
    mov dword ptr ss:[esp+ecx], eax //ecx=0x24 保存原始edi vmcontext.edi
    sub esi,4
    mov ecx, dword ptr ds:[esi]
    xor ecx, ebx
    sub ecx, 0x3E850F9
    rol ecx, 0x1
    sub ecx, 0x6BA107A2
    xor ecx, 0x53345A
    lea ecx, ds:[ecx+0x53205C24]
    neg ecx
    add ecx, 0x177B784E
    rol ecx, 0x1
    xor ebx, ecx
    add ebp, ecx
    jmp ebp //ebp=0x004710C6
```

依旧是部分指令，在这里我们可以先划出堆栈图，此时有朋友应该能发现刚开始压入的`0x92124021`不是随便的，而是会通过这个值进行解密得到`opcode`尾地址，并且复制给`esi`寄存器，为啥是尾地址呢，因为后续操作都是`sub esi,4`
`sub esi,1` `lea esi,[esi-4]`,`lea esi,[esi-1]` 其中这两条`lea`指令是和`sub`指令等价的。还有两条指令也是我们需要注意的，`lea esp,dword ptr ss:[esp-0xc0]` //提升堆栈,为虚拟机分配堆栈和环境，`lea ebp,dword ptr ds:[0x466596]` // 可以将`ebp`理解为基址，因为在后面都是加`ebp`实现跳转，在得到`opcode`尾地址之后，就开始对`opcode`进行访问，然后解密，解密完成加上`ebp`得到最终要跳转的地址。

![](https://attach.52pojie.cn/forum/202505/18/192543z1c2eexyyy6jez16.png)

dbb8bd34-79b4-40ea-9f78-f40b68f90a19.png (23.7 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3NzkyM3xhOWJiYmEyOHwxNzkzNTcwOTY4fDE2ODA4NnwyMDMyMzQz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3NzkyM3xhOWJiYmEyOHwxNzkzNTcwOTY4fDE2ODA4NnwyMDMyMzQz&nothumb=yes)

堆栈图

2025-5-18 19:25 上传

再仔细观察一下，应该会想到这些指令看着很复杂，但是终究就是算跳转地址，所做的真正有用的指令是有限的，比如`mov dword ptr ss:[esp+ecx], eax` //`ecx=0x2c` 保存真实的`elags`寄存器到虚拟机的`elags`寄存器，初始化虚拟机环境 `vmcontext.eflags`这些指令，边做完有用的指令边计算跳转，那么在这条指令中`ecx`又是怎么来的呢，在上面所述的代码中`ecx`是一个偏移量，也是通过`opcoede`进行动态解密得到的，看下面的堆栈图

![](https://attach.52pojie.cn/forum/202505/18/194156hszrq0sj7zhjah7k.png)

78db96cc-c591-4b7f-b423-d78417e17061.png (43.21 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=Mjc3NzkyNHw5MTM2ZWIyMnwxNzkzNTcwOTY4fDE2ODA4NnwyMDMyMzQz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3NzkyNHw5MTM2ZWIyMnwxNzkzNTcwOTY4fDE2ODA4NnwyMDMyMzQz&nothumb=yes)

堆栈图2

2025-5-18 19:41 上传

这些偏移都是通过`opcode`计算得出，理解成虚拟机的虚拟环境，一个特别的数据结构，这么一说是不是感觉有种拨开云雾的感觉，然后我跟着虚拟机的解密写了一部分的解密，太多了实现不想写了，因为原理就是这样了，解密`opcode`,得到跳转地址，偏移啥的

```c
#include 

#include

UINT32 init_esi;

UINT32 init_ebp = 0x466596;

UINT32 CalcEsiBase(UINT32 key, UINT32 ecx) {
    key--;                  // dec esi
    key ^= 0x7733592f;      // xor esi, 0x7733592f
    key++;                  // inc esi
    key ^= 0x66bf7d8f;      // xor esi, 0x66bf7d8f
    key = (key > (32 - 3));  // rol esi, 3
    key -= 0x1cb30155;      // sub esi, 0x1cb30115
    return key + ecx;       // lea esi, [esi+ecx]
}

UINT32 CalcJmpAddress(UINT32& ebx, UINT32& opcode_ptr, UINT32& initial_ebp, UINT32 ecx) {
    ebx -= ecx;
    opcode_ptr -= 4;                         // sub esi, 4
    UINT32 eax = *reinterpret_cast(opcode_ptr); // mov eax, [esi] (读取 DWORD)
    eax ^= ebx;                       // xor eax, ebx
    eax = ~eax;                       // not eax
    eax--;                            // dec eax
    eax = (eax >> 3) | (eax (opcode_ptr); // mov ecx, [esi]
    ecx ^= ebx;                    // xor ecx, ebx
    ecx -= 0x3E850F9;              // sub ecx, 0x3E850F9
    ecx = (ecx > 31); // rol ecx, 1
    ecx -= 0x6BA107A2;             // sub ecx, 0x6BA107A2
    ecx ^= 0x53345A;               // xor ecx, 0x53345A
    ecx += 0x53205C24;             // lea ecx, [ecx+0x53205C24]
    ecx = (~ecx) + 1; ;                    // neg ecx
    ecx += 0x177B784E;             // add ecx, 0x177B784E
    ecx = (ecx > 31); // rol ecx, 1
    ebx ^= ecx;                    // xor ebx, ecx
    initial_ebp += ecx;            // add ebp, ecx
    return initial_ebp;            // 返回更新后的 ebp
}

UINT32 CalcEbpAddress(UINT32& ebx, UINT32& opcode_ptr, UINT32& initial_ebp) {
    opcode_ptr -= 4;               // sub esi, 4
    UINT32 eax = *reinterpret_cast(opcode_ptr); // mov eax, [esi]
    eax ^= ebx;                    // xor eax, ebx
    eax = (eax >> 3) | (eax (opcode_ptr); // mov eax, [esi]
    eax ^= ebx;                    // xor eax, ebx
    eax = (eax > (32 - 3)); // rol eax, 3
    eax++;                         // inc eax
    eax = _byteswap_ulong(eax);    // bswap eax (字节序反转)
    eax = (eax > (32 - 2)); // rol eax, 0x2
    ebx ^= eax;                    // xor ebx, eax
    initial_ebp += eax;            // add ebp, eax
    return initial_ebp;            // 返回更新后的 ebp
}

UINT32 CalcEbpAddress3(UINT32& ebx, UINT32& opcode_ptr, UINT32& initial_ebp) {
    opcode_ptr -= 4;               // sub esi, 4
    UINT32 ecx = *reinterpret_cast(opcode_ptr); // mov ecx, [esi]
    ecx ^= ebx;                    // xor ecx, ebx
    ecx -= 0x7F4B67A2;             // sub ecx, 0x7F4B67A2
    ecx = ~ecx;                    // not ecx
    ecx--;                         // dec ecx
    ecx = ~ecx;                    // not ecx
    ebx ^= ecx;                    // xor ebx, ecx
    initial_ebp += ecx;            // add ebp, ecx
    return initial_ebp;            // 返回更新后的 ebp
}

UINT32 CalcEbpAddress4(UINT32& ebx, UINT32& opcode_ptr, UINT32& initial_ebp) {
    opcode_ptr -= 4;
    UINT32 ecx = *reinterpret_cast(opcode_ptr);
    ecx ^= ebx;
    ecx = (ecx > 30);
    ecx += 0x3C7C310B;
    ecx = ~ecx + 1;
    ecx =  _byteswap_ulong(ecx);
    ecx--;
    ebx ^= ecx;
    initial_ebp += ecx;
    return initial_ebp;
}

UINT32 CalcImm32(UINT32& ebx, UINT32& opcode_ptr) {
    opcode_ptr -= 4;               // sub esi, 0x4
    UINT32 ecx = *reinterpret_cast(opcode_ptr); // mov ecx, [esi]
    ecx ^= ebx;                    // xor ecx, ebx
    ecx--;                         // dec ecx
    ecx = _byteswap_ulong(ecx);    // bswap ecx (字节序反转)
```
