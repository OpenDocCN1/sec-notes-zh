
# Terminal Profile

## Terminal Profile Byte 23

`0x68`；`Alpha Identifier in REFRESH command`；`Proactive SIM: PROVIDE LOCAL INFORMATION (MEID)`；`Proactive SIM: PROVIDE LOCAL INFORMATION (NMR(UTRAN/E-UTRAN))`

```
.... ...0 = Proactive SIM: SET FRAMES: Not supported
.... ..0. = Proactive SIM: GET FRAMES STATUS: Not supported
.... .0.. = MMS notification download: Not supported
.... 1... = Alpha Identifier in REFRESH command: Supported
...0 .... = Geographical Location Reporting: Not supported
..1. .... = Proactive SIM: PROVIDE LOCAL INFORMATION (MEID): Supported
.1.. .... = Proactive SIM: PROVIDE LOCAL INFORMATION (NMR(UTRAN/E-UTRAN)): Supported
0... .... = USSD Data download and application mode: Not supported
```

## Terminal Profile Byte 24

`0x00`（Class "i"）

```
.... 0000 = Maximum number of frames supported: 0
0000 .... = RFU: 0x0
```

## Terminal Profile Byte 25

`0x51`；`Event: Browsing status`；`Event: Network Rejection for GERAN/UTRAN`；`Event: Network Rejection for E-UTRAN`

```
.... ...1 = Event: Browsing status: Supported
.... ..0. = Event: MMS Transfer status: Not supported
.... .0.. = Event: Frame Information changed: Not supported
.... 0... = Event: I-WLAN Access status: Not supported
...1 .... = Event: Network Rejection for GERAN/UTRAN: Supported
..0. .... = Event: HCI connectivity: Not supported
.1.. .... = Event: Network Rejection for E-UTRAN: Supported
0... .... = Multiple access technologies supported in Event Access Technology Change and PROVIDE LOCAL INFORMATION: Not supported
```

## Terminal Profile Byte 26

`0x01`；`Event: CSG Cell Selection`

```
.... ...1 = Event: CSG Cell Selection: Supported
.... ..0. = Event: Contactless state request: Not supported
0000 00.. = RFU: 0x00
```

## Terminal Profile Byte 27

`0x00`

```
0000 0000 = RFU: 0x00
```

## Terminal Profile Byte 28

`0x00`

```
.... ...0 = Alignment left: Not supported
.... ..0. = Alignment centre: Not supported
.... .0.. = Alignment right: Not supported
.... 0... = Font size normal: Not supported
...0 .... = Font size large: Not supported
..0. .... = Font size small: Not supported
00.. .... = RFU: 0x0
```

## Terminal Profile Byte 29

`0x00`

```
.... ...0 = Style normal: Not supported
.... ..0. = Style bold: Not supported
.... .0.. = Style italic: Not supported
.... 0... = Style underlined: Not supported
...0 .... = Style strikethrough: Not supported
..0. .... = Style text foreground colour: Not supported
.0.. .... = Style text background colour: Not supported
0... .... = RFU: 0x0
```

## Terminal Profile Byte 30

`0x08`；`"Steering of Roaming" REFRESH`

```
.... ...0 = I-WLAN bearer: Not supported
.... ..0. = Proactive SIM: PROVIDE LOCAL INFORMATION (WSID of the current I-WLAN connection): Not supported
.... .0.. = TERMINAL APPLICATIONS: Not supported
.... 1... = "Steering of Roaming" REFRESH: Supported
...0 .... = Proactive SIM: ACTIVATE: Not supported
..0. .... = Proactive SIM: Geographical Location Request: Not supported
.0.. .... = Proactive SIM: PROVIDE LOCAL INFORMATION (Broadcast Network Information): Not supported
0... .... = "Steering of Roaming for I-WLAN" REFRESH: Not supported
```

## Terminal Profile Byte 31

`0x02`；`CSG cell discovery`

```
.... ...0 = Proactive SIM: Contactless State Changed: Not supported
.... ..1. = CSG cell discovery: Supported
.... .0.. = Confirmation parameters supported for OPEN CHANNEL in Terminal Server Mode: Not supported
.... 0... = Communication Control for IMS: Not supported
...0 .... = CAT over the modem interface: Not supported
..0. .... = Event: Incoming IMS Data: Not supported
.0.. .... = Event: IMS Registration: Not supported
0... .... = Proactive SIM: Profile Container, Envelope Container, COMMAND CONTAINER and ENCAPSULATED SESSION CONTROL: Not supported
```

## Terminal Profile Byte 32

`0x80`；`Support of refresh enforcement policy`

