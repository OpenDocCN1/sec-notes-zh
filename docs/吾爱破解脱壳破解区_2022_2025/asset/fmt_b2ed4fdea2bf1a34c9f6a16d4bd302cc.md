
# 反汇编分析

```
0066FC4F  | 55                  | push ebp ==>这里是代码的头部
0066FC50  | 8BEC                | mov ebp,esp                         |
0066FC52  | 83C4 A8             | add esp,FFFFFFA8                    |
0066FC55  | B8 FC976D00         | mov eax,idr.6D97FC                  |
0066FC5A  | 53                  | push ebx                            |
0066FC5B  | 56                  | push esi                            |
0066FC5C  | 57                  | push edi                            |
0066FC5D  | E8 825CFFFF         | call                |
0066FC62  | FF75 08             | push dword ptr ss:[ebp+8]           |
0066FC65  | E8 4CFFFFFF         | call                |
0066FC6A  | 59                  | pop ecx                             |
0066FC6B  | 25 FF000000         | and eax,FF                          |
0066FC70  | 83F8 03             | cmp eax,3                           |
0066FC73  | 7C 05               | jl idr.66FC7A                       |
0066FC75  | 83F8 0F             | cmp eax,F                           |
0066FC78  | 7E 10               | jle idr.66FC8A                      |
0066FC7A  | 33C0                | xor eax,eax                         |
0066FC7C  | 8B55 B0             | mov edx,dword ptr ss:[ebp-50]       |
0066FC7F  | 6467:8916 0000      | mov dword ptr fs:[0],edx            |
0066FC85  | E9 B0030000         | jmp idr.67003A                      |
0066FC8A  | 8B34C5 D4966D00     | mov esi,dword ptr ds:[eax*8+6D96D4] |
0066FC91  | 83C0 FD             | add eax,FFFFFFFD                    |
0066FC94  | 83F8 0B             | cmp eax,B                           | B:'\v'
0066FC97  | 0F87 C5020000       | ja idr.66FF62                       |
0066FC9D  | FF2485 A4FC6600     | jmp dword ptr ds:[eax*4+66FCA4]     |
0066FCA4  | D4 FC               | aam FC                              |
0066FCA6  | 66:0010             | add byte ptr ds:[eax],dl            |
0066FCA9  | FD                  | std                                 |
0066FCAA  | 66:004CFD 66        | add byte ptr ss:[ebp+edi*8+66],cl   |
0066FCAF  | 0088 FD6600C4       | add byte ptr ds:[eax-3BFF9903],cl   |
0066FCB5  | FD                  | std                                 |
0066FCB6  | 66:0000             | add byte ptr ds:[eax],al            |
0066FCB9  | FE                  | ???                                 |
0066FCBA  | 66:003CFE           | add byte ptr ds:[esi+edi*8],bh      |
0066FCBE  | 66:0062 FF          | add byte ptr ds:[edx-1],ah          |
0066FCC2  | 66:0078 FE          | add byte ptr ds:[eax-2],bh          |
0066FCC6  | 66:00B4FE 6600F0FE  | add byte ptr ds:[esi+edi*8-10FFF9A] |
0066FCCE  | 66:0029             | add byte ptr ds:[ecx],ch            |
0066FCD1  | FF66 00             | jmp dword ptr ds:[esi]              | [esi]:sub_401000
0066FCD4  | 66:C745 C0 0800     | mov word ptr ss:[ebp-40],8          |
0066FCDA  | BA 5CA16D00         | mov edx,idr.6DA15C                  | 6DA15C:"EDivByZero"
0066FCDF  | 8D45 FC             | lea eax,dword ptr ss:[ebp-4]        | [ebp-4]:&"剂c"
0066FCE2  | E8 DD370000         | call                |
0066FCE7  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FCEA  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FCEC  | B2 01               | mov dl,1                            |
0066FCEE  | A1 68C46400         | mov eax,dword ptr ds:[64C468]       | 0064C468:"茨d"
0066FCF3  | E8 24250000         | call                |
0066FCF8  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FCFB  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FCFE  | 8D45 FC             | lea eax,dword ptr ss:[ebp-4]        | [ebp-4]:&"剂c"
0066FD01  | BA 02000000         | mov edx,2                           |
0066FD06  | E8 A5390000         | call                |
0066FD0B  | E9 72020000         | jmp idr.66FF82                      |
0066FD10  | 66:C745 C0 1400     | mov word ptr ss:[ebp-40],14         |
0066FD16  | BA 67A16D00         | mov edx,idr.6DA167                  | 6DA167:"ERangeError"
0066FD1B  | 8D45 F8             | lea eax,dword ptr ss:[ebp-8]        |
0066FD1E  | E8 A1370000         | call                |
0066FD23  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FD26  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FD28  | B2 01               | mov dl,1                            |
0066FD2A  | A1 C0C46400         | mov eax,dword ptr ds:[64C4C0]       | 0064C4C0:&"\vERangeErrord興"
0066FD2F  | E8 CC10EDFF         | call                |
0066FD34  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FD37  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FD3A  | 8D45 F8             | lea eax,dword ptr ss:[ebp-8]        |
0066FD3D  | BA 02000000         | mov edx,2                           |
0066FD42  | E8 69390000         | call                |
0066FD47  | E9 36020000         | jmp idr.66FF82                      |
0066FD4C  | 66:C745 C0 2000     | mov word ptr ss:[ebp-40],20         | 20:' '
0066FD52  | BA 73A16D00         | mov edx,idr.6DA173                  | 6DA173:"EIntOverflow"
0066FD57  | 8D45 F4             | lea eax,dword ptr ss:[ebp-C]        |
0066FD5A  | E8 65370000         | call                |
0066FD5F  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FD62  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FD64  | B2 01               | mov dl,1                            |
0066FD66  | A1 18C56400         | mov eax,dword ptr ds:[64C518]       | 0064C518:&"\fEIntOverflow悙惱興"
0066FD6B  | E8 58250000         | call                |
0066FD70  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FD73  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FD76  | 8D45 F4             | lea eax,dword ptr ss:[ebp-C]        |
0066FD79  | BA 02000000         | mov edx,2                           |
0066FD7E  | E8 2D390000         | call                |
0066FD83  | E9 FA010000         | jmp idr.66FF82                      |
0066FD88  | 66:C745 C0 2C00     | mov word ptr ss:[ebp-40],2C         | 2C:','
0066FD8E  | BA 80A16D00         | mov edx,idr.6DA180                  | 6DA180:&"1-0.dll"
0066FD93  | 8D45 F0             | lea eax,dword ptr ss:[ebp-10]       |
0066FD96  | E8 29370000         | call                |
0066FD9B  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FD9E  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FDA0  | B2 01               | mov dl,1                            |
0066FDA2  | A1 CCC56400         | mov eax,dword ptr ds:[64C5CC]       | 0064C5CC:&"\nEInvalidOp恜芼"
0066FDA7  | E8 C8250000         | call                |
0066FDAC  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FDAF  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FDB2  | 8D45 F0             | lea eax,dword ptr ss:[ebp-10]       |
0066FDB5  | BA 02000000         | mov edx,2                           |
0066FDBA  | E8 F1380000         | call                |
0066FDBF  | E9 BE010000         | jmp idr.66FF82                      |
0066FDC4  | 66:C745 C0 3800     | mov word ptr ss:[ebp-40],38         | 38:'8'
0066FDCA  | BA 8BA16D00         | mov edx,idr.6DA18B                  | 6DA18B:"EZeroDivide"
0066FDCF  | 8D45 EC             | lea eax,dword ptr ss:[ebp-14]       |
0066FDD2  | E8 ED360000         | call                |
0066FDD7  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FDDA  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FDDC  | B2 01               | mov dl,1                            |
0066FDDE  | A1 24C66400         | mov eax,dword ptr ds:[] | 0064C624:&"\vEZeroDivide绕d"
0066FDE3  | E8 38260000         | call                |
0066FDE8  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FDEB  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FDEE  | 8D45 EC             | lea eax,dword ptr ss:[ebp-14]       |
0066FDF1  | BA 02000000         | mov edx,2                           |
0066FDF6  | E8 B5380000         | call                |
0066FDFB  | E9 82010000         | jmp idr.66FF82                      |
0066FE00  | 66:C745 C0 4400     | mov word ptr ss:[ebp-40],44         | 44:'D'
0066FE06  | BA 97A16D00         | mov edx,idr.6DA197                  | 6DA197:"EOverflow"
0066FE0B  | 8D45 E8             | lea eax,dword ptr ss:[ebp-18]       |
0066FE0E  | E8 B1360000         | call                |
0066FE13  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FE16  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FE18  | B2 01               | mov dl,1                            |
0066FE1A  | A1 7CC66400         | mov eax,dword ptr ds:[] | 0064C67C:&"\tEOverflow悙 莇"
0066FE1F  | E8 A8260000         | call                |
0066FE24  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FE27  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FE2A  | 8D45 E8             | lea eax,dword ptr ss:[ebp-18]       |
0066FE2D  | BA 02000000         | mov edx,2                           |
0066FE32  | E8 79380000         | call                |
0066FE37  | E9 46010000         | jmp idr.66FF82                      |
0066FE3C  | 66:C745 C0 5000     | mov word ptr ss:[ebp-40],50         | 50:'P'
0066FE42  | BA A1A16D00         | mov edx,idr.6DA1A1                  | 6DA1A1:"EUnderflow"
0066FE47  | 8D45 E4             | lea eax,dword ptr ss:[ebp-1C]       |
0066FE4A  | E8 75360000         | call                |
0066FE4F  | FF45 CC             | inc dword ptr ss:[ebp-34]           |
0066FE52  | 8B08                | mov ecx,dword ptr ds:[eax]          |
0066FE54  | B2 01               | mov dl,1                            |
0066FE56  | A1 D4C66400         | mov eax,dword ptr ds:[] | 0064C6D4:&"\nEUnderflow恱莇"
0066FE5B  | E8 18270000         | call                |
0066FE60  | 8945 A8             | mov dword ptr ss:[ebp-58],eax       |
0066FE63  | FF4D CC             | dec dword ptr ss:[ebp-34]           |
0066FE66  | 8D45 E4             | lea eax,dword ptr ss:[ebp-1C]       |
0066FE69  | BA 02000000         | mov edx,2                           |
0066FE6E  | E8 3D380000         | call                |
0066FE73  | E9 0A010000         | jmp idr.66FF82                      |
0066FE78  | 66:C745 C0 5C00     | mov word ptr ss:[ebp-40],5C         | 5C:'\\'
0066FE7E  | BA ACA16D00         | mov edx,idr.6DA1AC                  | ==>EAccessViolation==》我们当前在这里！
```

出头部引用点出来后，就会来到下面的位置处：

我们试着给上面的条件跳转改成强跳！

`0067129D  | 74 29               | je idr.6712C8     ===>EB 29`

然后，我们再去保存。。经过漫长的等待就会成功了。
