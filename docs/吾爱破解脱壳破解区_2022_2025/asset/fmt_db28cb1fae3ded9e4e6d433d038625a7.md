
# [分享] 不使用第三方工具或漏洞让Windows系统加载自己签名的内核驱动

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1971554
- 抓取时间: 2026-11-01T22:22:03

修改了生成内核代码证书规则的代码，设置规则为审计模式，解决了无法加载没有微软签名的第三方驱动的问题。

看论坛内帖子“从Sandboxie源码分析软件注册机制及逆向思路”（https://www.52pojie.cn/thread-1793118-1-1.html）发现最后加载驱动时要关闭驱动程序强制签名，而关闭强制签名会被部分游戏或程序检测到导致程序拒绝启动。如果能够不关闭强制签名去加载内核驱动，对系统的影响是最小的。

（以下方法在Windows 10 企业版 21H2 19044.5011测试通过）

在网络上查资料找到了这个项目“HyperSine/Windows10-CustomKernelSigners”（https://github.com/HyperSine/Windows10-CustomKernelSigners），

## 项目介绍

> 
`Custom Kernel Signers`(CKS) 是Windows10（可能从1703开始）支持的一种产品策略。这个产品策略的全名是`CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners`，它允许用户自定义内核代码证书，从而使得用户可以摆脱“驱动必须由微软签名”的强制性要求。

如果一个Windows10 PC满足下列条件：

1. 产品策略`CodeIntegrity-AllowConfigurablePolicy-CustomKernelSigners`是开启的。 （也许`CodeIntegrity-AllowConfigurablePolicy`也要开启。）
2. `SecureBoot`也是开启的。

那么任何拥有该PC的UEFI Platform Key的人都可以自定义内核代码证书。这意味着，在不开启调试模式、不开启`TestSigning`、不关闭`DSE`的情况下，他可以使系统允许自签名驱动的加载。

于是跟随 项目内教程 创建了3个证书（`localhost-root-ca`，`localhost-km`，`localhost-pk`），`localhost-root-ca`作为根证书用于签发`localhost-km`和`localhost-pk`，`localhost-km`用于给要加载的内核驱动签名，`localhost-pk`作为PK 证书用于导入UEFI 固件。

为了导入自己PK 证书需要先将安全启动重置为Setup Mode，重置后现有PK 证书会被清除。如果电脑的BIOS设置可以直接导入crt格式PK 证书直接导入就行，不必重置进Setup Mode。部分BIOS只支持通过AUTH文件导入PK 证书，这种就需要重置为Setup Mode。

生成AUTH文件需要`efitools`，`efitools`只能在Linux系统上使用。Windows电脑可以通过`WSL 1`（不需要`WSL 2`）安装一个Ubuntu，在Ubuntu内安装`efitools`的命令是

```shell
sudo apt install efitools
```

`efitools`的软件包下载网页：

https://packages.ubuntu.com/zh-cn/jammy/amd64/efitools/download

`efitools`二进制文件下载网页：

https://archlinux.org/packages/extra/x86_64/efitools/（下载地址：https://archlinux.org/packages/extra/x86_64/efitools/download/）

以下是在WSL内用`efitools`生成AUTH文件的代码的示例（注意换行符为LF）

