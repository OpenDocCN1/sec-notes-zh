
# 查看类名与成员表

```
properties_struct {
    pointer   base_class;
    qword     unknown1;
    char *    class_name;
    dword     unknown2; 
    dword     unknown3;
    .......   properties_list
}
```

可以看到，第3个属性就是类名了，下面以图説明：

![](https://attach.52pojie.cn/forum/202208/15/081930ubxa885x1vab7018.jpg)

`small_129.jpg` (316.88 KB, 下载次数: 237) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzOHxiODY2ZTIxYnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes) 2022-8-15 08:19 上传

上图中，`rcx` 指向一个对象，`[rcx + 0x08]` 就是属性和成员表了。

![](https://attach.52pojie.cn/forum/202208/15/081933azjnnnn3pnnfzbij.jpg)

`small_129_1.jpg` (308.29 KB, 下载次数: 232) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjAzOXwxMmVlMjJiYnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes) 2022-8-15 08:19 上传

在数据区按 `CTRL+G`，输入 `[rcx+0x08]` 定位到属性和成员表，如上图所示位置，其中第3个属性是一个地址，指向类名，也就是指向 `[[rcx + 0x08] + 0x10]`。

![](https://attach.52pojie.cn/forum/202208/15/081935d9tlxo9gld8twfkd.jpg)

`small_129_2.jpg` (312.3 KB, 下载次数: 243) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjA0MHw3MGQ1MDIwMnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes) 2022-8-15 08:19 上传

再在数据区按 `CTRL+G`，输入 `[[rcx+0x08] + 0x10]` 就可以看到类名，这样只要知道 `rcx` 是一个对象，直接 `[[rcx+0x08] + 0x10]` 就会显示类名了。

还有就是它的对类的方法和属性的名称，函数的返回值和参数的类型，也都可以查到，如下图所示是 `XMLDocument` 的一个方法：

![](https://attach.52pojie.cn/forum/202208/15/081938zk2hmt2glxxxtlgg.jpg)

`small_130.jpg` (293.23 KB, 下载次数: 243) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjA0MXw5MmRhMzU4NXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes) 2022-8-15 08:19 上传

---

# 注册机

## RSA 签名插件

下面我们进入注册机的主题了，生成一个可用的授权文件（当然前提是 RSA 的公钥要换成我们自己的）。

这里还要説明一下，它那个 RSA 签名过程与标准的开源的不一样，我作了对比，有些差异，并且它的 RSA 签名过程是通过 `Plugin` 来完成的，我们也要通过这种方式来完成，否则签名是通不过的。

还有就是系统自带了 RSA 签名的函数，但这个函数生成的签名也是通不过的，这个函数如下：

```
RSASign(data As MemoryBlock, privateKey As MemoryBlock) As MemoryBlock
```

所以我们也只能调用它的插件来实现签名，但是系统没有带这个插件，其插件目录中没有这个 RSA 签名的插件。但是系统的库目录（`Xojo Libs`）带的 RSA 签名的动态库，有这个就可以了，我们自己来依葫芦画瓢制作一个，

我们查看一下插件目录（`Xojo Libs`），有系统自带的插件，其实这些插件是 ZIP 格式的文件，我们用压缩软件打开一个插件，就可以看到，其结构很简单，就是包括不同编译目标平台的动态库。

![](https://attach.52pojie.cn/forum/202208/15/081940g8ky6lm6cik938k6.jpg)

`small_131.jpg` (30.77 KB, 下载次数: 235) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjA0MnxkODIzNDQ5MXwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes) 2022-8-15 08:19 上传

我们只有 `x86_64` 的库，只要这个就可以，如下图所示，我们自制了一个 `OpenSSL` 插件：

![](https://attach.52pojie.cn/forum/202208/15/081943a00jsas7ja0juj4p.jpg)

`small_132.jpg` (39.23 KB, 下载次数: 235) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU0NjA0M3wxYTNmMTg3ZnwxNzkzNTc0NjY3fDE2ODA4NnwxNjc1MzY4&nothumb=yes) 2022-8-15 08:19 上传

插件中的 DLL 来自其 `Libs` 目录中的库文件。

保存这个文件到其安装目录下的 `Plugins` 子目录，并重新启动 IDE 就可以用了。具体怎么用可看看其 `plugin SDK` 的文档，有 c++ 的接口説明，再用 IDA 看看 DLL 就差不多理解了。

## Blowfish 加密

还有就是前面説过的 Blowfish 的密码长度超过正常的长度，所以得用其内置的 `_Blowfish` 类来处理 Blowfish 的加密，不然也是通不过的，而这个 `_Blowfish` 类是不公开的，也就上没有文档説明，其文档中的两个 Blowfish 相关函数：

```
BlowFishDecrypt(publicKey As String, data As MemoryBlock, blockMode As Crypto = BlockModes.CBC, initializationVector As MemoryBlock) As MemoryBlock
BlowFishEncrypt(publicKey As String, data As MemoryBlock, blockMode As Crypto = BlockModes.CBC, initializationVector As MemoryBlock) As MemoryBlock
```

