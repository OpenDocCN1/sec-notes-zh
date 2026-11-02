
## 生成RSA签名文本

```
address = address.ReplaceLineEndings(EndOfLine.UNIX)

var msg as String = serial_xml + crlf + address

var signRet as String = ""

var private_key_pem as String = "-----BEGIN PRIVATE KEY-----" + EndOfLine.UNIX + privateKey + EndOfLine.UNIX + "-----END PRIVATE KEY-----"

//var public_key_pem as String = "-----BEGIN PUBLIC KEY-----" + EndOfLine.UNIX + publicKey + EndOfLine.UNIX + "-----END LUBLIC KEY-----"

////  注意：需要安装插件 OpenSSLPlugin.xojo_plugin，否则不能调用下面4个OpenSSL函数，该插件可自制

////  注意：需要安装插件 OpenSSLPlugin.xojo_plugin，否则不能调用下面4个OpenSSL函数，该插件可自制

////  注意：需要安装插件 OpenSSLPlugin.xojo_plugin，否则不能调用下面4个OpenSSL函数，该插件可自制

var rsa as Ptr =  OpenSSL.RSA_New()

rsa = OpenSSL.PEM_read_RSAPrivateKey(private_key_pem, rsa)

var isRsaSignOK as Boolean = OpenSSL.RSA_Sign(NID_SHA1, msg, signRet, rsa)   ////  生成RSA签名: signRet

OpenSSL.RSA_Free(rsa)

var signRet_str as String = ""

if isRsaSignOK then

  signRet_str = EncodeHex(signRet)

  //// debugInfo

  ////MessageBox "RSA Sinature return: " + crlf + signRet_str

else

  MessageBox "Generate RSA Signature failure!!!!!!"

  exit sub

end if

if signRet_str.Length <> 512 then

  MessageBox "Encode RSA Signature failure!!!!!!"

  exit sub

end if

////// debug

///taLicenseKeyContent.Text = signRet_str

/////==================================================================================================================
```

## 生成 XML 文件的 signature 标签内容

