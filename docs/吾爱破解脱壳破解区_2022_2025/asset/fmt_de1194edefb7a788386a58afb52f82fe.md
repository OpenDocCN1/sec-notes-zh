
```
0046BEA0  /$  55                    push ebp
0046BEA1  |.  8BEC                  mov ebp,esp
0046BEA3  |.  83EC 10               sub esp,10
0046BEA6  |.  53                    push ebx
0046BEA7  |.  56                    push esi
0046BEA8  |.  57                    push edi
0046BEA9  |.  894D F0               mov [local.4],ecx        ;  66:FF45 FC            /inc word ptr ss:[ebp-4] ;  0FBF45 FC              movsx eax,word ptr ss:[ebp-4]
0046BED8  |.  83F8 13               |cmp eax,13 ;^ E9 CDFFFFFF           \jmp HEROES.0046BED0        ;  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]        ;  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046BF32  |.  8B4D F0               mov ecx,[local.4]
0046BF35  |.  C64408 66 FF          mov byte ptr ds:[eax+ecx+66],0FF        ;  FF45 F8               /inc [local.2]        ;  837D F8 13             cmp [local.2],13
0046BF5D  |.  0F8D 33000000         |jge HEROES.0046BF96
0046BF63  |.  8B45 F8               |mov eax,[local.2]
0046BF66  |.  8B4D F0               |mov ecx,[local.4]
0046BF69  |.  8A4408 66             |mov al,byte ptr ds:[eax+ecx+66]
0046BF6D  |.  8B4D F8               |mov ecx,[local.2]
0046BF70  |.  8B55 F0               |mov edx,[local.4]
0046BF73  |.  884411 65             |mov byte ptr ds:[ecx+edx+65],al
0046BF77  |.  8B45 F8               |mov eax,[local.2]
0046BF7A  |.  8B4D F0               |mov ecx,[local.4]
0046BF7D  |.  8A8408 83000000       |mov al,byte ptr ds:[eax+ecx+83]
0046BF84  |.  8B4D F8               |mov ecx,[local.2]
0046BF87  |.  8B55 F0               |mov edx,[local.4]
0046BF8A  |.  888411 82000000       |mov byte ptr ds:[ecx+edx+82],al
0046BF91  |.^ E9 C0FFFFFF           \jmp HEROES.0046BF56
0046BF96  |>  8B45 F0               mov eax,[local.4]
0046BF99  |.  C640 78 FF            mov byte ptr ds:[eax+78],0FF        ;  E9 FF000000           jmp HEROES.0046C0AB
0046BFAC  |>  0FBE45 08             movsx eax,byte ptr ss:[ebp+8]
0046BFB0  |.  83F8 13               cmp eax,13
0046BFB3  |.  0F8C F2000000         jl HEROES.0046C0AB
0046BFB9  |.  0FBE45 08             movsx eax,byte ptr ss:[ebp+8]
0046BFBD  |.  83F8 1D               cmp eax,1D
0046BFC0  |.  0F8D E5000000         jge HEROES.0046C0AB
0046BFC6  |.  66:C745 FC 1300       mov word ptr ss:[ebp-4],13
0046BFCC  |.  E9 04000000           jmp HEROES.0046BFD5
0046BFD1  |>  66:FF45 FC            /inc word ptr ss:[ebp-4]        ;  0FBF45 FC              movsx eax,word ptr ss:[ebp-4]
0046BFD9  |.  83F8 1D               |cmp eax,1D
0046BFDC  |.  0F8D 22000000         |jge HEROES.0046C004
0046BFE2  |.  0FBF45 FC             |movsx eax,word ptr ss:[ebp-4]
0046BFE6  |.  8B4D F0               |mov ecx,[local.4]
0046BFE9  |.  0FBE4408 66           |movsx eax,byte ptr ds:[eax+ecx+66]
0046BFEE  |.  0FBE4D 08             |movsx ecx,byte ptr ss:[ebp+8]
0046BFF2  |.  3BC1                  |cmp eax,ecx
0046BFF4  |.  0F85 05000000         |jnz HEROES.0046BFFF
0046BFFA  |.  E9 05000000           |jmp HEROES.0046C004
0046BFFF  |>^ E9 CDFFFFFF           \jmp HEROES.0046BFD1
0046C004  |>  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046C008  |.  8B4D F0               mov ecx,[local.4]
0046C00B  |.  0FBE8408 83000000     movsx eax,byte ptr ds:[eax+ecx+83]
0046C013  |.  83F8 01               cmp eax,1
0046C016  |.  0F8E 13000000         jle HEROES.0046C02F
0046C01C  |.  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046C020  |.  8B4D F0               mov ecx,[local.4]
0046C023  |.  FE8C08 83000000       dec byte ptr ds:[eax+ecx+83]
0046C02A  |.  E9 7C000000           jmp HEROES.0046C0AB
0046C02F  |>  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046C033  |.  8B4D F0               mov ecx,[local.4]
0046C036  |.  C64408 66 FF          mov byte ptr ds:[eax+ecx+66],0FF
0046C03B  |.  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046C03F  |.  8B4D F0               mov ecx,[local.4]
0046C042  |.  C68408 83000000 00    mov byte ptr ds:[eax+ecx+83],0
0046C04A  |.  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046C04E  |.  40                    inc eax
0046C04F  |.  8945 F4               mov [local.3],eax
0046C052  |.  E9 03000000           jmp HEROES.0046C05A
0046C057  |>  FF45 F4               /inc [local.3]
0046C05A  |>  837D F4 1D             cmp [local.3],1D
0046C05E  |.  0F8D 33000000         |jge HEROES.0046C097
0046C064  |.  8B45 F4               |mov eax,[local.3]
0046C067  |.  8B4D F0               |mov ecx,[local.4]
0046C06A  |.  8A4408 66             |mov al,byte ptr ds:[eax+ecx+66]
0046C06E  |.  8B4D F4               |mov ecx,[local.3]
0046C071  |.  8B55 F0               |mov edx,[local.4]
0046C074  |.  884411 65             |mov byte ptr ds:[ecx+edx+65],al
0046C078  |.  8B45 F4               |mov eax,[local.3]
0046C07B  |.  8B4D F0               |mov ecx,[local.4]
0046C07E  |.  8A8408 83000000       |mov al,byte ptr ds:[eax+ecx+83]
0046C085  |.  8B4D F4               |mov ecx,[local.3]
0046C088  |.  8B55 F0               |mov edx,[local.4]
0046C08B  |.  888411 82000000       |mov byte ptr ds:[ecx+edx+82],al
0046C092  |.^ E9 C0FFFFFF           \jmp HEROES.0046C057
0046C097  |>  8B45 F0               mov eax,[local.4]
0046C09A  |.  C680 82000000 FF      mov byte ptr ds:[eax+82],0FF
0046C0A1  |.  8B45 F0               mov eax,[local.4]
0046C0A4  |.  C680 9F000000 00      mov byte ptr ds:[eax+9F],0
0046C0AB  |>  E9 00000000           jmp HEROES.0046C0B0
0046C0B0  |>  5F                    pop edi
0046C0B1  |.  5E                    pop esi
0046C0B2  |.  5B                    pop ebx
0046C0B3  |.  C9                    leave
0046C0B4  \.  C2 0400               retn 4
```

