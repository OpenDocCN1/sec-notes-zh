
# PowerShell 脚本

```
if(!([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] 'Administrator')) {
    Start-Process -FilePath PowerShell.exe -Verb RunAs -ArgumentList ("-NoExit",("cd {0} ;" -f $PSScriptRoot),("`"$($MyInvocation.MyCommand.Path)`" $($MyInvocation.UnboundArguments)"))
    Exit
   }

Set-Location -path "$(Get-Location)"
$signtool = "D:\signtool.exe"
$cert_km = "D:\cert\localhost-km.crt"
$cert_pk = "D:\cert\localhost-pk.pfx"
$pk_root = "D:\cert\localhost-root-ca.crt"
New-CIPolicy -FilePath SiPolicy.xml -Level RootCertificate -ScanPath "C:\Windows\System32\"
Add-SignerRule -FilePath .\SiPolicy.xml -CertificatePath $cert_km -Kernel -Update -Supplemental
Set-RuleOption -FilePath .\SiPolicy.xml -Option 0 -Delete   #0  - Enabled: UMCI
Set-RuleOption -FilePath .\SiPolicy.xml -Option 2 -Delete   #2  - Required: WHQL
Set-RuleOption -FilePath .\SiPolicy.xml -Option 12 -Delete  #12 - Required: Enforce Store Applications
Set-RuleOption -FilePath .\SiPolicy.xml -Option 3   #3  - Enabled: Audit Mode (Default)
Set-RuleOption -FilePath .\SiPolicy.xml -Option 5   #5  - Enabled: Inherit Default Policy
Set-RuleOption -FilePath .\SiPolicy.xml -Option 6   #6  - Enabled：Unsigned System Integrity Policy
Set-RuleOption -FilePath .\SiPolicy.xml -Option 9   #9  - Enabled：Advanced Boot Options Menu
Set-RuleOption -FilePath .\SiPolicy.xml -Option 10  #10 - Enabled：Boot Audit on Failure
Set-RuleOption -FilePath .\SiPolicy.xml -Option 16  #16 - Enabled: Update Policy No Reboot
Set-RuleOption -FilePath .\SiPolicy.xml -Option 17  #17 - Enabled: Allow Supplemental Policies
Set-CIPolicyVersion -FilePath .\SiPolicy.xml -Version 10.0.0.0
$xmlPath = "$(Get-Location)" + "\SiPolicy.xml"
$xmlPath_Modified = "$(Get-Location)" + "\SiPolicy_Modified.xml"
$xml = New-Object System.Xml.XmlDocument
$xml.Load("$xmlPath")
$nsManager = New-Object System.Xml.XmlNamespaceManager($xml.NameTable)
$nsManager.AddNamespace("sipolicy", "urn:schemas-microsoft-com:sipolicy")
$signers = $xml.SelectSingleNode("//sipolicy:Signers", $nsManager)

$newSigner1_0 = $xml.CreateElement("Signer", $xml.DocumentElement.NamespaceURI)
$newSigner1_0.SetAttribute("ID", "ID_SIGNER_KNOWNROOT_1_0")
$newSigner1_0.SetAttribute("Name", "Unknown Root")
$certRoot1_0 = $xml.CreateElement("CertRoot", $xml.DocumentElement.NamespaceURI)
$certRoot1_0.SetAttribute("Type", "Wellknown")
$certRoot1_0.SetAttribute("Value", "01")
$newSigner1_0.AppendChild($certRoot1_0)
$signers.AppendChild($newSigner1_0)

$newSigner2_0 = $xml.CreateElement("Signer", $xml.DocumentElement.NamespaceURI)
$newSigner2_0.SetAttribute("ID", "ID_SIGNER_KNOWNROOT_2_0")
$newSigner2_0.SetAttribute("Name", "Self-Signed Root")
$certRoot2_0 = $xml.CreateElement("CertRoot", $xml.DocumentElement.NamespaceURI)
$certRoot2_0.SetAttribute("Type", "Wellknown")
$certRoot2_0.SetAttribute("Value", "02")
$newSigner2_0.AppendChild($certRoot2_0)
$signers.AppendChild($newSigner2_0)

$newSigner14_0 = $xml.CreateElement("Signer", $xml.DocumentElement.NamespaceURI)
$newSigner14_0.SetAttribute("ID", "ID_SIGNER_KNOWNROOT_14_0")
$newSigner14_0.SetAttribute("Name", "Microsoft Trusted Root Store")
$certRoot14_0 = $xml.CreateElement("CertRoot", $xml.DocumentElement.NamespaceURI)
$certRoot14_0.SetAttribute("Type", "Wellknown")
$certRoot14_0.SetAttribute("Value", "14")
$newSigner14_0.AppendChild($certRoot14_0)
$signers.AppendChild($newSigner14_0)