```
var sign_pass As MemoryBlock = new MemoryBlock(64)

sign_pass.LittleEndian = False

//MD5

var md5Value As MemoryBlock = Crypto.MD5(password)

sign_pass.StringValue(0, 16) = md5Value.StringValue(0,16)

//Serial

sign_pass.StringValue(16, 24) = serial_hex

var signature_password_constant as String = password_retrieval_table  //// 含一定的字符串：“CAF", "BAD", "11", "8", "B", "3", "7", "D", "F", "C"等

var signature_verify as String = signRet_str

if signature_verify = "" then

  MessageBox "RSA Signature First!!!!!"

  exit sub

end if

var pass_str as String 

if chkRandomPwdBase.Value then

  //var randBytes as MemoryBlock = Crypto.GenerateRandomBytes(256)

  //var signature_password_random as String = EncodeHex(randBytes.StringValue(0, 256), false)

  var signature_password_random as String = getRandomString()  //// 生成随机的密码检索表

  //// debugInfo

  //// MessageBox "random string: " + signature_password_random

  pass_str = signature_password_random     //// 随机

else

  pass_str = signature_password_constant    //// 固定

end if

var sign_verify_str as String = signature_verify 

var sign_verify_str_hex as String = app.MyDecodeHex(sign_verify_str)  // 16 进制的签名

if pass_str.Length <> 512 then

  MessageBox "pass base string length is not 512 bytes!"

  exit sub

end if

if sign_verify_str.Length <> 512 then

  MessageBox "signature verity string length is not 512 bytes!"

  exit sub

end

//MessageBox "sign_verify_str.length = " + sign_verify_str.Length.ToString

//MessageBox "sign_verify_str_hex.length = " + sign_verify_str_hex.Length.ToString

while (sign_verify_str_hex.length mod 8) <> 0

  sign_verify_str_hex = sign_verify_str_hex + " "

wend

var count as Integer  = 0

var min_val as Integer = 0

var sum_val as Integer = 0

var position as Integer = 0

var start as Integer = 0

var pow_val as Double = 0.0

var str_tmp1 as string = ""

var str_tmp2 as string = ""

var str_tmp3 as string = ""

/// byte(40)

count = CountOfStr(pass_str, "A")

sign_pass.Int8Value(40) = count

/// byte(41)

str_tmp1 = pass_str.Right(1)

str_tmp2 = MidStr(pass_str, pass_str.Length-2, 1)

position = charAt(pass_str, str_tmp2)  // Instr()

count = CountOfStr(pass_str, str_tmp1, position)

sign_pass.Int8Value(41) = count

//messagebox str_tmp1 + ", " + str_tmp2 + "," + position.ToString

/// byte(42)

count = CountOfStr(pass_str, "CAF")

sign_pass.Int8Value(42) = count

//messagebox "count of ""CAF"": " + count.ToString

/// byte(43)

count = CountOfStr(pass_str, "8")

sign_pass.Int8Value(43) = count

//messagebox "count of ""8"": " + count.ToString

/// byte(44)

position = charAt(pass_str, "B")  // Instr()

min_val = Min(position * 8, 255)

sign_pass.Int8Value(44) = min_val

//messagebox "min of InStr(""B"")*3 between ""255"": " + min_val.ToString

/// byte(45)

count = CountOfStr(pass_str, "BAD")

sign_pass.Int8Value(45) = count

//messagebox "count of ""BAD"": " + count.ToString

//// byte(46)

position = charAt(pass_str, "3")  // Instr()

count = CountOfStr(pass_str, "7", position)

min_val = Min(count, 12)

sign_pass.Int8Value(46) = min_val

//messagebox "min of InStr(""7"", InStr(""3"")) between ""12"": " + min_val.ToString

/// byte(47)

sign_pass.Int8Value(47) = 9   /// '\t'

/// byte(48)

sign_pass.Int8Value(48) = password.Length   //  用户密码长度

/// byte(49)

str_tmp1 = MidStr(pass_str, 21, 1)

str_tmp2 = MidStr(pass_str, 203, 1)

str_tmp3 = MidStr(pass_str, 48, 1)

sum_val = str_tmp1.Asc + str_tmp2.Asc + str_tmp3.Asc

sign_pass.Int8Value(49) = sum_val

//MessageBox str_tmp1 + ", " + str_tmp2 + ", " + str_tmp3

/// byte(50)

position = charAt(pass_str, "11")  // Instr()

if position > 0 then

  sign_pass.Int8Value(50) = 1

else

  sign_pass.Int8Value(50) = 0  ///  默认就是0，可跳过

end if

/// byte(51)

sign_pass.Int8Value(51) = 42

/// byte(52)

sign_pass.Int8Value(52) = sign_pass.Int8Value(0)   /// byte(52) = MD5Bytes[0]

/// byte(53)

sign_pass.Int8Value(53) = sign_pass.Int8Value(31)  * 7  /// byte(53) = serialBytes[15] * 7

/// byte(54)

count = CountOfStr(pass_str, "D", 127)

sign_pass.Int8Value(54) = count

/// byte(55)

str_tmp1 = MidStr(pass_str, pass_str.Length-2, 1)

str_tmp2 = pass_str.Right(1)

position = charAt(pass_str, str_tmp2)

count = CountOfStr(pass_str, str_tmp1, position)

sign_pass.Int8Value(55) = count

//MessageBox str_tmp1 + ", " + str_tmp2 + ", pos = " + position.ToString + ", count = " + count.ToString

/// byte(56) ~ byte(63)

position = charAt(pass_str, "F")

count = CountOfStr(pass_str, "C", 1)

pow_val = Pow(position, count)

sign_pass.DoubleValue(56) = pow_val

var sign_pass_string as String = sign_pass.StringValue(0, 64)

///  enrypted signature_verify_part

var sign_verify_data as MemoryBlock = new MemoryBlock(sign_verify_str_hex.Length)

sign_verify_data.StringValue(0, sign_verify_str_hex.Length) = sign_verify_str_hex

////  加密

var bf as _Blowfish = new _Blowfish(sign_pass_string)

//MessageBox "sign_verify_str_hex.length = " + sign_verify_str_hex.Length.ToString

var data0 as MemoryBlock = new MemoryBlock(sign_verify_str_hex.Length)

data0.StringValue(0, sign_verify_str_hex.Length) = sign_verify_str_hex

var data as MemoryBlock = new MemoryBlock(sign_verify_str_hex.Length)

data.StringValue(0, sign_verify_str_hex.Length) = sign_verify_str_hex

data.LittleEndian = False

bf.Encipher(data)  ///  加密

//// 256

var xml_signature as String = ""

var encrypted_signature as String

if data <> nil then

  //MessageBox encrypt_verify_data.Size.ToString 

  encrypted_signature = EncodeHex(data.StringValue(0, data.Size), false)

  xml_signature = pass_str + encrypted_signature

  //MessageBox "encryped signature: "+ crlf + encrypted_signature

  //MessageBox "signature ok!!!"

else

  MessageBox "XML Signature is null"

  exit sub

end if

////// debug

////taLicenseKeyContent.Text = xml_signature

/////==================================================================================================================
```

## 生成 XML 文件内容

```
if encrypted_signature = "" then

  MessageBox "XML Signature is empty."

  exit sub

end if

if encrypted_signature.Length <> 512 then

  MessageBox "Ecrypted Signature String Length not equal 512"

  exit sub

end if

//if pass_str.Length <> 512 then

//MessageBox "Password Base String Length not equal 512"

//exit sub

//end if

//// XML 文件

var xml_str  as String = "" + EndOfLine.Unix +  "" + EndOfLine.Unix + ""

var xml as XmlDocument = new XmlDocument()

xml.PreserveWhitespace = true

xml.LoadXml(xml_str)

var xml_element as XmlElement = xml.DocumentElement

var n as Integer = xml_element.ChildCount
```
