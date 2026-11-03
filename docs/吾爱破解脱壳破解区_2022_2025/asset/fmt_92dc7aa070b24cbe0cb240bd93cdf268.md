
```
0000000140035A8A | 48:85C0                  | test rax,rax                               |
0000000140035A8D | 75 07                    | jne usbtrace.140035A96                     |
0000000140035A8F | 33FF                     | xor edi,edi                                |
0000000140035A91 | E9 5E020000              | jmp usbtrace.140035CF4                     |
0000000140035A96 | 48:8D15 F3FA2100         | lea rdx,qword ptr [0x140255590]            | 
0000000140035A9D | 48:8BC8                  | mov rcx,rax                                | 
0000000140035AA0 | E8 2B4E1D00              | call                      |以"#"为分隔符 将分割出的第一个和snlicdat进行比较
0000000140035AA5 | 85C0                     | test eax,eax                               |
0000000140035AA7 | 74 07                    | je usbtrace.140035AB0                      |
0000000140035AA9 | 33FF                     | xor edi,edi                                |
0000000140035AAB | E9 44020000              | jmp usbtrace.140035CF4                     |
0000000140035AB0 | 4C:8D4424 60             | lea r8,qword ptr [rsp+0x60]                |
0000000140035AB5 | 48:8D5424 40             | lea rdx,qword ptr [rsp+0x40]               |
0000000140035ABA | 33C9                     | xor ecx,ecx                                | 
0000000140035ABC | E8 B3481D00              | call                    |
0000000140035AC1 | 48:8BD8                  | mov rbx,rax                                |
0000000140035AC4 | 48:85C0                  | test rax,rax                               |
0000000140035AC7 | 75 07                    | jne usbtrace.140035AD0                     |
0000000140035AC9 | 33FF                     | xor edi,edi                                |
0000000140035ACB | E9 24020000              | jmp usbtrace.140035CF4                     |
0000000140035AD0 | 4C:8BC0                  | mov r8,rax                                 |
0000000140035AD3 | BA 04010000              | mov edx,0x104                              | edx:"#"
0000000140035AD8 | 48:8D4D 90               | lea rcx,qword ptr [rbp-0x70]               |
0000000140035ADC | E8 2F351D00              | call                    |"将300024144拼接到一个缓冲区"
0000000140035AE1 | 4C:8D0D E0E52D00         | lea r9,qword ptr [0x1403140C8]             | r9:&"PE", 
0000000140035AE8 | 4C:8D15 F9E52D00         | lea r10,qword ptr [0x1403140E8]            | r10:NtReadFile+14
0000000140035AEF | 90                       | nop                                        |
0000000140035AF0 | 48:8BC3                  | mov rax,rbx                                |
0000000140035AF3 | 4D:8B01                  | mov r8,qword ptr [r9]                      | [r9]:"PE"
0000000140035AF6 | 4C:2BC3                  | sub r8,rbx                                 |
0000000140035AF9 | 0F1F80 00000000          | nop dword ptr [rax],eax                    |
0000000140035B00 | 0FB610                   | movzx edx,byte ptr [rax]                   | edx:"#"
0000000140035B03 | 42:0FB60C00              | movzx ecx,byte ptr [rax+r8]                | 
0000000140035B08 | 2BD1                     | sub edx,ecx                                | edx:"#", 
0000000140035B0A | 75 07                    | jne usbtrace.140035B13                     |
0000000140035B0C | 48:FFC0                  | inc rax                                    |
0000000140035B0F | 85C9                     | test ecx,ecx                               | 
0000000140035B11 | 75 ED                    | jne usbtrace.140035B00                     |
0000000140035B13 | 85D2                     | test edx,edx                               | edx:"#"
0000000140035B15 | 74 16                    | je usbtrace.140035B2D                      |
0000000140035B17 | 49:83C1 08               | add r9,0x8                                 | r9:&"PE"
0000000140035B1B | 4D:3BCA                  | cmp r9,r10                                 | r9:&"PE", r10:NtReadFile+14
0000000140035B1E | 7C D0                    | jl usbtrace.140035AF0                      |
0000000140035B20 | 397424 68                | cmp dword ptr [rsp+0x68],esi               |
0000000140035B24 | 75 07                    | jne usbtrace.140035B2D                     |
0000000140035B26 | 33FF                     | xor edi,edi                                |
0000000140035B28 | E9 C7010000              | jmp usbtrace.140035CF4                     |
0000000140035B2D | 4C:8D4424 60             | lea r8,qword ptr [rsp+0x60]                |
0000000140035B32 | 48:8D5424 40             | lea rdx,qword ptr [rsp+0x40]               |
0000000140035B37 | 33C9                     | xor ecx,ecx                                | 
0000000140035B39 | E8 36481D00              | call                    |
0000000140035B3E | 48:8BD8                  | mov rbx,rax                                |
0000000140035B41 | 48:85C0                  | test rax,rax                               |
0000000140035B44 | 75 07                    | jne usbtrace.140035B4D                     |
0000000140035B46 | 33FF                     | xor edi,edi                                |
0000000140035B48 | E9 A7010000              | jmp usbtrace.140035CF4                     |
0000000140035B4D | 4C:8BC0                  | mov r8,rax                                 |
0000000140035B50 | BA 04010000              | mov edx,0x104                              | edx:"#"
0000000140035B55 | 48:8D4D 90               | lea rcx,qword ptr [rbp-0x70]               |
0000000140035B59 | E8 B2341D00              | call                    |"将TestUser连接到300024144的后面，最终这个字符串会进行加密运算"
0000000140035B5E | 4D:85ED                  | test r13,r13                               |
0000000140035B61 | 74 10                    | je usbtrace.140035B73                      |
0000000140035B63 | 4C:8BC3                  | mov r8,rbx                                 |
0000000140035B66 | BA 04010000              | mov edx,0x104                              | edx:"#"
0000000140035B6B | 49:8BCD                  | mov rcx,r13                                | 
0000000140035B6E | E8 39341D00              | call                    |
0000000140035B73 | 4C:8D4424 60             | lea r8,qword ptr [rsp+0x60]                |
0000000140035B78 | 48:8D5424 40             | lea rdx,qword ptr [rsp+0x40]               |
0000000140035B7D | 33C9                     | xor ecx,ecx                                | 
0000000140035B7F | E8 F0471D00              | call                    |
0000000140035B84 | 48:8BD8                  | mov rbx,rax                                |
0000000140035B87 | 4C:8BC0                  | mov r8,rax                                 |
0000000140035B8A | BA 04010000              | mov edx,0x104                              | edx:"#"
0000000140035B8F | 48:8D4D 90               | lea rcx,qword ptr [rbp-0x70]               |
0000000140035B93 | E8 78341D00              | call                    |"这里是22/11/2025日期，后面有个函数对这个字符串进行时间校验"
0000000140035B98 | 48:8BD3                  | mov rdx,rbx                                | 第四个#号
0000000140035B9B | 4C:8B6C24 70             | mov r13,qword ptr [rsp+0x70]               |
0000000140035BA0 | 49:8BCD                  | mov rcx,r13                                | 
0000000140035BA3 | E8 48FBFFFF              | call usbtrace.1400356F0                    |"校验函数 对上面分割出的时间22/11/2025进行校验"
0000000140035BA8 | 85C0                     | test eax,eax                               |
0000000140035BAA | 75 07                    | jne usbtrace.140035BB3                     |
0000000140035BAC | 33FF                     | xor edi,edi                                |
0000000140035BAE | E9 41010000              | jmp usbtrace.140035CF4                     |
0000000140035BB3 | 4C:8D4424 60             | lea r8,qword ptr [rsp+0x60]                |
0000000140035BB8 | 48:8D5424 40             | lea rdx,qword ptr [rsp+0x40]               |
```