$driversScenario = $xml.SelectSingleNode("//sipolicy:SigningScenario[@ID='ID_SIGNINGSCENARIO_DRIVERS_1']", $nsManager)
$allowedsigners_DRIVERS = $driversScenario.SelectSingleNode("./sipolicy:ProductSigners/sipolicy:AllowedSigners", $nsManager)
$allowedsigner1_0 = $xml.CreateElement("AllowedSigner", $xml.DocumentElement.NamespaceURI)
$allowedsigner1_0.SetAttribute("SignerId", "ID_SIGNER_KNOWNROOT_1_0")
$allowedsigners_DRIVERS.AppendChild($allowedsigner1_0)
$allowedsigner2_0 = $xml.CreateElement("AllowedSigner", $xml.DocumentElement.NamespaceURI)
$allowedsigner2_0.SetAttribute("SignerId", "ID_SIGNER_KNOWNROOT_2_0")
$allowedsigners_DRIVERS.AppendChild($allowedsigner2_0)
$allowedsigner14_0 = $xml.CreateElement("AllowedSigner", $xml.DocumentElement.NamespaceURI)
$allowedsigner14_0.SetAttribute("SignerId", "ID_SIGNER_KNOWNROOT_14_0")
$allowedsigners_DRIVERS.AppendChild($allowedsigner14_0)

$xml.Save($xmlPath_Modified)
ConvertFrom-CIPolicy -XmlFilePath .\SiPolicy_Modified.xml -BinaryFilePath .\SiPolicy.bin
& $signtool sign /v /p7 . /p7co 1.3.6.1.4.1.311.79.1 /fd sha256 /ac $pk_root /f $cert_pk SiPolicy.bin
Move-Item -Force -Path .\SiPolicy.bin.p7 -Destination .\SiPolicy.p7b
mountvol X: /s
Copy-Item -Force -Path .\SiPolicy.p7b -Destination X:\EFI\Microsoft\Boot\
Write-Host "(EFI分区)SiPolicy.p7b签名状态："
certutil.exe -asn X:\EFI\Microsoft\Boot\SiPolicy.p7b
mountvol X: /d
Read-Host -Prompt "内核代码证书规则SiPolicy.p7b构建已完成，按任意键删除临时文件并退出本程序。"
Remove-Item -Path .\SiPolicy.p7b, .\SiPolicy.bin, .\SiPolicy.xml, .\SiPolicy_Modified.xml
$host.SetShouldExit(0)
```

# 使用说明

保存代码到`ps1`后缀的脚本文件双击执行即可。

记得设置脚本前面的变量指定`signtool.exe`、`localhost-km.crt`、`localhost-pk.pfx`、`localhost-root-ca.crt`四个文件的位置。

`signtool.exe`是`Windows Software Development Kit (SDK)`的一部分，可以在网络上单独下载到。

# 项目介绍

据项目介绍

> 
CKS 的开关保存在`HKLM\SYSTEM\CurrentControlSet\Control\ProductOptions`键的`ProductPolicy`值里。

尽管管理员可以修改这个值，但是这个值在被修改后会立即恢复原状。这是因为在内核初始化完后，这个值只是内核里一个变量的映射，只有通过`ExUpdateLicenseData`这个内核API才能修改。而这个API只能在内核里被调用，或者通过`NtQuerySystemInformation`的`SystemPolicyInformation`功能号间接调用。很遗憾的是后者只有`Protected Process`才能成功调用。

所以我们只能在内核还尚未初始化完的时候修改 CKS 开关。有这个机会吗？有，Windows的Setup Mode可以给我们提供这个机会。

我已经写了一个程序来帮助我们打开 CKS，二进制程序是`EnableCKS.exe`。`EnableCKS.exe`会自动启动并开启

`CodeIntegrity-AllowConfigurablePolicy`

`CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners`

但微软在新系统中删除了`CodeIntegrity-AllowConfigurablePolicy`这个产品策略，运行`EnableCKS.exe`后程序无法开启`CodeIntegrity-AllowConfigurablePolicy`会导致无限重启。（如果已经无限重启，可进PE的注册表编辑器，点击`HKEY_LOCAL_MACHINE`，再点击文件-加载配置单元，选择`C:\Windows\System32\config`文件夹下的`SYSTEM`文件，修改加载的注册表`HKEY_LOCAL_MACHINE\SYSTEM\Setup`下`SetupType`的值为0即可。修改完成后记得卸载配置单元保存修改。）

我用了另一个项目`valinet/ssde`（`https://github.com/valinet/ssde`），这个项目作者编译的`ssde_enable.exe`解决了这个错误，可以正常开启`CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners`并重启。

# CustomKernelSigners 持久化

据项目介绍

> 
CustomKernelSigners持久化

重新进入正常模式后，你应该就可以加载由`localhost-km.pfx`签署的驱动了。但是别高兴得太早，大约在10分钟之内，CKS 会被`sppsvc`服务重置为关闭，除非你的`Windows10`是中国政府特供版。但不用担心，关闭还得等重启后才会实际生效。

所以我们得趁这个机会，加载自己编写的驱动，通过不断调用`ExUpdateLicenseData`来持久化 CKS。

`ssde`项目的作者同样编译了一个驱动`ssde.sys`来持久化 CKS，但是这个驱动没有签名。需要通过以下命令用`localhost-km`证书签名`ssde.sys`

```
signtool sign /v /fd sha1 /ac localhost-root-ca.crt /f localhost-km.pfx /tr "http://timestamp.digicert.com" ssde.sys

signtool sign /v /fd sha256 /as /ac localhost-root-ca.crt /f localhost-km.pfx /tr "http://timestamp.digicert.com" ssde.sys
```

将签名完成的`ssde.sys`复制到`%SystemRoot%\System32\drivers\`目录下（一般都是`C:\Windows\System32\drivers\`）。

然后用管理员权限运行以下命令
