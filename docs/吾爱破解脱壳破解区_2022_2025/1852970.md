
# [原创] Nevercenter CameraBag Pro 2024.0.1破解简易分析

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1852970`
- 抓取时间: `2026-11-01T22:43:48`

0day放出的补丁都是加VMP壳的，不利于我们这样的小白对比学习呀！所以在这里做一个简易的记录，此方法不是最完美的，但可以供大家共同探讨。

### 第一步
运行软件会出现注册窗口，任意输入注册码确认后会出现 `"Invalid registration code. Please try again - ……" `字符串，所以我们打开 `X64DBG`，加载运行程序。

### 第二步
在 `X64DBG` 中查找字符串 `"Invalid registration code."`，找到后在其上双击来到反汇编窗口

```
00007FF747D1EA1 | 40:55 | PUSH RBP |
00007FF747D1EA1 | 53 | PUSH RBX |
00007FF747D1EA1 | 56 | PUSH RSI |
00007FF747D1EA1 | 57 | PUSH RDI |
00007FF747D1EA1 | 41:56 | PUSH R14 |
00007FF747D1EA1 | 41:57 | PUSH R15 |
00007FF747D1EA1 | 48:8BEC | MOV RBP,RSP |
00007FF747D1EA1 | 48:83EC 38 | SUB RSP,0x38 |
00007FF747D1EA2 | 48:8BF9 | MOV RDI,RCX |
00007FF747D1EA2 | 0F297C24 20 | MOVAPS XMMWORD PTR SS:[RSP+0x20] |
00007FF747D1EA2 | 48:8D4D 38 | LEA RCX,QWORD PTR SS:[RBP+0x38] |
00007FF747D1EA2 | FF15 AE003000 | CALL QWORD PTR DS:[...]
```

`00007FF747D1DE07 mov byte ptr ds:[rcx+EC],0`

`00007FF747D1E1AF mov byte ptr ds:[rcx+EC],0`

`00007FF747D1E270 mov byte ptr ds:[rcx+EC],0`

把三处赋值的 `0` 分别修改为 `1`（此处可视为破解点-2）

### 第四步
以上完成后运行破解完的程序，在启动界面点击 `"Dismiss"`，程序终止，看来还有关键位置。重新开始分析，在 `00007FF6B8C2DE0 MOV BYTE PTR DS:[RCX+0xEC],0x0` 处往下分析，发现有一个退出函数调用 `00007FF6B8C2DE3 CALL QWORD PTR DS:[]`，上面有一个 `je`，在上面使 `cmp` 比较语句，`00007FF6B8C2DE2 CMP BYTE PTR DS:[RCX+0x9C],0x0` 很显然只要 `DS:[RCX+0x9C]=0`，`je` 跳转实现，所以在此 `cmp` 上右键——查找引用——常量，找给 `DS:[XXX+0x9C]` 赋值为 `1` 的语句，得到4处：

```
MOV BYTE PTR DS:[RCX+0x9C],0x1
MOV BYTE PTR DS:[RAX+0x9C],0x1
MOV BYTE PTR DS:[RBX+0x9C],0x1
MOV DWORD PTR DS:[RAX+0x9C],0x1
```

经逐步分析，`MOV BYTE PTR DS:[RAX+0x9C],0x1`是我们所要的，在此处赋值为 `0`（此处视为破解点-3），有兴趣的可以逐个试一下就明白了。
