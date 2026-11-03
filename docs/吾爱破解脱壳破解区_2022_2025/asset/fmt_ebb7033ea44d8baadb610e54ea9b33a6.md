
## 启动时签名校验分支分析

```
__text:0x10012D418     ADRL         X0, $s6Reveal30ApplicationBuildTimestampErrorON ; type metadata for ApplicationBuildTimestampError
__text:0x10012D420     MOV          X2, #0
__text:0x10012D424     MOV          W3, #0
__text:0x10012D428     BL           _swift_allocError
__text:0x10012D42C     MOV          X19, X0
__text:0x10012D430     MOV          X21, X0
__text:0x10012D434     BL           _swift_willThrow
```

此分支在地址`sub_10012CFA0+1D4`有被引用，以下是引用处的汇编片段。开头调用了`sub_100208AA4`和`sub_1002080A0`，它们内部又有对`_SecCodeCopySigningInformation`与`_kSecCodeInfoTimestamp`的调用，这两函数分别负责提取应用的代码签名信息和签名时间戳。因而可得，该分支判断背后实际建立在签名校验结果的基础之上。

```
__text:0x10012D12C     BL       sub_100208AA4
__text:0x10012D130     CBNZ     X21, loc_10012D3F8
__text:0x10012D134     MOV      X25, X0
__text:0x10012D138     STUR     X21, [X29,#var_80]
__text:0x10012D13C     STUR     X23, [X29,#var_60]
__text:0x10012D140     MOV      X0, X20             ; id
__text:0x10012D144     BL       _objc_release
__text:0x10012D148     MOV      X8, X28
__text:0x10012D14C     MOV      X0, X25
__text:0x10012D150     BL       sub_1002080A0
__text:0x10012D154     MOV      X0, X25
__text:0x10012D158     BL       _swift_bridgeObjectRelease
__text:0x10012D15C     LDR      X8, [X26,#0x30]
__text:0x10012D160     MOV      X0, X28
__text:0x10012D164     MOV      W1, #1
__text:0x10012D168     MOV      X2, X19
__text:0x10012D16C     BLR      X8
__text:0x10012D170     CMP      W0, #1
__text:0x10012D174     B.EQ     loc_10012D408
```

之后在地址`0x10012D170`，程序将寄存器`W0`的值与1进行比较，随即分支判断随之生效，若`W0`的值等于1，则跳转至`loc_10012D408`。从该路径继续执行至`loc_10012D438`，最终触发`ApplicationBuildTimestampError.noTimestamp`错误。

因此，要规避该异常，只需将原本与1的比较改为与0进行比较，即可阻断报错分支。

```
__text:0x10012D170     CMP    W0, #0
```

## 3> 验证废除

成功绕过启动时的代码签名校验后，应用现已能够正常运行。下面正式分析 JWT 签名验证机制，并尝试将其彻底废除。

根据开头的分析，服务器端对返回的 JWT Token 采用了 RS256 签名算法。而对于 Apple 应用，执行此类非对称加密签名验证的标准常用途径，就是调用 `Security` 框架中的 `SecKeyVerifySignature` 函数。所以，`SecKeyVerifySignature` 自然成为本次验证废除的关键。

### 定位 `SecKeyVerifySignature`

```
__stubs:0x100C4AECC ; Boolean __cdecl SecKeyVerifySignature(SecKeyRef key, SecKeyAlgorithm algorithm, CFDataRef signedData, CFDataRef signature, CFErrorRef *error)
__stubs:0x100C4AECC _SecKeyVerifySignature  ; CODE XREF: sub_1009521EC+50↑p
__stubs:0x100C4AECC                         ; sub_1009524D4+4C↑p
__stubs:0x100C4AECC     ADRP    X16, #_SecKeyVerifySignature_ptr@PAGE
__stubs:0x100C4AED0     LDR     X16, [X16,#_SecKeyVerifySignature_ptr@PAGEOFF]
__stubs:0x100C4AED4     BR      X16 ; __imp__SecKeyVerifySignature
```

为精准定位哪些地址上调用了它，我们直接在 LLDB 中对 `SecKeyVerifySignature` 设置断点，然后运行应用。

### 设置断点并运行