```
.... ...0 = IMS bearer: Not supported
.... ..0. = Proactive SIM: PROVIDE LOCAL INFORMATION (H(e)NB IP address): Not supported
.... .0.. = Proactive SIM: PROVIDE LOCAL INFORMATION (H(e)NB surrounding macrocells): Not supported
.... 0... = Launch parameters supported for OPEN CHANNEL in Terminal Server Mode: Not supported
...0 .... = Direct communication channel supported for OPEN CHANNEL in Terminal Server Mode: Not supported
..0. .... = Proactive SIM: Security for Profile Container, Envelope Container, COMMAND CONTAINER and ENCAPSULATED SESSION CONTROL: Not supported
.0.. .... = CAT service list for eCAT client: Not supported
1... .... = Support of refresh enforcement policy: Supported
```

## Terminal Profile Byte 33

`0x07`；`Support of DNS server address request for OPEN CHANNEL related to packet data service bearer`；`Support of Network Access Name reuse indication for CLOSE CHANNEL related to packet data service bearer`；Ev

```
.... ...1 = Support of DNS server address request for OPEN CHANNEL related to packet data service bearer: Supported
.... ..1. = Support of Network Access Name reuse indication for CLOSE CHANNEL related to packet data service bearer: Supported
.... .1.. = Event: Poll Interval Negotiation: Supported
0000 0... = RFU: 0x00
```

# eSIM 关键参数更新过程

下面开始针对 eSIM 的关键参数进行更新的过程：进行了一系列的 `SMS-PP Download` 过程，通过 `SIM OTA` 过程进行远程放号。

![](https://attach.52pojie.cn/forum/202506/21/140146aqvrr4ph377qk4ak.png)

`image.png`（63.26 KB，下载次数：0）

[href="forum.php?mod=attachment&aid=Mjc4NDM4MXxmMWU0M2UwMHwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc4NDM4MXxmMWU0M2UwMHwxNzkzNTcwODcyfDE2ODA4NnwyMDQwNTEy&nothumb=yes)

2025-6-21 14:01 上传

## SIM APDU 数据与解码结果

### APDU 数据

把 `TX` 过程的 APDU 单独拷贝出来整合到一起，

```
80 C2 00 00 6BD1 69 02 02 83 81 06 07 91 28 01 08 90 99 99 0B 5A 40 0B 99 99 22 99 99 99 F9 7F F6 22 40 41 61 05 32 63 47 07 00 03 00 07 01 70 00 01 C1 15 02 01 25 25 B0 00 01 00 00 00 00 00 00 98 9C 60 FE 03 4B 63 C3 00 A4 08 0C 04 7F FF AF F1 00 D6 00 00 48 80 21 34 35 30 30 36 31 30 38 33 33 33 32 30 30 38 40 6C 74 65 2D 6C 67 75
```

放到 `SIM APDU` 解码器解码结果如下：

### 解码概要

```
1000 .... = Class Coding: ETSI TS 102.221 (0x8)
.... 00.. = Secure Messaging Indication: No SM used between terminal and card (0x0)
.... ..00 = Logical Channel number: 0
Instruction: ENVELOPE (0xc2)
Length of Expected Response Data: 107
BER-TLV Tag: GSM/3GPP/3GPP2 - SMS-PP Download (0xd1)
Card Application Toolkit ETSI TS 102.223
Device identity: 8381
Source Device ID: Network (0x83)
Destination Device ID: SIM / USIM / UICC (0x81)
Address: 91280108909999
.001 .... = TON: International Number (0x1)
.... 0001 = NPI: ISDN/telephony numbering plan (Recommendation ITU-Ts E.164 and E.163 (0x1)
Address String: (拢貌锟斤拷锟?
3GPP SMS TPDU: 400b999922999999f97ff62240416105326347070003000701700001c11502012525b000鈥?
```

### GSM SMS TPDU (GSM 03.40) SMS-DELIVER

```
0... .... = TP-RP: TP Reply Path parameter is not set in this SMS SUBMIT/DELIVER
.1.. .... = TP-UDHI: The beginning of the TP UD field contains a Header in addition to the short message
..0. .... = TP-SRI: A status report shall not be returned to the SME
.... 0... = TP-LP: The message has not been forwarded and is not a spawned message
.... .0.. = TP-MMS: More messages are waiting for the MS in this SC
.... ..00 = TP-MTI: SMS-DELIVER (0)
TP-Originating-Address - (99229999999)
Length: 11 address digits
1... .... = Extension: No extension
.001 .... = Type of number: International (1)
.... 1001 = Numbering plan: Private numbering plan (9)
TP-OA Digits: 99229999999
TP-PID: 127
01.. .... = Defines formatting for subsequent bits: 0x1
..11 1111 = Message type: (U)SIM Data download (63)
TP-DCS: 246
1111 .... = Coding Group Bits: Data coding/message class (15)
.... 0... = Reserved: 0
.... .1.. = Message coding: 8 bit data
.... ..10 = Message Class: Class 2 (U)SIM specific message (0x2)
```

### TP-User-Data

```
..................................这里省略一些无用内容
TP-User-Data-Length: (71) depends on Data-Coding-Scheme
TP-User-Data
User-Data Header
User Data Header Length: 7
IE: Concatenated short messages, 8-bit reference number (SMS Control)
```
