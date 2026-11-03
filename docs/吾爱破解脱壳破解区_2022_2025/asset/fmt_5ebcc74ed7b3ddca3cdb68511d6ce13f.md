
```c
void __fastcall VerifiyFunc(unsigned __int16 **a1, const void *content, Sai_License_Info *licenseInfo)
{
    licenseInfo->MustBeOne01 = 1;
    licenseInfo->MustBeOne02 = 1;
    licenseInfo->MustBeOne03 = 1;
    licenseInfo->HardwareCodeFromLicense = HardwareCodeFromLicense;
}
```

## 修改后的汇编码

```asm
mov [rsp+0x8], rbx
mov [rsp+0x10], rsi
push rdi
sub rsp,0x860
mov dword ptr [r8], 0x1
mov dword ptr [r8+0x14], 0x1
mov dword ptr [r8+0x1c], 0x1
mov eax, dword ptr cs:[HardwareCodeFromLicense]
mov dword ptr [r8+0x4], eax
lea r11, [rsp+0x868+0x8]
mov rbx, [r11+0x10]
mov rsi, [r11+0x18]
mov rsp, r11
pop rdi
retn
```

### 修改前：

![](https://attach.52pojie.cn/forum/202503/22/190559fc8wty3bdytb3hcz.png)

Pasted image 20250322173543.png (73.21 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NTAxOXw1MDU5YWMxN3wxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 19:05 上传

![](https://attach.52pojie.cn/forum/202503/22/190603by6cv8cc48pi88i5.png)

Pasted image 20250322173616.png (56.97 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NTAyMHw3YTQ1MzhiOXwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 19:06 上传

### 修改后程序如下：

![](https://attach.52pojie.cn/forum/202503/22/190619cqhpzxh3q4a5pa5h.png)

Pasted image 20250322162946.png (73.08 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NTAyMXw2ZjQ2ZGVjM3wxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 19:06 上传

![](https://attach.52pojie.cn/forum/202503/22/190634fcwczcifrvwdzozs.png)

Pasted image 20250322160001.png (89.74 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NTAyMnw2ZDVhY2JiYnwxNzkzNTcxMTA3fDE2ODA4NnwyMDE3NjMx&nothumb=yes)

2025-3-22 19:06 上传

爆破成功，可以随便打开、保存文件。
