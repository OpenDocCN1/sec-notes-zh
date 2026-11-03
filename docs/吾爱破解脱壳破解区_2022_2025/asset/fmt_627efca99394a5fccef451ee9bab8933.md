
```cpp
SB_STATUS CSbieAPI::ReloadConf(quint32 flags, quint32 SessionId)
{
        __declspec(align(8)) ULONG64 parms[API_NUM_ARGS];

        memset(parms, 0, sizeof(parms));
        parms[0] = API_RELOAD_CONF;
        parms[1] = SessionId;
        parms[2] = flags;

        NTSTATUS status = m->IoControl(parms);
    // 新增的代码
        /**********************开始*************************/
    if(flags & SBIE_CONF_FLAG_RELOAD_CERT)
        {
                WCHAR exePatchPath[MAX_PATH] = { L'\x00' };
                if (GetModuleFileNameW(NULL, exePatchPath, MAX_PATH))
                {
                        PathRemoveFileSpecW(exePatchPath);
            // SandBoxiePlusPatch是我们编译出的补丁文件，用于加载驱动和修改Verify_CertInfo变量
                        wcscat(exePatchPath, L"\\SandBoxiePlusPatch.exe");
                        ShellExecuteW(NULL, L"open", exePatchPath, NULL, NULL, SW_SHOW);
                        OutputDebugStringW(exePatchPath);
                }
        }
    /**********************结尾*************************/
        if (!NT_SUCCESS(status))
                return SB_ERR(status);

        emit ConfigReloaded();

        m_bBoxesDirty = true;

        return SB_OK;
}
```

## 总结

本文利用`BYOVD`方法直接修改驱动`SbieDrv`的变量`Verify_CertInfo`为一个有效的值，实现了无证书的情况下使用其高级功能的效果。
