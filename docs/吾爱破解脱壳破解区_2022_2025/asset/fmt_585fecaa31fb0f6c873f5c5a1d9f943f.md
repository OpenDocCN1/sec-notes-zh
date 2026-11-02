
# 万能五笔外接程序与配置修改

## 汇编片段

```asm
00D23354 | 0F1005 240CE200          | movups xmm0,xmmword ptr ds:[E20C24]          | 00E20C24:L"WanNengWB"

00C50100 | 51                       | push ecx                                     |
00C50101 | 56                       | push esi                                     |
00C50102 | 8BF1                     | mov esi,ecx                                  |
00C50104 | 57                       | push edi                                     |
00C50105 | 8D86 C0010000            | lea eax,dword ptr ds:[esi+1C0]               |
00C5010B | 50                       | push eax                                     |
00C5010C | FF15 7813E000            | call dword ptr ds:[                       |
00C318C2 | 83C4 48                  | add esp,48                                   |
00C318C5 | 8D85 DCF7FFFF            | lea eax,dword ptr ss:[ebp-824]               |
00C318CB | 6A 00                    | push 0                                       |
00C318CD | 6A 2B                    | push 2B                                      |
00C318CF | 50                       | push eax                                     |
00C318D0 | 6A 00                    | push 0                                       |
00C318D2 | FF15 F413DE00            | call dword ptr ds:[eax=003EF00C L"C:\\Program Files (x86)\\Common Files\\WanNengWB\\WanNengWB.ini"
```

## 绿色版与配置文件

2) 现在我们可以正常使用绿色版的了吗？

- `C:\Program Files (x86)\Common Files\WanNengWBInput\WanNengWB.ini`
- `C:\Program Files (x86)\Common Files\WanNengWB\WanNengWB.ini`