也不是通过这个类来实现的，这两个函数也不能处理超过长度的密码。

## 注册机主要代码

以下是注册机的主要代码：

```
const NID_SHA1 as Integer = 64 ////&H40

var features() as int64 = Array(1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 2147479552)

var crlf as String = EndOfLine.UNIX

const xml_cryptoKey as String = "+%%oooi4i4"

//// email and password

var email as String = txtEmail.Text.Trim.Lowercase

if email = "" then

  MessageBox "Please Input a Email Address!!!"

  exit sub

end if

//if lstEditions.SelectedRowIndex = ListBox.NoSelection then

//MessageBox "Please select edition features!!!"

//exit sub

//end if

///// ==================================================================================================================

/////  生成序列号

var password as string = email

//if password = "" then

//MessageBox "Password(email address) is Null, Input eMail Address First!!!!!!"

//exit sub

//end if

var mem as MemoryBlock = new MemoryBlock(24)

mem.LittleEndian = false

// unused

mem.Int8Value(0) = &H20

// release date

// 2022-04-06

/// shipped_date

var today as date = new Date

var shippedDate as date = dtpShippedDate.SelectedDate

var year as UInt16 = shippedDate.Year

var wks as UInt8 = shippedDate.WeekOfYear

var wDays as UInt8 = shippedDate.DayOfWeek

/// shipped date

mem.UInt16Value(1) = Year

mem.UInt8Value(3) = wks  // yestoday

mem.UInt8Value(4) = wDays

/// expiration = valid_days - today - 1

var expiredDays as Integer = CType((dtpLicensedDate.SelectedDate.SecondsFrom1970 - dtpShippedDate.SelectedDate.SecondsFrom1970) / 86400, Integer)

if expiredDays  (&H7FFE - today.Day) then

  MessageBox "Licensed Date too bigger!!!"

  Exit sub

end if

//mem.Int16Value(5) = expiredDays  //// key的有效天数，默认 365 天，生成的授权文件在 365 天内加入都 有效，否则会失效

mem.Int16Value(5) = &H7FFF - 365

// loop one, [7]>1 则 raise UnsupportedFormatException

mem.Int8Value(7) = 1  // not loop

// unused

mem.Int8Value(8) = &H20

mem.Int8Value(9) = &H20

// SubscriptionID

mem.Int32Value(10) = Integer.FromString(txtSubscriptionID.Text.Trim)//&H88888888

// serial_index

mem.Int16Value(14) = Integer.FromString(txtSerialIndex.Text.Trim) //&H6667

// unused

mem.Int8Value(16) = &H20

//feature

var feature as UInt32 = 0

for i as Integer = 0 to lstEditions.LastRowIndex 

  if lstEditions.RowAt(i).Selected then

    feature = feature or features(i)

  end if

next i

if bitwise.BitAnd(feature, &H36) = 0 then        //// Desktop, Web, iOS, Console 必选

  feature = &H476  // Desktop, Web, iOS, Console, Pro, Pro Plus

end if

mem.UInt32Value(17) = feature//&H7FFFFFFE

// tag

mem.Int8Value(21) = &H2C    ///  逗号 ','，序列号有效性标志

// unused

mem.Int8Value(22) = &H20

mem.Int8Value(23) = &H20 

//Crypto.BlowFishEncrypt(key As String, data As MemoryBlock, blockMode As BlockModes = BlockModes.CBC, initializationVector As MemoryBlock) As MemoryBlock

var mem2 as MemoryBlock = Crypto.BlowFishEncrypt(password, mem, Crypto.BlockModes.ECB, nil)

var serial_hex as string = mem2.StringValue(0, mem2.Size)

var serial_str as string = EncodeHex(serial_hex)

var serial_xml as string = serial_str.Middle(0, 8) + "-" + serial_str.Middle(8, 8) + "-" + serial_str.Middle(16, 8) + "-" + serial_str.Middle(24, 8) + "-" + serial_str.Middle(32, 8) + "-" + serial_str.Middle(40, 8)

////// debug

////MessageBox "SerialNo: " + serial_xml

////// ==================================================================================================================

/////  生成序列号和地址信息的签名

var privateKey as String = Self.rsa_private_key.ReplaceLineEndings(EndOfLine.UNIX)  //// 取得私钥

if privateKey = "" then 

  MessageBox "RSA private key is null!!!!"

  exit sub

end if

////  用户名，一般为电脑登陆名

var userName as String = txtName.Text.Trim

if userName = "" then

  userName = app.LoggedInUserName   ///

end if

/// 生成地址信息

var address as String = txtName.Text + crlf + txtAddress1.Text + crlf + txtAddress2.Text + crlf + txtCity.Text + "," + txtProvince.Text + "," + txtPostCode.Text + crlf + txtCountry.Text
```
