
# 虚拟机反汇编分析

``` 
    push 0x402110
    push 0x92124021
    push 0x440841

    push ebx
    push ecx
    push ebp
    push eax
    push esi
    push edx
    push edi
    pushfd  
    mov ecx,0
    push ecx
    //mov esi, 0x4023a7
    mov edi,esp
    lea esp, ss:[esp-0xC0]  //  提升堆栈 为虚拟机分配所需内存

    lea ebp, ds:[0x00466596]    //  跳转基址

    mov eax, dword ptr ds:[edi]     //0 压入的ecx 
    add edi,4
    mov dword ptr ss:[esp+8], eax

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x2c], eax //初始context.eflags

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x24], eax //context.edi

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x18], eax //context.edx

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0], eax //context.esi

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x30], eax //context.eax

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x10], eax //context.ebp

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x14], eax //context.ecx

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x38], eax //context.ebx

    mov eax, dword ptr ds:[edi]     
    add edi,4
    mov dword ptr ss:[esp+0x20], eax //0x440841 最开始的call的返回,其实没啥用

    mov eax, dword ptr ds:[edi] 
    add edi,4
    mov dword ptr ss:[esp+0x1c], eax  ////0x92124021 用于解密opcode的数据,现在也没啥用

    //上面是保存现场环境
    mov edx,edi
    sub edi,4
    mov dword ptr ds:[edi], edx

    sub edi,4
    mov dword ptr ds:[edi],4
    mov ecx, dword ptr ds:[edi] //mov ecx,4
    mov edx, dword ptr ds:[edi+0x4]
    add ecx,edx
    mov dword ptr ds:[edi+0x4], ecx //ecx = old_esp 被vm之前的esp (第一条push指令(push 0x402110)前的esp) 感觉这几行代码完全没有意义

    pushfd
    pop dword ptr ss:[edi]
    mov eax, dword ptr ds:[edi]     //一个eflags的值 依旧没意义 

    add edi,4
    mov dword ptr ss:[esp+0x3c], eax //一个毫无意义的eflags，可能是为了占位esp+0x3c
    mov edi, dword ptr ds:[edi]     //edi = old_esp

    //解密得到imm32,解密步骤直接省略
    mov ecx,0x10000000
    sub edi,4
    mov dword ptr ds:[edi], ecx //ecx=0x10000000

    mov eax, dword ptr ds:[edi]
    mov dword ptr ss:[esp+0xc], eax     //0x10000000

    //解密得到imm32
    mov ecx,0x1234
    sub edi,4
    mov dword ptr ds:[edi], ecx

    //将解密得到的数据写入栈，覆盖之前的数据

    mov eax, dword ptr ss:[esp+0xc]     //  取出0x10000000
    sub edi,4
    mov dword ptr ds:[edi], eax

    mov ecx, dword ptr ds:[edi]     //0x10000000
    mov edx, dword ptr ds:[edi+0x4]     //0x1234
    add ecx, edx    //  求和
    mov dword ptr ds:[edi+0x4], ecx //保存求和后的结果到栈

    //由于add指令会影响eflgas寄存器 得到求和之后的eflags
    pushfd
    pop dword ptr ss:[edi]

    mov eax, dword ptr ds:[edi]     //求和之后的eflags
    add edi,4
    mov dword ptr ss:[esp+0x1c], eax //求和之后的eflags覆盖之前的垃圾数据(0x92124021) vmcontext.eflags

    mov eax, dword ptr ds:[edi] //求和后的结果
    add edi,4
    mov dword ptr ss:[esp+0x3c], eax    //将得到的0x10001234覆盖之前的无意义eflags 后续会发现esp+0x3c是vmcontext.eax

    mov eax, dword ptr ss:[esp+0x3c]
    sub edi,4
    mov dword ptr ds:[edi], eax //无意义的指令 因为[edi]此时的值就是0x10001234 可能单纯为了sub edi,4？猜测

    mov eax, dword ptr ss:[esp+0x10]   //vmcontext.ebp
    sub edi,4
    mov dword ptr ds:[edi], eax        //vmcontext.ebp
    sub edi,4

    //就是是解密imm32 可能突然觉得这个立即数毫无头绪，但是模拟的指令有一条是mov dword ptr ss:[ebp-0x18], eax
    //这样看是不是就有什么发现了
    mov ecx,0xFFFFFFE8

    mov dword ptr ds:[edi], ecx     //0xFFFFFFE8或者-0x18
    mov ecx, dword ptr ds:[edi]     
    mov edx, dword ptr ds:[edi+0x4] //  ebp
    add ecx,edx                     //得到ebp-0x18的地址
    mov dword ptr ds:[edi+0x4], ecx //保存ebp-0x18的地址

    pushfd
    pop dword ptr ss:[edi]  //上面有add指令，依旧是保存eflags,但是没啥用其实
    mov eax, dword ptr ds:[edi]     //eflags
    add edi,4
    mov dword ptr ss:[esp+0x4], eax //eflags

    mov edx, dword ptr ds:[edi]     //此时edx=ebp-0x18
    mov eax, dword ptr ds:[edi+0x4] //取出0x10001234
    mov dword ptr ss:[edx], eax //完成mov dword ptr ss:[ebp-0x18], eax的虚拟化

    //解密得到返回地址
    mov ecx,0040102E

    sub edi,4
    mov dword ptr ds:[edi], ecx    //此时ecx的值恰好是虚拟机返回地址

    //后面将虚拟寄存器转化成特殊的数据结构,从低地址到高地址分别是 
    /*
    eflags
    edi
    edx
    esi
    eax=0x10001234
    ebp
    ecx
    ebx
    0x40102e //解密得到的返回地址
    */
    //与后面的pop reg刚好是对应的

    mov eax, dword ptr ss:[esp+0x38] 
    sub edi,4
    mov dword ptr ds:[edi], eax //vmcontext.ebx

    mov eax, dword ptr ss:[esp+0x14] 
    sub edi,4
    mov dword ptr ds:[edi], eax //vmcontext.ecx

    mov eax, dword ptr ss:[esp+0x10] 
    sub edi,4
    mov dword ptr ds:[edi], eax //vmcontext.ebp

    mov eax, dword ptr ss:[esp+0x3c]    
    sub edi,4
    mov dword ptr ds:[edi], eax //应该是vmcontext.eax

    mov eax, dword ptr ss:[esp+0]       
    sub edi,4
    mov dword ptr ds:[edi], eax //vmcontext.esi

    mov eax, dword ptr ss:[esp+0x18]        
    sub edi, 0x4
    mov dword ptr ds:[edi], eax //vmcontext.edx

    mov eax, dword ptr ss:[esp+0x24]        
    sub edi,4
    mov dword ptr ds:[edi], eax //vmcontext.edi

    mov eax, dword ptr ss:[esp+0x1c]
    sub edi,4
    mov dword ptr ds:[edi], eax         //vmcontext.eflags

    mov esp,edi         //设置esp寄存器 准备恢复寄存器的值到真实寄存器

    //恢复现场
    popfd
    pop edi
    pop edx
    pop esi
    pop eax
    pop ebp
    pop ecx
    pop ebx
    ret 
```

//至此，整个虚拟化的分析大概完成

至此，整个虚拟化的流程大概分析完毕了，`vmp`就是边解密边执行再加上各种的垃圾指令很容易懵逼，刚开始接触`vm`，依旧有很多不足之处，仍需继续努力
