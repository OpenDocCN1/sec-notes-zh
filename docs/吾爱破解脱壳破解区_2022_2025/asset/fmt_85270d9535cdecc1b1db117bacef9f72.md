
这是一段 IDA 反汇编日志，展示了某个程序验证 license 文件的过程。日志主要包含两个关键函数：函数4（进入关键算法）和函数5（注册算法验证）。

```bash
00007FF8970FADB0 | 48:0F455424 20       | cmovne rdx,qword ptr ss:[rsp+20]        | [qword ptr ss:[rsp+20]]:&"D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic"

00007FF8970FADB6 | 48:85C0              | test rax,rax                            |

00007FF8970FADB9 | 48:0F454C24 38       | cmovne rcx,qword ptr ss:[rsp+38]        |

00007FF8970FADBF | 44:8BC6              | mov r8d,esi                             |

00007FF8970FADC2 | FF15 107A0900        | call qword ptr ds:[]          |  调用 CreateFileW 打开 license

}

00007FF8970FAE89 | 48:8BE8              | mov rbp,rax                             |

00007FF8970FAE8C | 48:85C0              | test rax,rax                            |

00007FF8970FAE8F | 0F84 BA000000        | je ida.7FF8970FAF4F                     |

00007FF8970FAE95 | 48:8BC8              | mov rcx,rax                             |

00007FF8970FAE98 | 40:32F6              | xor sil,sil                             |

00007FF8970FAE9B | E8 D0120000          | call                        |

00007FF8970FAEA0 | 48:8BD8              | mov rbx,rax                             |

00007FF8970FAEA3 | 48:85C0              | test rax,rax                            |

00007FF8970FAEA6 | 7E 68                | jle ida.7FF8970FAF10                    |

00007FF8970FAEA8 | 49:3BC6              | cmp rax,r14                             |

00007FF8970FAEAB | 7F 63                | jg ida.7FF8970FAF10                     |

00007FF8970FAEAD | 48:8BD0              | mov rdx,rax                             | rdx:"D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic"

00007FF8970FAEB0 | 49:8BCF              | mov rcx,r15                             | r15:&"D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic"

00007FF8970FAEB3 | E8 482FD1FF          | call              |

00007FF8970FAEB8 | 49:8B17              | mov rdx,qword ptr ds:[r15]              | rdx:"D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic", 

00007FF8970FAEBE | 48:8BCD              | mov rcx,rbp                             |

00007FF8970FAEC1 | E8 7A040000          | call                        | 读取到 RDX 中

00007FF8970FAEC6 | 48:3BC3              | cmp rax,rbx                             |

00007FF8970FAEC9 | 75 05                | jne ida.7FF8970FAED0                    |

}

{
// 【函数4】这里是进入关键算法
// 00007FF89712AB43  call              | 这里是验证算法，返回 TURE 为验证成功
...
00007FF89712C4E7 | 48:8D4C24 20         | lea rcx,qword ptr ss:[rsp+20]           |
00007FF89712C4EC | E8 BF77F8FF          | call             | 解析完 json
00007FF89712C4F1 | 85C0                 | test eax,eax                            |
00007FF89712C4F3 | 75 5C                | jne ida.7FF89712C551                    |
00007FF89712C4F5 | 837C24 20 03         | cmp dword ptr ss:[rsp+20],3             |
00007FF89712C4FA | 75 55                | jne ida.7FF89712C551                    |
00007FF89712C4FC | 4C:8BC5              | mov r8,rbp                              |
00007FF89712C4FF | 41:8BD6              | mov edx,r14d                            |
00007FF89712C502 | 48:8B4C24 28         | mov rcx,qword ptr ss:[rsp+28]           |
00007FF89712C507 | E8 94000000          | call              | ### 注册算法验证 ###【见下文函数5】
00007FF89712C50C | 85C0                 | test eax,eax                            | 这里返回 EAX = 0 则代表成功
00007FF89712C50E | 75 41                | jne ida.7FF89712C551                    | 这里不跳
00007FF89712C510 | 48:85DB              | test rbx,rbx                            |
00007FF89712C513 | 74 38                | je ida.7FF89712C54D                     |
00007FF89712C515 | 837C24 20 03         | cmp dword ptr ss:[rsp+20],3             |
00007FF89712C51A | 75 6C                | jne ida.7FF89712C588                    |
00007FF89712C51C | 4C:8B0B              | mov r9,qword ptr ds:[rbx]               |
00007FF89712C51F | 4C:8B43 08           | mov r8,qword ptr ds:[rbx+8]             | r8:&"{\n  "payload": {\n    "name": "Nisy/PYG"
00007FF89712C523 | 48:8B53 10           | mov rdx,qword ptr ds:[rbx+10]           |
00007FF89712C527 | 48:8B4C24 28         | mov rcx,qword ptr ss:[rsp+28]           |
00007FF89712C52C | 48:8B01              | mov rax,qword ptr ds:[rcx]              |
00007FF89712C52F | 48:8903              | mov qword ptr ds:[rbx],rax              |
00007FF89712C532 | 48:8B41 08           | mov rax,qword ptr ds:[rcx+8]            |
00007FF89712C536 | 48:8943 08           | mov qword ptr ds:[rbx+8],rax            |
00007FF89712C53A | 48:8B41 10           | mov rax,qword ptr ds:[rcx+10]           |
00007FF89712C53E | 48:8943 10           | mov qword ptr ds:[rbx+10],rax           |
00007FF89712C542 | 4C:8909              | mov qword ptr ds:[rcx],r9               |
00007FF89712C545 | 4C:8941 08           | mov qword ptr ds:[rcx+8],r8             | r8:&"{\n  "payload": {\n    "name": "Nisy/PYG"
00007FF89712C549 | 48:8951 10           | mov qword ptr ds:[rcx+10],rdx           |
00007FF89712C54D | B3 01                | mov bl,1                                | 验证成功赋值为 1
00007FF89712C54F | EB 02                | jmp ida.7FF89712C553                    |
00007FF89712C551 | 32DB                 | xor bl,bl                               |
...                                 |
00007FF89712C569 | 0FB6C3               | movzx eax,bl                            |
00007FF89712C56C | 48:8B5C24 70         | mov rbx,qword ptr ss:[rsp+70]           | [qword ptr ss:[rsp+70]]:writebytes+10D2B
00007FF89712C571 | 48:8B6C24 78         | mov rbp,qword ptr ss:[rsp+78]           | [qword ptr ss:[rsp+78]]:&"D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic"
00007FF89712C576 | 48:8BB424 80000000   | mov rsi,qword ptr ss:[rsp+80]           |
00007FF89712C57E | 48:83C4 50           | add rsp,50                              |
00007FF89712C582 | 41:5F                | pop r15                                 | r15:&"D:\\Program Files\\IDA Professional 9.0\\idapro_96-2923-BE84-E1.hexlic"
00007FF89712C584 | 41:5E                | pop r14                                 |
00007FF89712C586 | 5F                   | pop rdi                                 |
00007FF89712C587 | C3                   | ret                                     |
}
```

