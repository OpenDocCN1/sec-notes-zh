
# WeChat 消息接收函数定位日志分析

## 原始日志

```text
5.685838        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  %s Current Sql : %s--MSG0.db MultiDBMsgStorage--PrepareStmt
5.686010        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  %s--pageMonitor--StorageBase::logPageMonitorInfo
5.686043        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  iPageTotal=%d, iPageFree=%d, iPageTableWrite=%d, iPageIndexWrite=%d,iPageOverflowWrite=%d,TimeConsuming = %d--pageMonitor--StorageBase::logPageMonitorInfo
5.686072        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  iPageTableRead=%d, iPageIndexRead=%d, iPageOverflowRead=%d--pageMonitor--StorageBase::logPageMonitorInfo
5.686100        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  Page Accsess statistics: cacheHit=%d, access1-4=%d, access4-8=%d, access>8=%d--pageMonitor--StorageBase::logPageMonitorInfo
5.686130        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  Page Accsess statistics: totalWrite=%d,totalRead=%d,totalTrans=%d--pageMonitor--StorageBase::logPageMonitorInfo
5.686507        2022/09/20 23:02:10.538        11964        WeChat.exe        WxLogs:  %s Current Sql : %s--MSG0.db MultiDBMsgStorage--PrepareStmt
5.687006        2022/09/20 23:02:10.539        11964        WeChat.exe        WxLogs:  %s--MultiDBMsgMgr--MultiDBMsgMgr::AddFilteredMsgList
5.692128        2022/09/20 23:02:10.544        11964        WeChat.exe        WxLogs:  no sessions in group box--SessionListModel--SessionListModel::ShowGroupBox
5.693921        2022/09/20 23:02:10.546        11964        WeChat.exe        WxLogs:  RefreshSessionGroipVoipStatus, name %s status %d--SessionListItemUI--SessionListItemUI::RefreshSessionGroipVoipStatus
5.886836        2022/09/20 23:02:10.739        11964        WeChat.exe        WxLogs:  out %s::buf2Resp unpackSize: %d, id:%d--NetSceneBaseEx--NetSceneBaseEx::buf2Resp
5.887095        2022/09/20 23:02:10.739        11964        WeChat.exe        WxLogs:  onGYNetEnd sceneID:%d errType:%d  errCode:%d--WinMarsMgr--WinMarsMgr::OnTaskEnd
5.887772        2022/09/20 23:02:10.739        11964        WeChat.exe        WxLogs:  onGYNetEnd(errType:%d, errCode:%d, sceneID:%d)--NetSceneSnsSync--NetSceneSnsSync::onGYNetEnd
5.887873        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  saveUInfo get SyncKey Buf md5: %s--SyncMgr--SyncMgr::MergeAndSaveSyncKeyBuf
5.888137        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  %s : %s--SyncKeyInfo --NetSceneInit::dumpSyncKey
5.888206        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  %s : %s--SyncKeyInfo --NetSceneInit::dumpSyncKey
5.888309        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  %s : %s--SyncKeyInfo --NetSceneInit::dumpSyncKey
5.888410        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  saveUInfo mergeKey Buf md5: %s--SyncMgr--SyncMgr::MergeAndSaveSyncKeyBuf
5.888491        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  On Sns Cmd List size : %d--SnsSyncMgr--SnsSyncMgr::OnSyncCmdList
5.888573        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  Process Sns Object : id %lu, size %d --SnsSyncMgr--SnsSyncMgr::ProcessSnsObject
5.888672        2022/09/20 23:02:10.740        11964        WeChat.exe        WxLogs:  the ad enable is %d--SNSAD_ONOFF--AdSwitch::AdvertiseEnable
5.888825        2022/09/20 23:02:10.741        11964        WeChat.exe        WxLogs:  CmdItem to SnsObject : id %lu time : %d type : %d user : %s--SnsSyncMgr--SnsSyncMgr::ProcessSnsObject
5.889718        2022/09/20 23:02:10.741        11964        WeChat.exe        WxLogs:  Do not preload before 10 min after login--SnsSyncMgr--SnsSyncMgr::ProcessSnsObject
5.889757        2022/09/20 23:02:10.741        11964        WeChat.exe        WxLogs:  Do Sync Finish--NetSceneSnsSync--NetSceneSnsSync::onGYNetEnd
5.889801        2022/09/20 23:02:10.742        11964        WeChat.exe        WxLogs:  ~  (id:%d)--NetSceneSnsSync--NetSceneSnsSync::~NetSceneSnsSync
7.101882        2022/09/20 23:02:11.954        0        [internal]        
7.101921        2022/09/20 23:02:11.954        0        [internal]        Source 'Win32 Messages' was removed.
```

## 可疑日志与字符串定位

既然是消息接收肯定会有跟消息相关的英文简写，可以尝试去猜测。通过对日志内容的粗略观察，`doAddMsg srvid: %d, msgtyp: %d,cTime:%u,msgseq: %d--SyncMgr--SyncMgr::doAddMsg` 这一句话很可疑。

然后到 `xdbg32` 中的 `WeChatWin.dll` 模块中去搜索 `doAddMsg` 这个字符串。

对地址的观察，发现前四个字节的数字是一样的，所以毫不犹豫地跳转到函数的头部下断，让程序跑起来。

再让另一个微信号给登录在电脑上的微信账号发送消息，发现在函数头部断下来了。

## 消息接收 `call` 定位

接下来就是一条一条地跟，这个跟也是有技巧的，重点来了：别人给你发送消息，接收到消息，肯定会有两个重要的内容，一个是消息发送者，另一个是消息内容。

这时候注意寄存器内有没有 `wxid` 的出现，这里需要反复地跟几遍，这个消息接收 `call` 距离函数头部并不是太远。

下面是我定位到的消息接收 `call` 位置处：

```asm
5FB4935B | 85C0                  | test eax,eax                            |
5FB4935D | 75 08                 | jne wechatwin.5FB49367                  |
5FB4935F | A1 1C8E9661           | mov eax,dword ptr ds:[0x61968E1C]       |
5FB49364 | 8B40 14               | mov eax,dword ptr ds:[eax+0x14]         |
5FB49367 | 6A 00                 | push 0x0                                |
5FB49369 | FF70 04               | push dword ptr ds:[eax+0x4]             |
5FB4936C | 8D8D 74EEFFFF         | lea ecx,dword ptr ss:[ebp-0x118C]       |
5FB49372 | E8 99892200           | call     |
```

可以在这里断下，让另一个微信发送各种消息格式，确定各种消息格式。

## 补充说明

这个消息接收 `call`，还有另外一个作用——解密加密的图片——得到 `aeskey` 值对图片进行解密。不过这里有个要注意的点：发送的图片不能是发送过的，如果是发送过的就不会带有 `aeskey`。

今天先到这里吧。最后附加一个，hook 日志的 dll。
