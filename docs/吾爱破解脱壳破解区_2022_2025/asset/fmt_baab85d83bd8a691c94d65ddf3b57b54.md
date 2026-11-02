
```
005160A8     | 8DB424 90000000        | lea esi,dword ptr ss:[esp+90]              | esi:EntryPoint
005160AF     | 90                     | nop                                        |
005160B0     | 8B16                   | mov edx,dword ptr ds:[esi]                 | edx:EntryPoint, esi:EntryPoint
005160B2     | 3B11                   | cmp edx,dword ptr ds:[ecx]                 | edx:EntryPoint, ecx:EntryPoint
005160B4     | 75 39                  | jne sourceinsight4_original.5160EF         |
005160B6     | 83E8 04                | sub eax,4                                  |
005160B9     | 83C1 04                | add ecx,4                                  | ecx:EntryPoint
005160BC     | 83C6 04                | add esi,4                                  | esi:EntryPoint
005160BF     | 83F8 04                | cmp eax,4                                  |
005160C2     | 73 EC                  | jae sourceinsight4.5160B0            |
005160C4     | 85C0                   | test eax,eax                         |
005160C6     | 74 20                  | je sourceinsight4.5160E8             |
005160C8     | 8A11                   | mov dl,byte ptr ds:[ecx]             | ecx:"C:\\ProgramData\\Source Insight\\4.0\\si4.lic"
005160CA     | 3A16                   | cmp dl,byte ptr ds:[esi]             |
005160CC     | 75 21                  | jne sourceinsight4.5160EF            |
005160CE     | 83F8 01                | cmp eax,1                            |
005160D1     | 76 15                  | jbe sourceinsight4.5160E8            |
005160D3     | 8A51 01                | mov dl,byte ptr ds:[ecx+1]           | ecx+1:":\\ProgramData\\Source Insight\\4.0\\si4.lic"
005160D6     | 3A56 01                | cmp dl,byte ptr ds:[esi+1]           |
005160D9     | 75 14                  | jne sourceinsight4.5160EF            |
005160DB     | 83F8 02                | cmp eax,2                            |
005160DE     | 76 08                  | jbe sourceinsight4.5160E8            |
005160E0     | 8A41 02                | mov al,byte ptr ds:[ecx+2]           | ecx+2:"\\ProgramData\\Source Insight\\4.0\\si4.lic"
005160E3     | 3A46 02                | cmp al,byte ptr ds:[esi+2]           |
005160E6     | 75 07                  | jne sourceinsight4.5160EF            |
005160E8     | BE 01000000            | mov esi,1                            |
005160ED     | EB 02                  | jmp sourceinsight4.5160F1            |
005160EF     | 33F6                   | xor esi,esi                          |
005160F1     | 55                     | push ebp                             |
005160F2     | E8 39EDF0FF            | call sourceinsight4.424E30           |
005160F7     | 83C4 04                | add esp,4                            |
005160FA     | 8BC6                   | mov eax,esi                          |
005160FC     | F7D8                   | neg eax                              |
005160FE     | 1BC0                   | sbb eax,eax                          |
00516100     | 25 FAFEFFFF            | and eax,FFFFFEFA                     |
00516105     | 5F                     | pop edi                              |
00516106     | 05 CE010000            | add eax,1CE                          |
0051610B     | 5D                     | pop ebp                              |
0051610C     | 5E                     | pop esi                              |
0051610D     | 81C4 24210000          | add esp,2124                         |
00516113     | C3                     | ret                                  |
```

这里我们将验证码比较部分，去除掉验证码不相同的判断。

```
005160B2     | 3B11                   | cmp edx,dword ptr ds:[ecx]                 | edx:EntryPoint, ecx:EntryPoint
005160B4     | 75 39                  | jne sourceinsight4_original.5160EF         |
```

改为

```
0051609F     | B8 80000000            | mov eax,80                           |
005160A4     | 8D4C24 10              | lea ecx,dword ptr ss:[esp+10]        |
005160A8     | 8DB424 90000000        | lea esi,dword ptr ss:[esp+90]        |
005160AF     | 90                     | nop                                  |
005160B0     | 8B16                   | mov edx,dword ptr ds:[esi]           |
005160B2     | 90                     | nop                                  |
005160B3     | 90                     | nop                                  |
005160B4     | 90                     | nop                                  |
005160B5     | 90                     | nop                                  |
005160B6     | 83E8 04                | sub eax,4                            |
005160B9     | 83C1 04                | add ecx,4                            |
005160BC     | 83C6 04                | add esi,4                            |
005160BF     | 83F8 04                | cmp eax,4                            |
005160C2     | 73 EC                  | jae sourceinsight4.5160B0            |
```

## 五、硬件设备的验证

### 硬件设备的检验的关键代码
