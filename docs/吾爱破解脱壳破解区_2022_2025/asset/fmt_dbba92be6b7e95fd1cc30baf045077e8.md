
# [原创] IDA 9.1 & IDA 8.5 算法分析

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2017438
- 抓取时间: 2026-11-01T22:11:53

看到分享了 8.5 安装包，之前的 kg 失效了，才发现替换成了 9.x 的注册模式。

做了简单分析，整理如下：

调试版本为 9.0（240905），新注册机制都一样，`ida.dll` 中导出表搜索 `license`，`get_license_manager` 看到这个比较可疑，下断运行：
// `ida.dll` | `ModuleBase`:7FF896E00000 `SizeOfImage`:48B000

```
{
// IDA.DLL 
00007FF896E366F0 | 48:895C24 08         | mov qword ptr ss:[rsp+8],rbx            | 
00007FF896E366F5 | 48:897424 10         | mov qword ptr ss:[rsp+10],rsi           |
00007FF896E366FA | 48:897C24 18         | mov qword ptr ss:[rsp+18],rdi           |
...
00007FF896E36AE2 | 41:B9 10000000       | mov r9d,10                              |
00007FF896E36AE8 | 4C:8965 BF           | mov qword ptr ss:[rbp-41],r12           |
00007FF896E36AEC | 4C:8965 C7           | mov qword ptr ss:[rbp-39],r12           |
00007FF896E36AF0 | 4C:8965 CF           | mov qword ptr ss:[rbp-31],r12           |
00007FF896E36AF4 | 48:8B0D 4DE54100     | mov rcx,qword ptr ds:[7FF897255048]     |
00007FF896E36AFB | 48:8B01              | mov rax,qword ptr ds:[rcx]              | "D:\\Program Files\\IDA Professional 9.0\\ida.exe"
00007FF896E36AFE | 48:8D55 BF           | lea rdx,qword ptr ss:[rbp-41]           |
00007FF896E36B02 | 48:895424 20         | mov qword ptr ss:[rsp+20],rdx           |
00007FF896E36B07 | 4C:8D45 1F           | lea r8,qword ptr ss:[rbp+1F]            |
00007FF896E36B0B | 49:8BD5              | mov rdx,r13                             |
00007FF896E36B0E | FF50 18              | call qword ptr ds:[rax+18]              | 这里开始计算 CALL 00007FF8971333F0 【见下文函数1】
00007FF896E36B11 | 85C0                 | test eax,eax                            |
00007FF896E36B13 | 74 1A                | je ida.7FF896E36B2F                     |
...
}
```

