
[href="forum.php?mod=attachment&aid=MjcwOTIyM3wwNDEzZTBmOXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwOTIyM3wwNDEzZTBmOXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-17 19:47 上传

把`PlantsVsZombies.exe+FE2F` - `je PlantsVsZombies.exe+10754`：`je`改成`jmp`必定跳转，就能实现植物的叠加种植了。

![](https://attach.52pojie.cn/forum/202407/17/194708pk681yxt687bxh8h.png)

`10.png` (45.91 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjcwOTIyNHxlODllM2Q4OXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwOTIyNHxlODllM2Q4OXwxNzkzNTcyMDQyfDE2ODA4NnwxOTQ0MTMy&nothumb=yes)

2024-7-17 19:47 上传

### CE脚本代码

[Asm] 纯文本查看 复制代码

```
CE脚本代码：
[ENABLE]
//code from here to '[DISABLE]' will be used to enable the cheat
alloc(newmem,2048)
label(returnhere)
label(originalcode)
label(exit)

newmem: //this is allocated memory, you have read,write,execute access
//place your code here

originalcode:
jmp PlantsVsZombies.exe+10754

exit:
jmp returnhere

"PlantsVsZombies.exe"+FE2F:
jmp newmem
nop
returnhere:

 
[DISABLE]
//code from here till the end of the code will be used to disable the cheat
dealloc(newmem)
"PlantsVsZombies.exe"+FE2F:
db 0F 84 1F 09 00 00
//je PlantsVsZombies.exe+10754

```
