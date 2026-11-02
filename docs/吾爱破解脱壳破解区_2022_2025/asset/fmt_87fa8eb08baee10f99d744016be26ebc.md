
## 函数声明

```vb
Private Declare Sub et_RSAGenKey Lib "ft_et299_api"()

'VA: 4EBD30

Private Declare Sub et_TDesDecrypt Lib "ft_et299_api"()

'VA: 4EBCE8

Private Declare Sub et_TDesEncrypt Lib "ft_et299_api"()

'VA: 4EBCB4

Private Declare Sub et_SetKeyEx Lib "ft_et299_api"()

'VA: 4EBC70

Private Declare Sub MD5_HMAC Lib "ft_et299_api"()

'VA: 4EBC2C

Private Declare Sub et_HMAC_MD5 Lib "ft_et299_api"()

'VA: 4EBBE8

Private Declare Sub et_SetKey Lib "ft_et299_api"()

'VA: 4EBBA4

Private Declare Sub et_GetSNEx Lib "ft_et299_api"()

'VA: 4EBB60

Private Declare Sub et_SetSNEx Lib "ft_et299_api"()

'VA: 4EBB1C

Private Declare Sub et_GetSN Lib "ft_et299_api"()

'VA: 4EBAC0

Private Declare Sub et_TurnOffLED Lib "ft_et299_api"()

'VA: 4EBA78

Private Declare Sub et_TurnOnLED Lib "ft_et299_api"()

'VA: 4EBA30

Private Declare Sub et_GenRandom Lib "ft_et299_api"()

'VA: 4EB9E8

Private Declare Sub et_ResetSecurityState Lib "ft_et299_api"()

'VA: 4EB998

Private Declare Sub et_SetupToken Lib "ft_et299_api"()

'VA: 4EB950

Private Declare Sub et_ChangeUserPIN Lib "ft_et299_api"()

'VA: 4EB918

Private Declare Sub et_ResetPIN Lib "ft_et299_api"()

'VA: 4EB8D4

Private Declare Sub et_GenSOPIN Lib "ft_et299_api"()

'VA: 4EB890

Private Declare Sub et_Verify Lib "ft_et299_api"()

'VA: 4EB84C

Private Declare Sub et_GenPID Lib "ft_et299_api"()

'VA: 4EB808

Private Declare Sub et_Write Lib "ft_et299_api"()

'VA: 4EB7C4

Private Declare Sub et_Read Lib "ft_et299_api"()

'VA: 4EB784

Private Declare Sub et_CloseToken Lib "ft_et299_api"()

'VA: 4EB72C

Private Declare Sub et_OpenToken Lib "ft_et299_api"()

'VA: 4EB6E4

Private Declare Sub et_FindToken Lib "ft_et299_api"()

'VA: 4EB5C0

Private Declare Sub FD181ChkCalc Lib "fd181"()

'VA: 4EB560

Private Declare Sub FD181GetRand Lib "fd181"()

'VA: 4EB518

Private Declare Sub FD181CloseDev Lib "fd181"()

'VA: 4EB4D0

Private Declare Sub FD181SetPID Lib "fd181"()

'VA: 4EB48C

Private Declare Sub FD181GetPID Lib "fd181"()

'VA: 4EB448

Private Declare Sub FD181SetDispInfo Lib "fd181"()

'VA: 4EB3FC

Private Declare Sub FD181GetAuthorityData Lib "fd181"()

'VA: 4EB3C0

Private Declare Sub FD181SetAuthorityData Lib "fd181"()

'VA: 4EB370

Private Declare Sub FD181SetUserPassword Lib "fd181"()

'VA: 4EB320

Private Declare Sub FD181SetSuperPassword Lib "fd181"()

'VA: 4EB2D0

Private Declare Sub FD181VerifyUserPassword Lib "fd181"()

'VA: 4EB280

Private Declare Sub FD181VerifySuperPassword Lib "fd181"()

'VA: 4EB22C

Private Declare Sub FD181GetDID Lib "fd181"()

'VA: 4EB1C8

Private Declare Sub FD181LedClose Lib "fd181"()

'VA: 4EB180

Private Declare Sub FD181LedOpen Lib "fd181"()

'VA: 4EB138

Private Declare Sub FD181DevRead Lib "fd181"()

'VA: 4EB0F0

Private Declare Sub FD181DevWrite Lib "fd181"()

'VA: 4EB0A8

Private Declare Sub FD181OpenDev Lib "fd181"()

'VA: 4EB060

Private Declare Sub FD181FindDev Lib "fd181"()

'VA: 4EAF28

Private Declare Sub rc_GetLicenseInfo Lib "rcgranddogw32"()

'VA: 4EAEDC

Private Declare Sub rc_Upgrade Lib "rcgranddogw32"()

'VA: 4EAE98

Private Declare Sub rc_GetUpgradeRequestString Lib "rcgranddogw32"()

'VA: 4EAE44

Private Declare Sub rc_ExecuteFile Lib "rcgranddogw32"()

'VA: 4EADFC

Private Declare Sub rc_DefragFileSystem Lib "rcgranddogw32"()

'VA: 4EADB0

Private Declare Sub rc_VisitLicenseFile Lib "rcgranddogw32"()

'VA: 4EAD64

Private Declare Sub rc_DeleteDir Lib "rcgranddogw32"()

'VA: 4EAD1C

Private Declare Sub rc_DeleteFile Lib "rcgranddogw32"()

'VA: 4EACD4

Private Declare Sub rc_WriteFile Lib "rcgranddogw32"()

'VA: 4EAC6C

Private Declare Sub rc_ReadFile Lib "rcgranddogw32"()

'VA: 4EAC28

Private Declare Sub rc_CreateFile Lib "rcgranddogw32"()

'VA: 4EABE0

Private Declare Sub rc_CreateDir Lib "rcgranddogw32"()

'VA: 4EAB98

Private Declare Sub rc_GetRandom Lib "rcgranddogw32"()

'VA: 4EAB50

Private Declare Sub rc_ConvertData Lib "rcgranddogw32"()

'VA: 4EAB08

Private Declare Sub rc_SignData Lib "rcgranddogw32"()

'VA: 4EAAA8

Private Declare Sub rc_DecryptData Lib "rcgranddogw32"()

'VA: 4EAA60

Private Declare Sub rc_EncryptData Lib "rcgranddogw32"()

'VA: 4EAA18

Private Declare Sub rc_SetKey Lib "rcgranddogw32"()

'VA: 4EA9D4

Private Declare Sub rc_ChangePassword Lib "rcgranddogw32"()

'VA: 4EA988

Private Declare Sub rc_VerifyPassword Lib "rcgranddogw32"()

'VA: 4EA93C

Private Declare Sub rc_CheckDog Lib "rcgranddogw32"()

'VA: 4EA8E4

Private Declare Sub rc_GetDogInfo Lib "rcgranddogw32"()

'VA: 4EA89C

Private Declare Sub rc_GetProductCurrentNo Lib "rcgranddogw32"()

'VA: 4EA84C

Private Declare Sub rc_OpenDog Lib "rcgranddogw32"()

'VA: 4EA5A8

Private Declare Sub rc_CloseDog Lib "rcgranddogw32"()

'VA: 4EA494
```

