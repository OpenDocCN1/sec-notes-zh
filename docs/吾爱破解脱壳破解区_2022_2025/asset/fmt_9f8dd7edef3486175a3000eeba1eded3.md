
## 重定位的操作

```asm
004051CE            | 8B95 22040000      | mov edx,dword ptr ss:[ebp+422]      |
004051D4            | 8B85 2D050000      | mov eax,dword ptr ss:[ebp+52D]      | 这个基地址我不知道是怎么得到的，读取文件16进制也是这么多
004051DA            | 2BD0               | sub edx,eax                         | 看文章说是判断下当前的ImageBase 决定是否要进行重定位
004051DC            | 74 79              | je capturescreen.405257             |
004051DE            | 8BC2               | mov eax,edx                         | 下面是重定位的处理，由于没遇见过，所以就没有分析了
004051E0            | C1E8 10            | shr eax,10                          | eax:"USER32.dll"
004051E3            | 33DB               | xor ebx,ebx                         |
004051E5            | 8BB5 39050000      | mov esi,dword ptr ss:[ebp+539]      | ebp+539:sub_40553E+E
004051EB            | 03B5 22040000      | add esi,dword ptr ss:[ebp+422]      |
004051F1            | 833E 00            | cmp dword ptr ds:[esi],0            | esi:"`("
004051F4            | 74 61              | je capturescreen.405257             |
004051F6            | 8B4E 04            | mov ecx,dword ptr ds:[esi+4]        |
004051F9            | 83E9 08            | sub ecx,8                           |
004051FC            | D1E9               | shr ecx,1                           |
004051FE            | 8B3E               | mov edi,dword ptr ds:[esi]          | esi:"`("
00405200            | 03BD 22040000      | add edi,dword ptr ss:[ebp+422]      |
00405206            | 83C6 08            | add esi,8                           | esi:"`("
00405209            | 66:8B1E            | mov bx,word ptr ds:[esi]            | esi:"`("
0040520C            | C1EB 0C            | shr ebx,C                           |
0040520F            | 83FB 01            | cmp ebx,1                           |
00405212            | 74 0C              | je capturescreen.405220             |
00405214            | 83FB 02            | cmp ebx,2                           |
00405217            | 74 16              | je capturescreen.40522F             |
00405219            | 83FB 03            | cmp ebx,3                           |
0040521C            | 74 20              | je capturescreen.40523E             |
0040521E            | EB 2C              | jmp capturescreen.40524C            |
00405220            | 66:8B1E            | mov bx,word ptr ds:[esi]            | esi:"`("
00405223            | 81E3 FF0F0000      | and ebx,FFF                         |
00405229            | 66:01041F          | add word ptr ds:[edi+ebx],ax        |
0040522D            | EB 1D              | jmp capturescreen.40524C            |
0040522F            | 66:8B1E            | mov bx,word ptr ds:[esi]            | esi:"`("
00405232            | 81E3 FF0F0000      | and ebx,FFF                         |
00405238            | 66:01141F          | add word ptr ds:[edi+ebx],dx        |
0040523C            | EB 0E              | jmp capturescreen.40524C            |
0040523E            | 66:8B1E            | mov bx,word ptr ds:[esi]            | esi:"`("
00405241            | 81E3 FF0F0000      | and ebx,FFF                         |
00405247            | 01141F             | add dword ptr ds:[edi+ebx],edx      |
0040524A            | EB 00              | jmp capturescreen.40524C            |
0040524C            | 66:830E FF         | or word ptr ds:[esi],FFFF           | esi:"`("
00405250            | 83C6 02            | add esi,2                           | esi:"`("
00405253            | E2 B4              | loop capturescreen.405209           |
00405255            | EB 9A              | jmp capturescreen.4051F1            |
```
