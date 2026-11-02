
```
02622FF0  | B9 58306202             | mov ecx,强删reportform-watermark_helpman.2623058         | 2623058:"Basic Edition"
02622FF5  | BA 74306202             | mov edx,   | 2623074:"STD"
02622FFA  | A1 48DB9B02             | mov eax,dword ptr ds:[29BDB48]                         |
02622FFF  | E8 448016FE             | call        |
02623004  | B9 84306202             | mov ecx,强删reportform-watermark_helpman.2623084         | 2623084:"Professional Edition"
02623009  | BA A8306202             | mov edx,   | 26230A8:"PRF"
0262300E  | A1 48DB9B02             | mov eax,dword ptr ds:[29BDB48]                         |
02623013  | E8 308016FE             | call        | yicang
02623018  | B9 B8306202             | mov ecx,强删reportform-watermark_helpman.26230B8         | 26230B8:"Floa蘨ng Professional License"
0262301D  | BA E4306202             | mov edx,   | 26230E4:"SRV"
02623022  | A1 48DB9B02             | mov eax,dword ptr ds:[29BDB48]                         |
02623027  | E8 1C8016FE             | call        | al=1
0262302C  | 8BC3                    | mov eax,ebx                                            |
0262302E  | 807D FF 00              | cmp byte ptr ss:[ebp-1],0                              |
02623032  | 74 0F                   | je 强删reportform-watermark_helpman.2623043              |
02623034  | E8 3B6CDEFD             | call 强删reportform-watermark_helpman.409C74             | al=1
02623039  | 64:8F05 00000000        | pop dword ptr fs:[0]                                   |
02623040  | 83C4 0C                 | add esp,C                                              |
02623043  | 8BC3                    | mov eax,ebx                                            |
02623045  | 5F                      | pop edi                                                |
02623046  | 5E                      | pop esi                                                |
02623047  | 5B                      | pop ebx                                                |
02623048  | 8BE5                    | mov esp,ebp                                            |
0262304A  | 5D                      | pop ebp                                                |
0262304B  | C3                      | ret                                                    |
```

![](https://attach.52pojie.cn/forum/202212/14/210032zjxyn6szksz8hsyu.png)

`image.png` (111.15 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjU3OTI0NXw1NmZhZTA2N3wxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTI0NXw1NmZhZTA2N3wxNzkzNTc0MjIxfDE2ODA4NnwxNzI1OTUy&nothumb=yes)

2022-12-14 21:00 上传

## 6.0版的授权流程比较简单：

[Asm] 纯文本查看 复制代码
