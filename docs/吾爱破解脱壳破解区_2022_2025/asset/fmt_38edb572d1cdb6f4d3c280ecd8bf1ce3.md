
```
#include
#include
#include
#include 
BYTE pOrgByte[5] = { 0, };
void unhook()

{
        DWORD dwOldProtect;
        PBYTE plstrcmp;
        FARPROC pFunc;
        pFunc = GetProcAddress(GetModuleHandleA("kernel32.dll"), "lstrcmp");
        plstrcmp = (PBYTE)pFunc;
        VirtualProtect(plstrcmp, 5, PAGE_EXECUTE_READWRITE, &dwOldProtect);
        memcpy(plstrcmp, pOrgByte, 5);
        VirtualProtect(plstrcmp, 5, dwOldProtect, &dwOldProtect);
}

int __stdcall Mylstrcmp(LPCTSTR lpString1, LPCTSTR lpString2)
{
        printf("hook!\n");
        unhook();
        return 0;
}

int main()
{
        HANDLE hFile;
        HMODULE hKernel32;
        FARPROC plstrcmp;
        FILE* fp = NULL;
        PBYTE pEditFunc;
        BYTE pJmpCode[6] = { 0xE9,0, };
        char pwd[15];
        char key[15];
        int ret;
        DWORD dwOldProtect, pOffset, dwWritenSize;
        char buf[] = "real_pwd";
        hKernel32 = GetModuleHandleA("kernel32.dll");
        plstrcmp = GetProcAddress(hKernel32, "lstrcmp");
        pEditFunc = (PBYTE)plstrcmp;

        if (VirtualProtect(pEditFunc, 5, PAGE_EXECUTE_READWRITE, &dwOldProtect))
        {
                memcpy(pOrgByte, pEditFunc, 5);
                pOffset = (ULONGLONG)Mylstrcmp - (ULONGLONG)plstrcmp - 5;
                memcpy(&pJmpCode[1], &pOffset, 4);
                memcpy(plstrcmp, &pJmpCode[0], 5);
                VirtualProtect(plstrcmp, 5, dwOldProtect, &dwOldProtect);
        }
        hFile = CreateFileA("pwd.txt", GENERIC_ALL, 0, NULL, CREATE_ALWAYS, 0x80, NULL);

        /*
        if(hFile!=NULL)
                printf("open1 success\n");
        else
                printf("open1 fail\n");
        */

        WriteFile(hFile, buf, strlen(buf), &dwWritenSize, NULL);
        CloseHandle(hFile);

        if (errno_t err = fopen_s(&fp, "file.txt", "r"))
                printf("open success\n");
        else
                printf("open fail\n");
        if (fp != NULL)
                fscanf_s(fp, "%s", pwd);
        else
                printf("scan fail\n");

        printf("plz input key\n");
        scanf_s("%s", key);

        if (fp != NULL)
                fclose(fp);

        ret = lstrcmp((LPCTSTR)key, (LPCTSTR)pwd);

        if (ret == 0)
                printf("congratulations!\n");
        else
                printf("try again!\n");

        system("Pause");
        return 0;
}
```
### 未修改前

