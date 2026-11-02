
```
000000006D525DC0 | 57                     | push rdi                                |
000000006D525DC1 | 56                     | push rsi                                |
000000006D525DC2 | 53                     | push rbx                                |
000000006D525DC3 | 48:83EC 20             | sub rsp,20                              |
000000006D525DC7 | 48:89CB                | mov rbx,rcx                             |
000000006D525DCA | 48:89D6                | mov rsi,rdx                             |
000000006D525DCD | E8 EED4D2FF            | call                   |
000000006D525DD2 | 48:85C0                | test rax,rax                            |
000000006D525DD5 | 74 18                  | je moretool.6D525DEF                    |
000000006D525DD7 | E8 E4D4D2FF            | call                   |
000000006D525DDC | 48:89C1                | mov rcx,rax                             |
000000006D525DDF | BA 1F000000            | mov edx,1F                              |
000000006D525DE4 | 4D:33C0                | xor r8,r8                               |
000000006D525DE7 | 4D:33C9                | xor r9,r9                               |
000000006D525DEA | E8 41DDD2FF            | call ======>这里反正就是发送窗口消息的意思
000000006D525DEF | E8 BC37D1FF            | call ==》或F7之后修改mov rdi,1; ret
000000006D525DF4 | 48:89C7                | mov rdi,rax                             |
000000006D525DF7 | 48:85FF                | test rdi,rdi                            |
000000006D525DFA | 74 62                  | je moretool.6D525E5E                    | =======》JMP  6D525E5E 也可以成功
000000006D525DFC | 48:89F9                | mov rcx,rdi                             |
000000006D525DFF | 48:8B15 F26CD3FF       | mov rdx,qword ptr ds:[6D25CAF8]         | 000000006D25CAF8:"浪%m"==&"`2'm"
000000006D525E06 | E8 05FFFFFF            | call             |
000000006D525E0B | 84C0                   | test al,al                              |
000000006D525E0D | 74 3F                  | je moretool.6D525E4E                    |
000000006D525E0F | 48:89F9                | mov rcx,rdi                             |
000000006D525E12 | 48:8B15 4F7DD3FF       | mov rdx,qword ptr ds:[]   | 000000006D25DB68:&"`2'm"
000000006D525E19 | E8 F2FEFFFF            | call             |
000000006D525E1E | 84C0                   | test al,al                              |
000000006D525E20 | 75 3C                  | jne moretool.6D525E5E                   |
000000006D525E22 | 48:83BB D0010000 00    | cmp qword ptr ds:[rbx+1D0],0            |
000000006D525E2A | 74 15                  | je moretool.6D525E41                    |
000000006D525E2C | 48:8D9B D0010000       | lea rbx,qword ptr ds:[rbx+1D0]          |
000000006D525E33 | 48:8B4B 08             | mov rcx,qword ptr ds:[rbx+8]            |
000000006D525E37 | 48:89F2                | mov rdx,rsi                             |
000000006D525E3A | 49:89F8                | mov r8,rdi                              |
000000006D525E3D | FF13                   | call qword ptr ds:[rbx]                 |
000000006D525E3F | EB 1D                  | jmp moretool.6D525E5E                   |
000000006D525E41 | 48:89D9                | mov rcx,rbx                             |
000000006D525E44 | 48:89FA                | mov rdx,rdi                             |
000000006D525E47 | E8 94060000            | call             | ========》NOP这里可以成功，或进入后的地方直接ret！
000000006D525E4C | EB 10                  | jmp moretool.6D525E5E                   | 回车之后来到了这里！
```

至此就算暴力法修复bug了。其他原因待考证。
![](https://static.52pojie.cn/static/image/smiley/laohu/laohu34.gif)
