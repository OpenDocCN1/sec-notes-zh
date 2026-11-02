
## 反汇编片段

```
5FDA017B | E8 E0082A01           | call wechatwin.61040A60                          |
5FDA0180 | A1 B4A28D61           | mov eax,dword ptr ds:[0x618DA2B4]                | 618DA2B4:"凣疶"
5FDA0185 | 33C4                  | xor eax,esp                                      |
5FDA0187 | 898424 78120000       | mov dword ptr ss:[esp+0x1278],eax                |
5FDA018E | 8B45 0C               | mov eax,dword ptr ss:[ebp+0xC]                   |
5FDA0191 | 53                    | push ebx                                         |
5FDA0192 | 8B5D 10               | mov ebx,dword ptr ss:[ebp+0x10]                  |
5FDA0195 | 56                    | push esi                                         |
5FDA0196 | 8B75 14               | mov esi,dword ptr ss:[ebp+0x14]                  |
5FDA0199 | 57                    | push edi                                         |
5FDA019A | 8BF9                  | mov edi,ecx                                      |
5FDA019C | 895424 10             | mov dword ptr ss:[esp+0x10],edx                  |
5FDA01A0 | 57                    | push edi                                         |
5FDA01A1 | 894424 18             | mov dword ptr ss:[esp+0x18],eax                  |
5FDA01A5 | E8 06A8ED00           | call wechatwin.60C7A9B0                          |
5FDA01AA | 83C4 04               | add esp,0x4                                      |
5FDA01AD | 85C0                  | test eax,eax                                     |
5FDA01AF | 0F84 83020000         | je wechatwin.5FDA0438                            |
5FDA01B5 | 8D8424 78100000       | lea eax,dword ptr ss:[esp+0x1078]                |
5FDA01BC | B9 06000000           | mov ecx,0x6                                      |
5FDA01C1 | C600 FF               | mov byte ptr ds:[eax],0xFF                       |
5FDA01C4 | 8D40 10               | lea eax,dword ptr ds:[eax+0x10]                  |
5FDA01C7 | 83E9 01               | sub ecx,0x1                                      |
5FDA01CA | 75 F5                 | jne wechatwin.5FDA01C1                           |
5FDA01CC | 0F1045 18             | movups xmm0,xmmword ptr ss:[ebp+0x18]            |
5FDA01D0 | 8D4424 74             | lea eax,dword ptr ss:[esp+0x74]                  |
5FDA01D4 | 884C24 74             | mov byte ptr ss:[esp+0x74],cl                    |
5FDA01D8 | 894424 68             | mov dword ptr ss:[esp+0x68],eax                  |
5FDA01DC | 33C0                  | xor eax,eax                                      |
5FDA01DE | 0F118424 E0100000     | movups xmmword ptr ss:[esp+0x10E0],xmm0          |
5FDA01E6 | 894C24 6C             | mov dword ptr ss:[esp+0x6C],ecx                  |
5FDA01EA | 33D2                  | xor edx,edx                                      |
5FDA01EC | 0F1045 28             | movups xmm0,xmmword ptr ss:[ebp+0x28]            |
5FDA01F0 | 898C24 D8100000       | mov dword ptr ss:[esp+0x10D8],ecx                |
5FDA01F7 | 8D8C24 E0100000       | lea ecx,dword ptr ss:[esp+0x10E0]                |
5FDA01FE | C74424 70 00100000    | mov dword ptr ss:[esp+0x70],0x1000               |
5FDA0206 | 0F118424 F0100000     | movups xmmword ptr ss:[esp+0x10F0],xmm0          |
5FDA020E | 898424 DC100000       | mov dword ptr ss:[esp+0x10DC],eax                |
5FDA0215 | 0F1045 38             | movups xmm0,xmmword ptr ss:[ebp+0x38]            |
5FDA0219 | 0F118424 00110000     | movups xmmword ptr ss:[esp+0x1100],xmm0          |
5FDA0221 | 0F1045 48             | movups xmm0,xmmword ptr ss:[ebp+0x48]            |
5FDA0225 | 0F118424 10110000     | movups xmmword ptr ss:[esp+0x1110],xmm0          |
5FDA022D | 0F1045 58             | movups xmm0,xmmword ptr ss:[ebp+0x58]            |
5FDA0231 | 0F118424 20110000     | movups xmmword ptr ss:[esp+0x1120],xmm0          |
5FDA0239 | 0F1045 68             | movups xmm0,xmmword ptr ss:[ebp+0x68]            |
5FDA023D | 0F118424 30110000     | movups xmmword ptr ss:[esp+0x1130],xmm0          |
5FDA0245 | 8039 FF               | cmp byte ptr ds:[ecx],0xFF                       |
5FDA0248 | 74 25                 | je wechatwin.5FDA026F                            |
5FDA024A | 0F1001                | movups xmm0,xmmword ptr ds:[ecx]                 |
5FDA024D | 03C0                  | add eax,eax                                      |
5FDA024F | 42                    | inc edx                                          |
5FDA0250 | 83C1 10               | add ecx,0x10                                     |
5FDA0253 | 0F1184C4 78100000     | movups xmmword ptr ss:[esp+eax*8+0x1078],xmm0    |
5FDA025B | 8B8424 DC100000       | mov eax,dword ptr ss:[esp+0x10DC]                |
5FDA0262 | 40                    | inc eax                                          |
5FDA0263 | 898424 DC100000       | mov dword ptr ss:[esp+0x10DC],eax                |
5FDA026A | 83FA 06               | cmp edx,0x6                                      |
5FDA026D | 7C D6                 | jl wechatwin.5FDA0245                            |
5FDA026F | 56                    | push esi                                         |
5FDA0270 | 8D4C24 6C             | lea ecx,dword ptr ss:[esp+0x6C]                  |
5FDA0274 | E8 D7F9FFFF           | call wechatwin.5FD9FC50                          |
5FDA0279 | 6A 01                 | push 0x1                                         |
5FDA027B | 8D4C24 6C             | lea ecx,dword ptr ss:[esp+0x6C]                  |
5FDA027F | E8 DCF6FFFF           | call wechatwin.5FD9F960                          |
5FDA0284 | 8B4C24 6C             | mov ecx,dword ptr ss:[esp+0x6C]                  |
5FDA0288 | 8B4424 68             | mov eax,dword ptr ss:[esp+0x68]                  |
5FDA028C | C60401 0A             | mov byte ptr ds:[ecx+eax],0xA                    | A:'\n'
5FDA0290 | 8D4424 58             | lea eax,dword ptr ss:[esp+0x58]                  |
5FDA0294 | FF4424 6C             | inc dword ptr ss:[esp+0x6C]                      |
5FDA0298 | 50                    | push eax                                         |
5FDA0299 | FF15 00A33161         | call dword ptr ds:[]              |
5FDA029F | B8 67666666           | mov eax,0x66666667                               |
5FDA02A4 | 8BCF                  | mov ecx,edi                                      |
5FDA02A6 | F7EF                  | imul edi                                         |
5FDA02A8 | 53                    | push ebx                                         |
5FDA02A9 | D1FA                  | sar edx,0x1                                      |
5FDA02AB | 8BC2                  | mov eax,edx                                      |
5FDA02AD | C1E8 1F               | shr eax,0x1F                                     |
5FDA02B0 | 03C2                  | add eax,edx                                      |
5FDA02B2 | 8D0480                | lea eax,dword ptr ds:[eax+eax*4]                 |
5FDA02B5 | 2BC8                  | sub ecx,eax                                      |
5FDA02B7 | FF348D D0625B61       | push dword ptr ds:[ecx*4+0x615B62D0]             |
5FDA02BE | FF15 44A33161         | call dword ptr ds:[]        |
5FDA02C4 | 50                    | push eax                                         |
5FDA02C5 | 0FB74424 72           | movzx eax,word ptr ss:[esp+0x72]                 |
5FDA02CA | 50                    | push eax                                         |
5FDA02CB | 0FB74424 74           | movzx eax,word ptr ss:[esp+0x74]                 |
5FDA02D0 | 50                    | push eax                                         |
5FDA02D1 | 0FB74424 76           | movzx eax,word ptr ss:[esp+0x76]                 |
5FDA02D6 | 50                    | push eax                                         |
5FDA02D7 | 0FB74424 78           | movzx eax,word ptr ss:[esp+0x78]                 |
5FDA02DC | 50                    | push eax                                         |
```