至此，我们清楚了魔法使用子程序的程序原理流程：

1. 首先根据使用的魔法代码，判断是战斗魔法还是冒险魔法，对这2类魔法分别处理，处理方式完全相同
2. 其次，拿使用的魔法代码去比对魔法书中每个位置的魔法代码，如果一样，就找到了正在使用的魔法在魔法书中位置了
3. 根据这个位置，再看对应的使用次数是否小于等于1。是的话，要再处理；不是的话就简单啦，把使用次数减1即可
4. 下面都是使用次数小于等于1的情况：先把这个魔法代码改成FF，使用次数改成0
5. 然后把魔法书中在使用魔法后面的所有魔法都前移一位，使用次数也一样前移一位
6. 最后把魔法书中最后位置的魔法代码设成FF，最后位置的使用次数设成0

至此，我们完全清楚了魔法在魔法书中消失的原因，下面就是怎么修改的问题了。首先，我们的目的是使用完次数后这个魔法不能再使用，但在访问公会或者神龛之后，这个魔法还会出现。因此，这个魔法要求要做到2点：一是在魔法书中不出现；二是魔法还在魔法书中，只是使用次数没有了而看不见。

## 五、使用魔法子程序的代码修改

在搞懂了魔法使用的子程序代码流程后，下面就是代码修改了，也就是重新设计这个子程序！设计程序的流程不是很难的活，但要思想清晰、每种可能的情况都要考虑到。

