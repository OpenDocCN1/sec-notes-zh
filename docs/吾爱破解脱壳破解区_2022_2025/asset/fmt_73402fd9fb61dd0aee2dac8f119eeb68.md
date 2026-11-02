
```c
int __cdecl CheckActId(char *pData, int a2, size_t Size_4, int n_x32, int n_1B7F)
{
  int v5; // esi
  void *ppTable; // ebp
  int v7; // edi

  v5 = 0;
  ppTable = sub_403240(a2, Size_4, n_x32, n_1B7F);// 返回了存有很多的 ActId 的表
  if ( n_x32 = n_x32 )
        goto LABEL_4;
    }
    sub_425090(ppTable);
    ReleaseBlock(ppTable);
    return v7;
  }
}
```

## 修改 `si.lic` 中的 `ActId` 字段

```
ActId="673A44D35B3608E5C603D775C76216F555E000D04B6718E33E93F35887A8A360D2F468E313DC7B3E047E08F10A51B75561B5L55576B63BF2D7750F09557AF661F14849F94F2652A903A10E9074E61EA4FE7E83A6F5090AD61F3F365D1C67DA22A478FA17"
```

## 四、`si.lic` 文件检验码验证

读取`si.lic`文件的内容，去除掉空白字符，通过`call sourceinsight4.403210`生成校验码1，然后再使用去除掉空白字符的文件内容和 “`Value`” 字段的值通过`call sourceinsight4.402F00`函数生成校验码2，对比两个校验码。相同则验证通过，否则验证失败。