```
(lldb) br s -n SecKeyVerifySignature
Breakpoint 1: where = Security`SecKeyVerifySignature, address = 0x00000001a502e15c
(lldb) r
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x00000001a502e15c Security`SecKeyVerifySignature
Security`SecKeyVerifySignature:
->  0x1a502e15c :  pacibsp
Target 0: (Reveal) stopped.
```

执行 `finish` 命令至调用 `SecKeyVerifySignature` 的返回点。

```
(lldb) finish
* thread #1, queue = 'com.apple.main-thread', stop reason = step out
    frame #0: 0x00000001002369b4 Reveal`___lldb_unnamed_symbol18939 + 80
Reveal`___lldb_unnamed_symbol18939:
->  0x1002369b4 : mov    x20, x0
Target 0: (Reveal) stopped.
```

### 查看调用栈

查看相应调用栈。

```
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = step out
  * frame #0: 0x00000001002369b4 Reveal`___lldb_unnamed_symbol18939 + 80
    frame #1: 0x00000001002340fc Reveal`___lldb_unnamed_symbol18827 + 216
    frame #2: 0x000000010025c0b0 Reveal`___lldb_unnamed_symbol19999 + 376
    frame #3: 0x000000010025a1f8 Reveal`___lldb_unnamed_symbol19980 + 244
    frame #4: 0x000000010025060c Reveal`___lldb_unnamed_symbol19496 + 764
    frame #5: 0x000000010024c718 Reveal`___lldb_unnamed_symbol19457 + 1248
    frame #6: 0x000000010012d2c4 Reveal`___lldb_unnamed_symbol12335 + 804
    frame #7: 0x000000010003f604 Reveal`___lldb_unnamed_symbol6634 + 196
    frame #8: 0x000000010003f920 Reveal`___lldb_unnamed_symbol6635 + 112
    ....
```

### 反汇编确认调用点

通过 `dis` 展示反汇编指令，可确定 `0x1002369b4-4` 地址正是调用 `SecKeyVerifySignature` 之处。

```
(lldb) dis
Reveal`___lldb_unnamed_symbol18939:
    ; ...
    0x1002369b0 :  bl     0x100c6a60c    ; symbol stub for: SecKeyVerifySignature
->  0x1002369b4 :  mov    x20, x0
    0x1002369b8 :  mov    x0, x23
    0x1002369bc :  bl     0x100c6b0b0    ; symbol stub for: objc_release
    0x1002369c0 :  mov    x0, x19
    0x1002369c4 :  bl     0x100c6b0b0    ; symbol stub for: objc_release
    0x1002369c8 : cmp    w20, #0x0
    0x1002369cc : cset   w0, ne
    0x1002369d0 : ldp    x29, x30, [sp, #0x30]
    0x1002369d4 : ldp    x20, x19, [sp, #0x20]
    0x1002369d8 : ldp    x22, x21, [sp, #0x10]
    0x1002369dc : ldp    x24, x23, [sp], #0x40
    0x1002369e0 : ret
```

于是我们再在地址 `0x1002369b0` 处设置断点，重启应用观察传入 `SecKeyVerifySignature` 的实参值。

### 检查传入实参

```
(lldb) br s -a 0x1002369b0
Breakpoint 2: where = Reveal`___lldb_unnamed_symbol18939 + 76, address = 0x00000001002369b0
(lldb) r
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 2.1
    frame #0: 0x00000001002369b0 Reveal`___lldb_unnamed_symbol18939 + 76
Reveal`___lldb_unnamed_symbol18939:
->  0x1002369b0 : bl     0x100c6a60c  ; symbol stub for: SecKeyVerifySignature
Target 0: (Reveal) stopped.
```

待中断触发，打印出传入实参值，结果如下。

- `x0`：2048 位 RSA 公钥。
- `x1`：RS256 签名算法。
- `x2`：JWT Header.Payload 部分。
- `x3`：JWT Signature 部分。