```bash
{【函数5】读取 JSON 中的 payload 对象，序列化后计算 SHA256 的哈希值，并用公钥(N,e)解密 signature 对象的 HEX 字符串值，将解密的数据[0x20~0x40)和 SHA256 数值做比较，若相同则验证成功

// 00007FF89712C507     call   

00007FF89712C5A0 | 48:895C24 10         | mov qword ptr ss:[rsp+10],rbx           | 进入验证函数

00007FF89712C5A5 | 55                   | push rbp                                |

00007FF89712C5A6 | 56                   | push rsi                                |

...

00007FF89712C5F0 | 4C:396B 08           | cmp qword ptr ds:[rbx+8],r13            |

00007FF89712C5F4 | 48:8D0D F06E0600     | lea rcx,qword ptr ds:[7FF8971934EB]     |

00007FF89712C5FB | 74 03                | je ida.7FF89712C600                     |

00007FF89712C5FD | 48:8B0B              | mov rcx,qword ptr ds:[rbx]              |

00007FF89712C600 | 48:8D15 41FA0900     | lea rdx,qword ptr ds:[7FF8971CC048]     | 找到 JSON 中 signature 字段

00007FF89712C607 | E8 84600100          | call                       |

00007FF89712C60C | 85C0                 | test eax,eax                            |

00007FF89712C60E | 74 4D                | je ida.7FF89712C65D                     | 找到之后，跳到新篇章

00007FF89712C610 | 48:FFC7              | inc rdi                                 |

00007FF89712C613 | 48:83C3 28           | add rbx,28                              | rbx 为当前 JSON 的字段信息: payload\header\signature

00007FF89712C617 | 48:3BFE              | cmp rdi,rsi                             | 枚举 JSON 中的字段（3个）

00007FF89712C61A | 72 D4                | jb ida.7FF89712C5F0                     |

...

00007FF89712C65D | 48:8D3CBF            | lea rdi,qword ptr ds:[rdi+rdi*4]        |
```