```
{ // 【函数1】带函数读取 keyfile 并进行验证

// 00007FF896E36B0E         call qword ptr ds:[rax+18]  

00007FF8971333F0 | 40:55                | push rbp                                | 这里开始验证

00007FF8971333F2 | 53                   | push rbx                                |

00007FF8971333F3 | 56                   | push rsi                                |

...

00007FF897133B19 | 4C:8BC6              | mov r8,rsi                              | "D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic", 

00007FF897133B1C | 48:8D5424 38         | lea rdx,qword ptr ss:[rsp+38]           |

00007FF897133B21 | 48:8BCF              | mov rcx,rdi                             |

00007FF897133B24 | E8 37BEFFFF          | call              | 判断文件是否存在

{

        00007FF89712F9DB | 48:8B0E              | mov rcx,qword ptr ds:[rsi]              | "D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic"

        00007FF89712F9DE | E8 8D55FCFF          | call                     | 取文件名

        00007FF89712F9E3 | BA 2E000000          | mov edx,2E                              | 2E:'.'

        00007FF89712F9E8 | 48:8BC8              | mov rcx,rax                             | rcx:".hexlic", rax:".hexlic"

        00007FF89712F9EB | E8 4C2C0100          | call                      |

        00007FF89712F9F0 | 48:85C0              | test rax,rax                            | rax:".hexlic"

        00007FF89712F9F3 | 0F84 F2010000        | je ida.7FF89712FBEB                     |

        00007FF89712F9F9 | 48:8D15 08CE0900     | lea rdx,qword ptr ds:[7FF8971CC808]     | rdx:".hexlic"

        00007FF89712FA00 | 48:8BC8              | mov rcx,rax                             | rcx:".hexlic"

        00007FF89712FA03 | FF15 0F2F0600        | call qword ptr ds:[]          | 比较后缀名

}

00007FF897133B2B | 75 1E                | jne ida.7FF897133B4B                    |

00007FF897133B2D | 48:8B4C24 38         | mov rcx,qword ptr ss:[rsp+38]           |

00007FF897133B32 | E8 C9C8FBFF          | call                         |

00007FF897133B37 | 90                   | nop                                     |

00007FF897133B38 | 48:8BCB              | mov rcx,rbx                             |

00007FF897133B3B | E8 B06EFCFF          | call                 |

00007FF897133B40 | 90                   | nop                                     |

00007FF897133B41 | 44:8B6424 30         | mov r12d,dword ptr ss:[rsp+30]          |

00007FF897133B46 | E9 C7020000          | jmp ida.7FF897133E12                    |

00007FF897133B4B | 48:8D4424 38         | lea rax,qword ptr ss:[rsp+38]           |

00007FF897133B50 | 48:3BF0              | cmp rsi,rax                             |

00007FF897133B53 | 48:8B4424 40         | mov rax,qword ptr ss:[rsp+40]           |

00007FF897133B58 | 74 2D                | je ida.7FF897133B87                     |

00007FF897133B5A | 4C:8D40 FF           | lea r8,qword ptr ds:[rax-1]             | r8:&"{\n  "payload": {\n    "name": "Nisy/PYG",

00007FF897133B5E | 48:85C0              | test rax,rax                            |

00007FF897133B61 | B9 00000000          | mov ecx,0                               |

00007FF897133B66 | 4C:0F44C1            | cmove r8,rcx                            | r8:&"{\n  "payload": {\n    "name": "Nisy/PYG",

00007FF897133B6A | 4D:85C0              | test r8,r8                              | r8:&"{\n  "payload": {\n    "name": "Nisy/PYG\

00007FF897133B6D | 74 14                | je ida.7FF897133B83                     |

00007FF897133B6F | 48:8B5424 38         | mov rdx,qword ptr ss:[rsp+38]           |

00007FF897133B74 | 48:8BCE              | mov rcx,rsi                             |

00007FF897133B77 | E8 84EECCFF          | call ida.7FF896E02A00                   |

00007FF897133B7C | 48:8B4424 40         | mov rax,qword ptr ss:[rsp+40]           |

00007FF897133B81 | EB 04                | jmp ida.7FF897133B87                    |

00007FF897133B83 | 48:894E 08           | mov qword ptr ds:[rsi+8],rcx            |

00007FF897133B87 | 48:8D35 5DF90500     | lea rsi,qword ptr ds:[7FF8971934EB]     |

00007FF897133B8E | 4C:8BCE              | mov r9,rsi                              |

00007FF897133B91 | 48:85C0              | test rax,rax                            |

00007FF897133B94 | 4C:0F454C24 38       | cmovne r9,qword ptr ss:[rsp+38]         |

00007FF897133B9A | 4C:8D87 58010000     | lea r8,qword ptr ds:[rdi+158]           | r8:&"{\n  "payload": {\n    "name": "Nisy/PYG"

00007FF897133BA1 | 48:8D97 88000000     | lea rdx,qword ptr ds:[rdi+88]           |

00007FF897133BA8 | 4C:896C24 28         | mov qword ptr ss:[rsp+28],r13           |

00007FF897133BAD | 44:8B6424 34         | mov r12d,dword ptr ss:[rsp+34]          |

00007FF897133BB2 | 44:896424 20         | mov dword ptr ss:[rsp+20],r12d          |

00007FF897133BB7 | 48:8D4F 70           | lea rcx,qword ptr ds:[rdi+70]           |

00007FF897133BBB | E8 E094FFFF          | call              | 读取 KEYFILE 并进行验证【见下文函数2】

00007FF897133BC0 | 85C0                 | test eax,eax                            |

00007FF897133BC2 | 0F85 2C020000        | jne ida.7FF897133DF4                    |

00007FF897133BC8 | 41:F6C4 10           | test r12b,10                            |

00007FF897133BCC | 0F84 22020000        | je ida.7FF897133DF4                     |

...
}
```