## 函数查找方法

然后根据加密狗常规函数调用经验，使用文本批量查找工具对以上函数名进行查找。

常规加密狗验证，首先要查找设备，打开设备，读取数据，为了高效我们只定位以上几个关键API。

### 2. 示例

比如我们查找`rc_OpenDog`，这个是和狗建立连接，我们在所有代码中搜索，定位到`modReg.bas`文件的`Proc_30_13_5EBBF4`方法中，然后我们再搜索`Proc_30_13_5EBBF4`，发现这个子方法并没有被任何程序调用，所以我们就可以软件中并不是使用这个加密狗模块。

### 3. 最后通过排除法确定模块

最后通过排除法最终确定通过搜索`FD181OpenDev`，定位到了`modfd181.bas`文件中的`Proc_79_9_573054`方法。

所以我们确定在VB Decompiler中`code`段的`modfd181`类中就是处理加密狗数据的代码内容了。

## 4> 整理笔记1

`frmmain`的`Form_Load`负责弹窗/验证授权

`frmlogin`是免费版网络验证窗口

`frmzc`是硬盘版授权&加密狗授权验证窗口

`frmGetZCHao`是生成加密硬盘ID的窗口(生成注册号)

`ModRij`是调用对称加密解密的模块 `Proc_60`前缀标志

`modFD181`是加密狗通信模块 `Proc_79`前缀标志

## 3> 第一阶段总结

主框架的核心代码段分布已经完成，找到了主要的分析节点，接下来就是继续深入解析，并找到关键点。

在不更改源程序的情况下只有两种过验证的方法

1.通过生成硬盘绑定模式的注册信息来实现正版验证

技术要点：将软件改为硬盘版(需要改代码)，通过分析验证过程来伪造验证文件。[具体验证文件是什么还需深入分析]

2.通过伪造加密狗来实现验证(需要改代码，因为我已经有了加密狗版的安装程序，所以优先考虑这种方法)