![](https://attach.52pojie.cn/forum/202505/17/112615n3m3j6nizw3pjq6j.png)

image-20250516155228654.png (206.13 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3NzY2OHwxMWEzZjA3NnwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3NzY2OHwxMWEzZjA3NnwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes)

2025-5-17 11:26 上传

![](../images/image-20250516155228654.png)
### 修改后

![](https://attach.52pojie.cn/forum/202505/17/112617r48acaa4m0jkpk1h.png)

image-20250516155335998.png (165.08 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3NzY2OXw5ZGJhOGFhOHwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3NzY2OXw5ZGJhOGFhOHwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes)

2025-5-17 11:26 上传

![](../images/image-20250516155335998.png)
### 脱钩后

![](https://attach.52pojie.cn/forum/202505/17/112619aso11u6uonxmn2b2.png)

image-20250516155524152.png (161.22 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3NzY3MHxiYjM1ZTU5NHwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3NzY3MHxiYjM1ZTU5NHwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes)

2025-5-17 11:26 上传

![](https://attach.52pojie.cn/forum/202505/17/112621d6uwqhf9x99uhqfp.png)

image-20250516155616119.png (18.97 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc3NzY3MXw0YzRlYjMzOXwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc3NzY3MXw0YzRlYjMzOXwxNzkzNTcwOTcyfDE2ODA4NnwyMDMyMDcx&nothumb=yes)

2025-5-17 11:26 上传

![](../images/image-20250516155524152.png)

![](../images/image-20250516155616119.png)
## 4> IAT Hook

这个的思想就是修改目标API函数的地址，那么这个地址存放在哪里呢，请看上一部分~~
1. 定位`KERNEL32.dll`里面`IAT`表的位置
2. 在`IAT`表中寻找要`Hook`函数对应的地址条目
3. 将`WriteFile`对应的地址条目替换为自己的函数地址

我们这个示例`Hook` `KERNEL32.dll`里面的`WriteFile`函数：
### 5> 1. 定位`KERNEL32.dll`里面`IAT`表的位置
```
IMAGE_DOS_HEADER* pImgDosHdr = (IMAGE_DOS_HEADER*)hModule;

IMAGE_OPTIONAL_HEADER* pImgOptHdr = (IMAGE_OPTIONAL_HEADER*)((DWORD)hModule + pImgDosHdr->e_lfanew + 24);

IMAGE_IMPORT_DESCRIPTOR* pImgImportDes = (IMAGE_IMPORT_DESCRIPTOR*)((DWORD)hModule + pImgOptHdr->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);
```
首先获取DOS头，然后Dos头的`e_lfanew`字段段找到NT头，再偏移24字节定位到可选头(`Optional Header`)

从可选头的`DataDirectory`数组中获取导入目录表(`Import Directory`)的位置（`VirtualAddress`）
```
while (pImgImportDes->Characteristics != 0)
{
    USES_CONVERSION;
    LPCTSTR lpszName = ((LPCSTR)(DWORD)hModule + pImgImportDes->Name);
    TargetLibraryName = lpszName;
    if (TargetLibraryName.compare(pImageName) == 0)
    {
        pImgThunkData = (IMAGE_THUNK_DATA*)((DWORD)hModule + pImgImportDes->FirstThunk);
        break;
    }
    pImgImportDes++;
}
```
遍历导入描述符(`Import Descriptor`)表，直到找到`Characteristics`为0的条目(表示结束)

对每个描述符获取DLL名称并与目标名称比较

找到匹配的DLL后，获取`FirstThunk`字段的地址，这就是该DLL的`IAT`表开始位置（本示例是`kernel.dll`）
### 5> 2. 在`IAT`表中寻找要`Hook`函数对应的地址条目 3. 将`WriteFile`对应的地址条目替换为自己的函数地址
```
    while ( pImgThunkData->u1.Function )
    {

        FuncAddress = ( LPDWORD ) & ( pImgThunkData->u1.Function );

        if ( *FuncAddress == ( DWORD )pTargetFuncAddr )
        {

            VirtualProtect( FuncAddress, sizeof( DWORD ), PAGE_READWRITE, &OldProtect );

            if ( !WriteProcessMemory( ( HANDLE ) - 1, FuncAddress, &pReplaceFuncAddr, 4, NULL ) )
            {

                return FALSE;

            }

            VirtualProtect( FuncAddress, sizeof( DWORD ), OldProtect, 0 );

            return TRUE;

        }

        pImgThunkData++;

    }

    return FALSE;
```
在`IAT`表里面遍历寻找要替换的那个`Hook`函数（本示例是`WriteFile`）对应的地址，然后替换为自己的函数地址

剩余的操作其实与`Inline Hook`差不多

在`MyWriteFile`中，使用了`p_WriteFile`，会正常调用`WriteFile`函数

看一下整体实现：
