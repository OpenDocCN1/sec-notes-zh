
# [原创] EditPlus 5.6 build 4772 按下 `Ctrl+B` 浏览器预览 bug 的解决

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1698195
- 抓取时间：2026-11-01T23:08:08

这个编辑器法王姥爷已经很多年没用过了。这两天看到某 pdf 书籍里时就偶尔安了个最新版本试一下。

结果随便打开个 `*.html`，按下 `Ctrl+B`，使用浏览器预览，就会触发下面的：

![](https://attach.52pojie.cn/forum/202210/11/202033kdx5lc7lwhta5hz1.png)

`image.png (104.51 KB, 下载次数: 0)`

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTgyNHxhNTA5NDUyNXwxNzkzNTc0NDg1fDE2ODA4NnwxNjk4MTk1&nothumb=yes)

2022-10-11 20:20 上传

如图所示，工具栏第二排左下角的那个按钮的 bug，就算你打了官方提供的补丁包也是无济于事。。。。

所以，我们就来汇编修补下吧。

咱下载的是 32 位的，所以就用 `x32dbg` 来打开它吧。

话不多说，当这个弹框出现后，你可以采用多种方法定位到。

- 搜索全部模块，所有字符串
- 使用 `string` 插件
- 利用暂停回溯法
- 既然是弹框，那就用弹框类的 `api`
- 利用跟踪窗口
- 利用句柄窗口
- 最笨的方法，随便找个前置断点，按下热键 `Ctrl+F8`，当 `Ctrl+B` 走到关键点上就会触发
- 下窗口类断点
- 把光标移到该工具栏上，我们得知 `Browser` 字符串恐怕会调用，所以全下断，果然能断到开始的地方

开始曾经怀疑过有反调试：

![](https://attach.52pojie.cn/forum/202210/11/202715zu11j13duuut5ku8.png)

`image.png (35.16 KB, 下载次数: 1)`

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTgyNXxhNTMzNTY0YXwxNzkzNTc0NDg1fDE2ODA4NnwxNjk4MTk1&nothumb=yes)

2022-10-11 20:27 上传

![](https://attach.52pojie.cn/forum/202210/11/202851kzg3uzebodookxui.png)

`image.png (150.79 KB, 下载次数: 1)`

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTgyNnxhZTU3OTY4N3wxNzkzNTc0NDg1fDE2ODA4NnwxNjk4MTk1&nothumb=yes)

2022-10-11 20:28 上传

你能搜索到好多。

你可以顺手给主模块下一个：

![](https://attach.52pojie.cn/forum/202210/11/203042d88prspshtyqht0p.png)

`image.png (201.3 KB, 下载次数: 1)`

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTgyOHxjZDVkNmMzZXwxNzkzNTc0NDg1fDE2ODA4NnwxNjk4MTk1&nothumb=yes)

2022-10-11 20:30 上传

当你按 `Ctrl+B` 时就会来到此处。

`WebView2Loader.dll` 是啥？那是源于 Edge dev 内核浏览器，开发 Delphi 浏览器控件所必须的文件支持。

接下来连按几次 `Ctrl+F9` 来到调用返回的地方。也可以结合堆栈窗口的地址，回车前往。又或者，可以用前面找到的：

`001D4451 | B9 E80E3700 | mov ecx,editplus.370EE8 | 370EE8:L"Browser"` 调用提示处的字符

![](https://attach.52pojie.cn/forum/202210/11/203907z3yuglbf3dd8icm3.png)

`image.png (12.38 KB, 下载次数: 1)`

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU1OTgyOXw4MzMyYThjYnwxNzkzNTc0NDg1fDE2ODA4NnwxNjk4MTk1&nothumb=yes)

2022-10-11 20:39 上传

不论咋样，你定位到了。