我们先来分析一下可能会出现的情况，当然要先清楚目标是什么！

目标：魔法在使用完后，从魔法书中暂时消失，在英雄访问公会或者神龛（这是魔法写入子程序的任务）后再现。

实现方式：使用完的魔法代码保留，将其次数改写成0，而将其在魔法书中的位置移到魔法书中现有魔法的最后位置。

可能遇到的情况：

1. 魔法书中只有一个魔法，自然它就是魔法书中的最后位置，因此，使用完，只要将其次数改成0，位置无须移动。
2. 魔法书中最后一个魔法，同（1）。
3. 在魔法书中的第13（十六进制）或者1D位置，自然也不需移动位置，但判断条件不同于（1）和（2）
4. 一般要移动的情况。

流程（本来要画图的，但我很久就不用word之类的了，就用文字描述了）：

1. 将使用的魔法代码与13比较，根据大于等于13或者小于13来判断是战斗还是冒险魔法
2. 战斗类，循环起始值设0，冒险类，循环起始值设13
3. 循环查找使用魔法在魔法书中位置：
    （a）首先看位置是否已经超过最大位置了，若是，跳出循环
    （b）比对使用魔法代码与查找位置的魔法代码，不一致，位置加1继续查找；一致则循环结束
4. 找到位置后，查看使用魔法的次数是否小于等于1。若不是，将使用次数减1，跳到子程序结束处；若是，则下面继续
5. 把使用完的魔法次数改成0
6. 检查位置是否是最后，若是，跳到子程序结束处；若不是，则下面继续
7. 循环：将后一位的魔法和次数都前移一位。每次循环都要判断：后面还有没有魔法以及是否是最后位置2种情况
8. 循环中首先判断后面的位置是否是最后位置了，若是，跳出循环，处理最后位置情况
9. 循环中再判断后面的位置是否已经没有魔法了，即代码是否是FF，若是，跳出循环，处理最后一步
10. 最后一步：把使用的魔法代码存入该位置，把对应的次数改成0

根据上述流程，利用OllyDBG将魔法使用子程序修改成下面样子即可（如果大家已经看懂上面原程序代码的分析，下面修改后的代码就很容易理解了）：

```
0046BEA0  /$  55                   push ebp
0046BEA1  |.  8BEC                 mov ebp,esp
0046BEA3  |.  83EC 10              sub esp,10
0046BEA6  |.  53                   push ebx
0046BEA7  |.  56                   push esi
0046BEA8  |.  57                   push edi
0046BEA9  |.  894D F0              mov [local.4],ecx
0046BEAC  |.  0FBE45 08            movsx eax,byte ptr ss:[ebp+8]
0046BEB0  |.  3C 13                cmp al,13
0046BEB2  |.  0F8D 0B000000        jge HEROES.0046BEC3
0046BEB8  |.  66:C745 FC 0000      mov word ptr ss:[ebp-4],0
0046BEBE  |.  E9 0F000000          jmp HEROES.0046BED2
0046BEC3  |>  66:C745 FC 1300      mov word ptr ss:[ebp-4],13
0046BEC9  |.  E9 04000000          jmp HEROES.0046BED2
0046BECE  |>  66:FF45 FC           /inc word ptr ss:[ebp-4]
0046BED2  |>  0FBF45 FC             movsx eax,word ptr ss:[ebp-4]
0046BED6  |.  3C 1D                |cmp al,1D
```