技术要点：需要C++编写`FD181.dll`文件，补齐所有头文件

3.网络验证本身就是功能受限，所以不考虑

那么接下来的工作就是分析代码了

## 3> 分析过程[由深到浅,逆向推进]

### 4> 加密狗验证逆向推进

1.首先工具中(VB Decompiler，以下简称工具)，`code`段定位到`modFD181`类的`Proc_79`子方法组，选中子方法并在每个子方法右键，选择查看引用(`Find references`)，如果没有弹出窗口就说明这个子方法没有被使用过，最后结果为：

- `Proc_79_9`
- `Proc_79_10`
- `Proc_79_13`
- `Proc_79_16`

共4个子方法被调用过，而且均指向`Proc_30_15`，

双击后跳转到`modReg`的`Proc_30_15`方法，所以可以确定`modReg`类就是软件验证的类，重点关注！

`Proc_79_9`分析：将传入的16位`String`转成`byte`数组并传给`FD181OpenDev`来获取返回值，

通过官方资料SDK资料：

`aHR0cHM6Ly93d3cudGFvZG9jcy5jb20vcC00OTY5OTA0ODIuaHRtbA==`

返回`handle`(大于0的整数/指针)

`Proc_79_10`分析：效验密码，具体略

`Proc_79_13`分析：`FD181GetDID`：获取设备编号

`Proc_79_16`：`FD181DevRead`：读取数据

2.我们转到`Proc_30_15`，具体分析加密狗数据交互流程(需参考以上四个子方法)

```vb
Private Sub Proc_30_15_59651C
  'Data Table: 4979BC
  Dim var_A8 As Long
  Dim var_CC As Variant
  PopAdLdVar 'Ignore this
  var_A8 = Proc_79_9_573054(CVar("F92DB2AFE056B4B5"))   //获取设备指针,大于0
  If (var_A8  var_AC(2)) Then   //比较处理后数据(数据效验)
            var_8C = "ERR:NOTCMP"
          Else
            var_8C = var_98
          End If
        End If
      End If
    End If
  End If
  var_88 = var_8C
  Exit Sub
End Sub
```

总结：只需要按照上面代码及其调用的子方法代码梳理清楚并通过C++编译，并返回对应的数据即可完全实现加密狗的数据交互，

3.继续通过查找引用，查找`Proc_30_15`被调用的方法

```vb
Private Sub Proc_30_14_55447C(arg_10) '55447C

  'Data Table: 4979BC

  var_94 = CVar(Proc_30_15_59651C()) 'Variant

  Result = arg_10: Exit Sub

End Sub
```

然后继续查找引用，定位`Proc_30_14`被哪些方法引用了

结果如下：

- `frmzc.Combo1_Click`
- `modReg.Proc_30_0`
- `modReg.Proc_30_16`
- `frmGetZCHao.Combo1_Click`
- `frmDog.Tmer1_Timer`

由此可见，`Proc_30_14`就是所有检测获取加密狗数据的最终方法，也算是一个爆破点，只要在这里返回正确的加密数据即可，至于这个数据有什么限制还需要具体分析，这里不做过多说明。

当然不排除以上这些代码内容中有没有写入在其他目前未知的地方进行的全局变量的标志检测。

### 4> 硬盘授权模式逆向推进

1.继续上面提到的`frmGetZCHao`类`Form_Load`窗体加载事件和`Combo1_Click`下拉框点击事件中的`clsdno`类

查看引用`Proc_55`两个子方法，只在当前类调用，忽略

然后使用文本批量查找工具查找导出的工程，

搜索：

- `GetModelNumber`
- `GetSerialNumber`   硬盘序列号，这个很重要

在这四个`frmgetzchao`、`frmzc`、`modreg`、`modwmihdd`类中有使用

然后我们用查看引用逆推看哪些是未被使用的代码

`frmgetzchao`、`frmzc`这两个类是获取硬盘数据，不太可能是启动验证的代码块，所以我们暂时分析后两个

2.查看引用逆向跟踪引用

- `GetSerialNumber`
- `Proc_106_3(modWMIHDD)`
- `Proc_81_13(modDiscSize)`
- `frmzc.Combo1_Click`
- `frmzc.Command5_Click`
- `modReg.Proc_30_12`
- `frmGetZCHao.Combo1_Click`

最终还是回到了这三个方法里面

所以我们我们可以得出结论：

`modreg`是最有可能是后台验证数据的方法段。

3.我们分析一下这个申请码是怎么生成的

`frmGetZCHao`类的`Combo1_Click`点击事件
