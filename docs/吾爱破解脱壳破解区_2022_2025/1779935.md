
# [原创] PCVX逆向全解：自己手搓多开补丁

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1779935
- 抓取时间：2026-11-01T22:53:16

## 前言

没错，该系列又复活了 :P

## 使用工具

- VX：3.9.2.23
- 吾爱专版OD

## 分析

- 提前打开一个微信
- 用OD打开再一个微信（而非附加！）
- 下断 `CreateMutexW`
- 断下后观察堆栈

![](https://i.328888.xyz/2023/04/29/iKkV85.png)

像这样 `MutexName` 是 `NULL` 或者调用模块非 `WeChatWin.dll` 的都可以直接继续运行。

断下、运行两三次以后就像这样就可以了。

![](https://i.328888.xyz/2023/04/29/iKkttE.png)

在 `/CALL` 到 `CreateMutexW` 来自 `WeChatWi.XXXXXXXX` 上右键点击反汇编窗口中跟随，并在到达的地方的上一句下断，取消原来的断点，重新运行微信。

即在此下断：

```asm
11526697    FF15 8C435A12   call dword ptr ds:[]         ; kernel32.CreateMutexW
```

断下后观察周围代码：

```asm
   11526692    50              push eax
   11526693    6A 00           push 0x0
   11526695    6A 00           push 0x0
   11526697    FF15 8C435A12   call dword ptr ds:[]         ; kernel32.CreateMutexW
   1152669D    8BF0            mov esi,eax
   1152669F    32DB            xor bl,bl
   115266A1    85F6            test esi,esi
   115266A3    74 1E           je short WeChatWi.115266C3
   115266A5    FF15 B0425A12   call dword ptr ds:[]         ; kernel32.GetLastError
   115266AB    3D B7000000     cmp eax,0xB7
   115266B0    75 0B           jnz short WeChatWi.115266BD
   115266B2    56              push esi
   115266B3    B3 01           mov bl,0x1
   115266B5    FF15 9C425A12   call dword ptr ds:[]          ; kernel32.CloseHandle
   115266BB    EB 06           jmp short WeChatWi.115266C3
   115266BD    8935 1CEBE412   mov dword ptr ds:[0x12E4EB1C],esi
   115266C3    8B45 98         mov eax,dword ptr ss:[ebp-0x68]
   115266C6    8B35 B8535A12   mov esi,dword ptr ds:[]         ; mmtcmall.mm_free
   115266CC    85C0            test eax,eax
   115266CE    74 0D           je short WeChatWi.115266DD
   115266D0    50              push eax
   115266D1    FFD6            call esi
```

发现 `CreateMutexW` 下有一 `je` 跳转未实现，没有跳过 `CloseHandle`（`CloseHandle` 是什么意思想必各位都知道吧）。

尝试将其改成 `jmp`。

运行，发现成功实现多开。

## 制作补丁

右键修改完的 `jmp`，复制到可执行文件 -> 所有修改 -> 全部复制。

![](https://i.328888.xyz/2023/04/29/iKkQHc.png)

右键 -> 保存文件 -> 选择路径。

补丁制作完成，替换安装目录下的文件即可（一定要备份原版的！！！）。

## 优缺点

- [+] 随时开，无限开
- [+] 无需借助第三方软件，无中毒风险
- [+] 长期可用
- [-] 每个版本都要单独搓补丁
- [-] 做补丁麻烦（吗？）

大家多评分啊，我要升级