![](https://attach.52pojie.cn/forum/202212/18/105939zg1lggzgt2rqrrqt.png)

image.png (275.58 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTg4NHw4OGM2YjFkMHwxNzkzNTc0MjAzfDE2ODA4NnwxNzI4MDUy&nothumb=yes)

2022-12-18 10:59 上传

![](https://attach.52pojie.cn/forum/202212/18/110007b21dgt1z1hhgb1bd.png)

image.png (69.78 KB, 下载次数: 1)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTg4NXw2YzJjMzBkN3wxNzkzNTc0MjAzfDE2ODA4NnwxNzI4MDUy&nothumb=yes)

2022-12-18 11:00 上传

接下来，我们要调用能调用的全部工具，即`wntools.exe`（那个抖动的美女广告由此产生），它是万能五笔输入法外接程序接口程序，主要负责安装类似扩展这种。

内置流氓广告……

其实说穿了，就是需要它输出的`*.ini`配置文件（`C:\wnwb\wnwb10.2.4.20809MyCofigFolder\WanNengWBIME\Config\Config.ini`）

内容如下：

```ini
[Setting]

IsUseWbMode=1

IsUsePyMode=0

IsUseYYMode=1

IsUseBhMode=0

IsUseYYToChMode=0

IsEmptyCodeQueryGBKWord=1

CodeFloatPegging=1

IsUseWbPegging=1

IsUsePyPegging=1

IsUseEnPegging=0

IsUseZGPegging=1

IsShowOutputWordSpeed=0

IsShiftInputEnInitialLetterCapital=1

IsIntelligentDeleteMakeWord=1

IsTransferEnOutputExistComp=1

IsPurePyRemindWbCode=1

IsMixWbFourCodeOnlyOutput=1

IsMixWbFourCodeAutoOutput=1

IsMixWbFourCodeEmptyCancel=1

IsMixWbFourCodeStopOutput=1

IsMixWbCodeGraduallyRemind=1

IsPureWbCodeGraduallyRemind=1

IsPureWbSingleInputMode=0

IsPureWbEmptyCodeCancelInput=1

IsPureWbFourCodeStopOutput=1

IsPureWbFourCodeAutoOutput=1

IsPureWbFourCodeOnlyOutput=1

WbSingleWordInput=0

IsWBOnlySameCodeShowPhrase=0

IsPureWbErrorCodeAlarm=1

IsSpaceToWord=0

IsLetterSelectWord=0

IsPureWbForuCodeRepeatAlarm=0

IsSlashToCovertPauseMarkV2=1

IsOppositeSlashToCovertPauseMarkV2=1

IsCandShowYYTranslate=1

IsNoUseShiftCtrlSwitchCE=0

IsUseWBCustomPhrase=1

IsUseWBUserPhrase=1

IsUsePYCustomPhrase=1

IsUsePYUserPhrase=1

IsUseDYUserPhrase=1

IsUseUrlUserPhrase=1

IsUseFirstLetterJianPin=1

IsUseDynamicMakePinYinWord=1

IsUseDynamicAddUserPinYinWord=1

IsUsePinYinErrorRecovery=1

IsErrorRecoveryGnToNg=1

IsErrorRecoveryMgToNg=1

IsErrorRecoveryIouToIu=1

IsErrorRecoveryUeiToUi=1

IsErrorRecoveryUenToUn=1

IsUseKJOemMinusPlus=1

IsUseKJSquareBrackets=1

IsUseKJOemCommaPeriod=1

IsKJUseTabShiftTab=1

IsUseKJOemMinusPlusPhraseChooseWord=0

IsUseKJSquareBracketsPhraseChooseWord=0

IsUseKJOemCommaPeriodPhraseChooseWord=0

IsPeggingAssociateAutoHideWnd=1

IsUseWbZRepeatInput=1

IsUseSmallKeyboardBJInput=0

IsBiaoDianAutoMatch=0

IsBJToQjSpace=0

IsUsePyFuzzy=0

IsWebMailAutoReplenish=1

IsKeypadInputEn=1

IsCapslockCompAnOutput=0

IsSemicolonInputMode=0

IsUserDefinePhrase=1

IsNumAfterOutputEnPeriod=1

NumLaterOutputEnCommas=0

NumBetweenOutputEnColon=1

IsWebInputMode=1

IsDifferentWndShowCompCand=0

IsHiddenStatus=0

IsHiddenStatusToTray=0

IsFullScreenHiddenStatus=1

IsUseSkin=1

IsReSetFontStyle=0

IsReSetFontColor=0

IsCnStatusUseEnPunctuation=0

ShowFixPosIcon=1

IsWBSameCandidateBrifeCodeMode=0

IsNoCursorFollow=0

IsNoCursorFollowOfWaiGua=1

SyncUserMB=1

SyncUserMBPhrase=1

SyncUserMBMake=1

SyncUserCfg=1

QueryChineseWordMode=0

MixWbInputPosRevision=0

PureWbInputPosRevision=0

PurePyInputPosRevision=0

WbSameCodeCandidateWordSortMode=0

WbSameCandidateBriefCodeOperateMode=0

EnterKeyUseFor=0

CandFontSize=16

GaoFPAdjust=1

StatusWndTransparency=0

CandWndTransparency=0

CodePeggingPhraseAssociateMode=0

PeggingAssociateAutoHideSecond=5

UseCEToCutOverHotKeyV2=1

UseCtrlNumOpenSoftKeyboard=0

AutoMakeWordSetting=1

CompCandWndShowMode=1

CandWndAlignAsCompWnd=1

ApplicationUpdate=2

AutoMakeWordSettingV1=0

ShowCandWordNum=6

IsUsePositionPhrase=1

IsUseDelUserPhrase=1

UseBhPeggingMode=0

IsUseLUSystemPhrase=1

IsUseCellDict=1

IsUseAutoUpdateCellDict=1

AllWndTransparency=0

IsUseWBToTempPY=1

IsUseDiyWbPhrase=0

UseSearchCand=0

UseSearchCandInDesktop=0

UseSearchCandInSearch=1

UseSearchCandInChat=0

UseSearchCandInWriting=0

UseSearchCandInWeb=0

UseSearchCandInShop=0

SearcchEngineType=0

IsCandRows=0

TextImage=1

WBCodeType=0

Choose23CandWord=0

WordPostionKey=0

WordDeleteKey=0

SymbolLibWndLeftPos=1320

SymbolLibWndTopPos=527

CharFaceWndLeftPos=-1

CharFaceWndTopPos=-1

CharDrawWndLeftPos=-1

CharDrawWndTopPos=-1

UseHengShortCutKey=H

UseShuShortCutKey=I

UsePieShortCutKey=P

UseZheShortCutKey=V

UseLaShortCutKey=N

UseFillUpShortCutKey=O

UseSkin=尊贵会员(尊享版)

EnFontName=Arial

CnFontName=宋体

CompColor=#cccccc

ChooseCandColor=#cccccc

OtherCandColor=#cccccc

MergeWndLeftPos=322

MergeWndTopPos=466

SkinName=尊贵会员(尊享版)

StatusLeftPosOfScreen=1510

StatusTopPosOfScreen=764

StatusWndWidth=325

StatusWndHeight=32

HandWndLeft=1373

HandWndTop=382

MakeStatusWndShow=0

StatusBtnColor=4627431

StatusCnBtn=1

StatusBQBtn=1

StatusBiaoDianBtn=1

StatusKeyBtn=1

StatusUserBtn=1

StatusJFBtn=1

StatusCutBtn=1

StatusMenuBtn=1

StatusHandBtn=1         //手写

StatusSkinBoxBtn=1

StatusVoiceBtn=1

StatusCalculatorBtn=1  //计算器

StatusSearchBtn=1       //搜索

StatusCloudDiskBtn=0

StatusClipboardBtn=0

InstallSkin=1
```

## 替换外部工具

接下来把电脑上好用的讯飞语音输入法、手写输入法、计算器、截图工具用同名文件覆盖过去。

![](https://attach.52pojie.cn/forum/202212/18/112748ogs14j8z14smm1yg.png)

image.png (123.77 KB, 下载次数: 0)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU3OTg4OHwzNjhmMDM1ZnwxNzkzNTc0MjAzfDE2ODA4NnwxNzI4MDUy&nothumb=yes)

2022-12-18 11:27 上传

## AHK 热键脚本

接下来，用AHK编写热键控制的脚本程序：

```ahk
;比如你要启动万能五笔主程序,你可以这么写按下Ctrl+5启动万能五笔

^5::

        Run,c:\wnwb\wnwb.exe

return

^Space::#Space      ;用Ctrl+Space替换win10输入法默认的Win+Space

其他的大家照着修改就OK了，能用定义的热键实在太多了，当然还有热字符串，也是可以的哟~~

F1 & E::                          ;F1+E打开EmEditor.exe

Process, Exist, EmEditor.exe

If ErrorLevel>0

    Sleep , 100

Else

    Run , D:\EmEditor\EmEditor.exe

return

F1::                                ;F1开或关讯飞语音输入法

  Process, Exist, iFlyVoice.exe

  If ErrorLevel>0

      Process,close, iFlyVoice.exe

  Else

      Run,D:\图文处理\讯飞语音\最新一代讯飞语音输入法\iFlyVoice.exe

return

F12::Suspend                                    ;挂起按键

;_____下边是Markdown热键__________________________________

^y::                        ;引用外部图片链接

  Send , {!}[图片说明](%Clipboard%)

return

;=========================

^b::                        ;加粗

  Send , ^x

  Send , **^v**

return

;=========================
```