```shell
cd ${0%/*}

cert_path=/mnt/d/cert

signing_cert=$cert_path/localhost-pk

cert-to-efi-sig-list -g "$(cat GUID.txt)" "$cert_path/localhost-pk.crt" "localhost-pk.esl"

echo PK.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt PK localhost-pk.esl PK.auth

echo noPK_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt PK /dev/null noPK_rplc.auth

echo noPK_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt PK /dev/null noPK_add.auth

cert_path=/mnt/d/cert

signing_cert=$cert_path/localhost-pk

openssl x509 -inform der -outform pem -in "microsoft corporation kek 2k ca 2023.crt" -out "microsoft corporation kek 2k ca 2023.pem"

openssl x509 -inform der -outform pem -in "MicCorKEKCA2011_2011-06-24.crt" -out "MicCorKEKCA2011_2011-06-24.pem"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "microsoft corporation kek 2k ca 2023.pem" "microsoft corporation kek 2k ca 2023.esl"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "MicCorKEKCA2011_2011-06-24.pem" "MicCorKEKCA2011_2011-06-24.esl"

cat "microsoft corporation kek 2k ca 2023.esl" "MicCorKEKCA2011_2011-06-24.esl" > "kek.esl"

echo KEK_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt KEK kek.esl KEK_rplc.auth

echo KEK_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt KEK kek.esl KEK_add.auth

echo noKEK_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt KEK /dev/null noKEK_rplc.auth

echo noKEK_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt KEK /dev/null noKEK_add.auth

cert_path=/mnt/d/cert

signing_cert=$cert_path/localhost-pk

openssl x509 -inform der -outform pem -in "MicCorUEFCA2011_2011-06-27.crt" -out "MicCorUEFCA2011_2011-06-27.pem"

openssl x509 -inform der -outform pem -in "microsoft uefi ca 2023.crt" -out "microsoft uefi ca 2023.pem"

openssl x509 -inform der -outform pem -in "MicWinProPCA2011_2011-10-19.crt" -out "MicWinProPCA2011_2011-10-19.pem"

openssl x509 -inform der -outform pem -in "windows uefi ca 2023.crt" -out "windows uefi ca 2023.pem"

openssl x509 -inform der -outform pem -in "microsoft option rom uefi ca 2023.crt" -out "microsoft option rom uefi ca 2023.pem"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "MicCorUEFCA2011_2011-06-27.pem" "MicCorUEFCA2011_2011-06-27.esl"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "microsoft uefi ca 2023.pem" "microsoft uefi ca 2023.esl"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "MicWinProPCA2011_2011-10-19.pem" "MicWinProPCA2011_2011-10-19.esl"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "windows uefi ca 2023.pem" "windows uefi ca 2023.esl"

cert-to-efi-sig-list -g "77fa9abd-0359-4d32-bd60-28f4e78f784b" "microsoft option rom uefi ca 2023.pem" "microsoft option rom uefi ca 2023.esl"

cat "MicCorUEFCA2011_2011-06-27.esl" "microsoft uefi ca 2023.esl" "MicWinProPCA2011_2011-10-19.esl" "windows uefi ca 2023.esl" "microsoft option rom uefi ca 2023.esl" > "db.esl"

echo DB_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DB db.esl DB_rplc.auth

echo DB_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DB db.esl DB_add.auth

echo noDB_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DB /dev/null noDB_rplc.auth

echo noDB_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DB /dev/null noDB_add.auth

cert_path=/mnt/d/cert

signing_cert=$cert_path/localhost-pk

echo DBX_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DBX dbxupdate_x64.bin DBX_rplc.auth

echo DBX_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DBX dbxupdate_x64.bin DBX_add.auth

echo noDBX_rplc.auth

sign-efi-sig-list -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DBX /dev/null noDBX_rplc.auth

echo noDBX_add.auth

sign-efi-sig-list -a -g "$(cat GUID.txt)" -k $signing_cert.key -c $signing_cert.crt DBX /dev/null noDBX_add.auth
```

`cert_path`变量设置签名证书存放的路径，`signing_cert`变量设置签名证书的文件名。

`db.esl`使用PK或者KEK签名均可。

`no`前缀为清空对应证书，`_rplc`代表通过“replace”方式加载，`_add`代表通过“Add new cert”方式加载。

需要在运行Shell的目录下创建一个`GUID.txt`，文件内容是自己生成的8-4-4-4-12格式的GUID（如微软的所有者 GUID：`77fa9abd-0359-4d32-bd60-28f4e78f784b`）。

使用PowerShell命令

```powershell
New-Guid
```

可以生成一个新的GUID。

Shell的目录下还需要有base-64编码的证书文件`localhost-pk.crt`及其私钥`localhost-pk.key`。

涉及到的微软证书文件在文章最后有下载地址。

## 内核代码证书规则构建

设置好PK证书后就要构建内核代码证书规则。一个现成的内核代码证书规则文件（https://www.geoffchappell.com/notes/windows/license/selfsign.xml.htm） 。

这个规则文件需要包含自己生成的`localhost-km`证书。

转换`localhost-pk.crt`为签名可用的`localhost-pk.pfx`的代码

```shell
openssl pkcs12 -export -in localhost-pk.crt -inkey localhost-pk.key -out localhost-pk.pfx
```

导出`localhost-pk.pfx`时会提示输入密码，全部留空直接回车即可。

规则生成并签名后需要移动到当前系统的EFI目录下（以下提供的命令可自动完成）。

以下是构建内核代码证书规则的PowerShell代码

```powershell

```