```
(lldb) register read x0 x1 x2 x3
      x0 = 0x0000600000c20900
      x1 = 0x00000001ff6d3030  @"algid:sign:RSA:message-PKCS1v15:SHA256"
      x2 = 0x0000600000c14ed0
      x3 = 0x0000600000c15c20

(lldb) expr -l objc -- @import CoreFoundation;
(lldb) expr -l objc -- @import Security;
(lldb) expr -l objc -O -- CFBridgingRelease(SecKeyCopyAttributes((SecKeyRef)$x0))
{
    esiz = 2048;
    priv = 0;
    ...
}
(lldb) expr -l objc -O -- [(NSData *)CFBridgingRelease(SecKeyCopyExternalRepresentation((SecKeyRef)$x0, NULL)) base64EncodedStringWithOptions:0]
MIIBCgKCAQEAnD9x77cVGbT+9+4odDK5+vqH4Z0JMJXPZ3eATw+aPPHQKj5GhHTJ2pUBUoazri0IIZWFYy8O3UfNF9awDmBc+4kfz1qBTpmYWGhKHKKDGGadIAGiUQxVu81lrqRqPbDUQFl7oryQIAoC7RosL2XlUM6pfNvUwMAlX6Eiuc0N/pba3cBbTdFNSPO+BG6hs7yVSH4q6DAHCMUxOzYE0YFS/f7XcY2bGWbWiKwvpqzNbuYX+JvAALBrS0/Ggr7FX/45c3aIiDlFrDEeFMlDwupjZqViUyU9GiPxa92j6VV1y2FJ+yepdKXVDdL7/zIft2TbskVilRBJ27sC2S6k2amdiwIDAQAB

(lldb) po (NSString *)$x1
algid:sign:RSA:message-PKCS1v15:SHA256

(lldb) expr -l objc -O -- [[NSString alloc] initWithData:(NSData *)$x2 encoding:4]
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE3NTgxODY4NjEuMDYwNjQ5OSwiZXhwIjoxNzU4MTg3MTYxLjA2MDY1LCJlbWFpbCI6InN2bm55NzdAZ21haWwuY29tIiwibGljZW5zZV9wYXRoIjoiXC9saWNlbnNlc1wvMjREQjBCM0ItNDhCQi0xNEE2LTkxQ0QtMDI4OTA4NDkyQ0UzIiwiY2xpZW50X2V4cGlyZXNfYXQiOjE3NTg0NDYzNjEuMDYwNjUsImxpY2Vuc2Vfc3R5bGUiOjAsIm9yZ2FuaXphdGlvbl9zdHlsZSI6MCwiaWRlbnRpZmllciI6IjRpR0R4bFBjaE1kMmsrK3lzOWVpTzE4eFJOYUdyV0xGWjRBeWRCaUJyOT0iLCJpc19zdWJzY3JpcHRpb25fdXNhYmxlIjpmYWxzZSwic3ViIjoiRDcxOEU1RDctOUE2OC0xNEY0LUNFRDEtNENFMkUyNEQwNTkwIiwib3JnYW5pemF0aW9uX25hbWUiOiJzdm5ueTc3QGdtYWlsLmNvbSIsImZpbmdlcnByaW50IjoiVTBoQk1qVTJJR1JwWjJWemREb2dNamxpTWpVek5XTmpNR00xWlRGbE56Y3pZV0V3TldGa01EUTFabUkzTURObVptSmpObVUwT1dVMlpXUXdZVFkyTVdVeFl6WmhaR1UzTkRNd056UmlNdz09IiwibGljZW5zZV9leHBpcnkiOjE3NTkzOTY0MDkuMDExOTU3MiwiaXNzIjoiUmV2ZWFsIGJ5IE1hbnRlbCJ9

(lldb) expr -l objc -O -- [(NSData *)$x3 base64EncodedStringWithOptions:0]
OPReNgGoxsJora0jvmLlq37rbOqNmQiHNtgFLwFZ0RBD5wiLz6xkXpPHWlk6Ot5Y8wisld_-F6muEwEeqPyPcGKWNuD_Ot4bnG8uYx4rUx-CeiW33saDDl-mMX1GCF50917wWJH00zYzfZtZMIZMwupsUutUswDm9S6DFwUyArvXNBagUc9Rvi6-X1yBm2BHBLvOkp8p726mMQphk8MJ1GcxiwbuAIcq3Wg2FxZronT234XG2gWOAzYkubKF4GbTP6kjlpt3jSGoEk0qNKwsa6mK7WNicIVARy4-6_FzGjPvzNx9O9qsHPy8LE-fad6YaG1JmikA-XQw9QsdITttjK
```

### 查看返回值

`ni` 单步执行，进入 `SecKeyVerifySignature` 后 `finish` 至调用返回地址 `0x1002369b4`，此时查看 `SecKeyVerifySignature` 的返回值。

```
->  0x1002369b0 : bl     0x100c6a60c  ; symbol stub for: SecKeyVerifySignature

(lldb) ni

* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1

    frame #0: 0x00000001a502e15c Security`SecKeyVerifySignature

Security`SecKeyVerifySignature:

->  0x1a502e15c :  pacibsp

(lldb) finish
```