```
00515F90     | B8 24210000            | mov eax,2124                         |
00515F95     | E8 36850A00            | call sourceinsight4.5BE4D0           |
00515F9A     | 56                     | push esi                             |
00515F9B     | 8BB424 2C210000        | mov esi,dword ptr ss:[esp+212C]      |
00515FA2     | 68 A01F0000            | push 1FA0                            |
00515FA7     | 8D8424 8C010000        | lea eax,dword ptr ss:[esp+18C]       |
00515FAE     | 50                     | push eax                             |
00515FAF     | 56                     | push esi                             |
00515FB0     | E8 5BE6FFFF            | call sourceinsight4.514610           | 打开文件，并读取内容。
00515FB5     | 83C4 0C                | add esp,C                            |
00515FB8     | 3D C8000000            | cmp eax,C8                           |
00515FBD     | 0F85 49010000          | jne sourceinsight4.51610C            |
00515FC3     | 8B8C24 30210000        | mov ecx,dword ptr ss:[esp+2130]      |
00515FCA     | 51                     | push ecx                             | ecx:"C:\\ProgramData\\Source Insight\\4.0\\si4.lic"
00515FCB     | 8D9424 8C010000        | lea edx,dword ptr ss:[esp+18C]       |
00515FD2     | 52                     | push edx                             |
00515FD3     | E8 88F8FFFF            | call sourceinsight4.515860           | 文件内容追加 C 盘的 GUID 生成一个 20 位的校验值
00515FD8     | 56                     | push esi                             |
00515FD9     | E8 0257F4FF            | call sourceinsight4.45B6E0           | 将文件内容转换为 XML 格式
00515FDE     | 83C4 0C                | add esp,C                            |
00515FE1     | 85C0                   | test eax,eax                         |
00515FE3     | 75 0D                  | jne sourceinsight4.515FF2            |
00515FE5     | B8 CC010000            | mov eax,1CC                          |
00515FEA     | 5E                     | pop esi                              |
00515FEB     | 81C4 24210000          | add esp,2124                         |
00515FF1     | C3                     | ret                                  |
00515FF2     | 68 085E6000            | push sourceinsight4.605E08           | 605E08:"Signature"
00515FF7     | 8BC8                   | mov ecx,eax                          | ecx:"C:\\ProgramData\\Source Insight\\4.0\\si4.lic"
00515FF9     | E8 722AF4FF            | call sourceinsight4.458A70           | 判断是否有 "Sigature" 字段
00515FFE     | 8BF0                   | mov esi,eax                          |
00516000     | 85F6                   | test esi,esi                         |
00516002     | 74 E1                  | je sourceinsight4.515FE5             |
00516004     | 68 889D5E00            | push sourceinsight4.5E9D88           | 5E9D88:"Value"
00516009     | 8BCE                   | mov ecx,esi                          | ecx:"C:\\ProgramData\\Source Insight\\4.0\\si4.lic"
0051600B     | E8 4032F4FF            | call sourceinsight4.459250           | 检查 "Value" 是否存在
00516010     | 85C0                   | test eax,eax                         |
00516012     | 74 D1                  | je sourceinsight4.515FE5             |
00516014     | 55                     | push ebp                             |
00516015     | 57                     | push edi                             |
00516016     | 8B78 18                | mov edi,dword ptr ds:[eax+18]        |
00516019     | 8B46 20                | mov eax,dword ptr ds:[esi+20]        |
0051601C     | 8D8C24 90010000        | lea ecx,dword ptr ss:[esp+190]       |
00516023     | 51                     | push ecx                             | ecx:"C:\\ProgramData\\Source Insight\\4.0\\si4.lic"
00516024     | C68404 94010000 00     | mov byte ptr ss:[esp+eax+194],0      |
0051602C     | E8 AFE8F2FF            | call sourceinsight4.4448E0           | 拷贝一个新的文件数据
00516031     | 8BE8                   | mov ebp,eax                          |
00516033     | 83C4 04                | add esp,4                            |
00516036     | 85ED                   | test ebp,ebp                         |
00516038     | 75 0F                  | jne sourceinsight4.516049            |
0051603A     | 5F                     | pop edi                              |
0051603B     | 5D                     | pop ebp                              |
0051603C     | B8 EC010000            | mov eax,1EC                          |
00516041     | 5E                     | pop esi                              |
00516042     | 81C4 24210000          | add esp,2124                         |
00516048     | C3                     | ret                                  |
00516049     | 55                     | push ebp                             |
0051604A     | 8D9424 94010000        | lea edx,dword ptr ss:[esp+194]       |
00516051     | 68 0C606000            | push sourceinsight4.60600C           | 60600C:"\n\r\t "
00516056     | 52                     | push edx                             |
00516057     | E8 14EAF2FF            | call sourceinsight4.444A70           | 取出掉文件数据中的空白字符
0051605C     | 83C4 0C                | add esp,C                            |
0051605F     | 8D4424 10              | lea eax,dword ptr ss:[esp+10]        |
00516063     | 50                     | push eax                             |
00516064     | 68 80000000            | push 80                              |
00516069     | 68 B0070000            | push 7B0                             |
0051606E     | 55                     | push ebp                             |
0051606F     | E8 6C4E0A00            | call sourceinsight4.5BAEE0           | 算算新长度
00516074     | 83C4 04                | add esp,4                            |
00516077     | 40                     | inc eax                              |
00516078     | 50                     | push eax                             |
00516079     | 55                     | push ebp                             |
0051607A     | E8 91D1EEFF            | call sourceinsight4.403210           | 根据新的文件数据重新生成新的 20 位序列号
0051607F     | 8D4C24 20              | lea ecx,dword ptr ss:[esp+20]        |
00516083     | 51                     | push ecx                             | ecx:"C:\\ProgramData\\Source Insight\\4.0\\si4.lic"
00516084     | 8D9424 A8000000        | lea edx,dword ptr ss:[esp+A8]        |
0051608B     | 52                     | push edx                             |
0051608C     | 57                     | push edi                             |
0051608D     | E8 6ECEEEFF            | call sourceinsight4.402F00           | 根据 ["Value"] 的值和文件内容生成一个 20 位序列号
00516092     | 83C4 20                | add esp,20                           |
00516095     | 817C24 0C 80000000     | cmp dword ptr ss:[esp+C],80          |
0051609D     | 75 50                  | jne sourceinsight4.5160EF            |
0051609F     | B8 80000000            | mov eax,80                           |
005160A4     | 8D4C24 10              | lea ecx,dword ptr ss:[esp+10]        |
```