```
001D4320
001D4321 | 8BEC | mov ebp,esp |
001D4323 | 6A FF | push FFFFFFFF |
001D4325 | 68 FD583300 | push |
001D432A | 64:A1 00000000 | mov eax,dword ptr fs:[0] | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html", [00000000]:"D蟪"
001D4330 | 50 | push eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4331 | 83EC 0C | sub esp,C |
001D4334 | A1 B82A3A00 | mov eax,dword ptr ds:[3A2AB8] | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4339 | 33C5 | xor eax,ebp |
001D433B | 8945 F0 | mov dword ptr ss:[ebp-10],eax |
001D433E | 53 | push ebx |
001D433F | 56 | push esi | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4340 | 57 | push edi | edi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4341 | 50 | push eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4342 | 8D45 F4 | lea eax,dword ptr ss:[ebp-C] |
001D4345 | 64:A3 00000000 | mov dword ptr fs:[0],eax | [00000000]:"D蟪", eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D434B | 8B75 14 | mov esi,dword ptr ss:[ebp+14] |
001D434E | 8B5D 08 | mov ebx,dword ptr ss:[ebp+8] | [ebp+8]:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4351 | 8B34B5 40ED3B00 | mov esi,dword ptr ds:[esi*4+3BED40] | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4358 | 85F6 | test esi,esi | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D435A | 0F84 90000000 | je editplus.1D43F0 |
001D4360 | 83FE 07 | cmp esi,7 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4363 | 0F84 87000000 | je editplus.1D43F0 |
001D4369 | 83FE 01 | cmp esi,1 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D436C | 75 0C | jne editplus.1D437A |
001D436E | 8B55 0C | mov edx,dword ptr ss:[ebp+C] |
001D4371 | 8BCB | mov ecx,ebx |
001D4373 | E8 A8FEFFFF | call |
001D4378 | EB 5D | jmp editplus.1D43D7 |
001D437A | 83FE 05 | cmp esi,5 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D437D | 75 15 | jne editplus.1D4394 |
001D437F | 8B0D 208F3C00 | mov ecx,dword ptr ds:[3C8F20] |
001D4385 | 8379 F4 00 | cmp dword ptr ds:[ecx-C],0 |
001D4389 | 7E 50 | jle editplus.1D43DB |
001D438B | 8BD3 | mov edx,ebx | edx:&L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D438D | E8 9EF7FFFF | call editplus.1D3B30 |
001D4392 | EB 43 | jmp editplus.1D43D7 |
001D4394 | 33C0 | xor eax,eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4396 | 83FE 02 | cmp esi,2 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4399 | 75 0A | jne editplus.1D43A5 |
001D439B | B9 5C0F3700 | mov ecx,editplus.370F5C | 370F5C:L"htmlfile"
001D43A0 | 8D46 FF | lea eax,dword ptr ds:[esi-1] | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43A3 | EB 22 | jmp editplus.1D43C7 |
001D43A5 | 83FE 06 | cmp esi,6 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43A8 | 75 07 | jne editplus.1D43B1 |
001D43AA | B9 700F3700 | mov ecx,editplus.370F70 | 370F70:L"FirefoxHTML"
001D43AF | EB 16 | jmp editplus.1D43C7 |
001D43B1 | 83FE 03 | cmp esi,3 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43B4 | 75 07 | jne editplus.1D43BD |
001D43B6 | B9 880F3700 | mov ecx,editplus.370F88 | 370F88:L"ChromeHTML"
001D43BB | EB 0A | jmp editplus.1D43C7 |
001D43BD | 83FE 04 | cmp esi,4 | esi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43C0 | 75 19 | jne editplus.1D43DB |
001D43C2 | B9 A00F3700 | mov ecx,editplus.370FA0 | 370FA0:L"MSEdgeHTM"
001D43C7 | 6A 00 | push 0 |
001D43C9 | 50 | push eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43CA | FF75 0C | push dword ptr ss:[ebp+C] |
001D43CD | 8BD3 | mov edx,ebx | edx:&L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43CF | E8 8CF8FFFF | call |
001D43D4 | 83C4 0C | add esp,C |
001D43D7 | 85C0 | test eax,eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43D9 | 75 0E | jne editplus.1D43E9 | 好蛋！
001D43DB | 6A FF | push FFFFFFFF |
001D43DD | 6A 00 | push 0 |
001D43DF | 68 7A4E0000 | push 4E7A |
001D43E4 | E8 56130F00 | call |
001D43E9 | 33C0 | xor eax,eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43EB | E9 42010000 | jmp editplus.1D4532 |
001D43F0 | C745 EC 00000000 | mov dword ptr ss:[ebp-14],0 | 貌似这里是好蛋才走的分支！！！
001D43F7 | E8 82FA0E00 | call |
001D43FC | 8BC8 | mov ecx,eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D43FE | 85C9 | test ecx,ecx |
001D4400 | 0F84 4A010000 | je editplus.1D4550 |
001D4406 | 8B01 | mov eax,dword ptr ds:[ecx] | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D4408 | FF50 0C | call dword ptr ds:[eax+C] |
001D440B | 83C0 10 | add eax,10 | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D440E | 8945 EC | mov dword ptr ss:[ebp-14],eax |
001D4411 | C745 FC 00000000 | mov dword ptr ss:[ebp-4],0 |
001D4418 | 33FF | xor edi,edi | edi:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D441A | 8B0D 64DE3A00 | mov ecx,dword ptr ds:[3ADE64] | 003ADE64:&"渜6"
001D4420 | E8 211E1000 | call |
001D4425 | 8945 E8 | mov dword ptr ss:[ebp-18],eax |
001D4428 | 85C0 | test eax,eax | eax:L"X:\\StudyJavaScript\\01.学习HTML\\StudyJS.html"
001D442A | 74 61 | je editplus.1D448D |
001D442C | 0F1F40 00 | nop dword ptr ds:[eax],eax |
001D4430 | 8B0D 64DE3A00 | mov ecx,dword ptr ds:[3ADE64] | 003ADE64:&"渜6"
```
