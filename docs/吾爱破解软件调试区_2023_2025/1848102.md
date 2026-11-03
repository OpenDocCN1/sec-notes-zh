
# [调试逆向] 外星人笔记本键盘USB协议逆向

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1848102
- 抓取时间: 2026-11-01T21:55:36

## 前言

我朋友一台 dell g16 购买时直接安装了linux系统，但是linux上没有官方的键盘控制中心，所以无法控制键盘灯光，于是我就想着能不能逆向一下键盘的协议，然后自己写一个控制键盘灯光的程序。我自己的外星人笔记本是m16，所以我就先从m16开始逆向。

![](https://attach.52pojie.cn/forum/202310/24/145009i5i0kik1xkrki5z6.jpg)

m16.jpg (333.05 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTM1MHxjYzAyNjgwMHwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM1MHxjYzAyNjgwMHwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:50 上传

## USB协议分析

通过 chatgpt 得知，AlienFX设备通常通过USB接口连接到计算机。键盘的灯光控制是通过HID (人机接口设备) 协议进行的。当你使用AlienFX软件时，这些程序会发送特定的命令到键盘，告诉它如何设置灯光效果。

现在wireshark已经支持HID协议的解析，所以我们可以直接使用wireshark来分析USB协议。在安装wireshark是需要勾选安装USBPcap

![](https://attach.52pojie.cn/forum/202310/24/145211iiddccobsqdocepz.jpg)

install_wireshark.jpg (117.48 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY1MTM1Nnw5MjNhZDdhYXwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM1Nnw5MjNhZDdhYXwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:52 上传

打开wireshark，选择USBPcap1

![](https://attach.52pojie.cn/forum/202310/24/145331takm0a86x3g985bl.jpg)

wireshark_device.jpg (223.35 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTM1OHxhMWY2NDE4MHwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM1OHxhMWY2NDE4MHwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:53 上传

设置要捕获的usb设备，然后点击start

![](https://attach.52pojie.cn/forum/202310/24/145239wnsdnywz5yn4dpyd.jpg)

wireshark_device_filter.jpg (144.18 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTM1N3xjMzgzYTFlMnwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM1N3xjMzgzYTFlMnwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:52 上传

打开 alienware command center，设置键盘灯光，在灯光效果除设置颜色为红色，然后点击应用。

![](https://attach.52pojie.cn/forum/202310/24/145350r2a8h6wnzpp88dpv.jpg)

awcc.jpg (235.58 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTM1OXxjMGYxMTRjMXwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM1OXxjMGYxMTRjMXwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:53 上传

然后我们就可以在wireshark中看到usb协议的数据包了， 我们可以看到有两个数据包，一个是发送数据包，一个是接收数据包。可以通过设置过滤器来过滤掉接收数据包，只看发送数据包，过滤设置为 `usb.src == "host"`

![](https://attach.52pojie.cn/forum/202310/24/145405r9f6zc89k9lv9oz2.jpg)

wireshark_data.jpg (528.78 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjY1MTM2MHxkNDAzMTNlZHwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM2MHxkNDAzMTNlZHwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:54 上传

发现仅仅是改了一个按键的颜色，就发送了很多数据包，而且每个数据包的长度都不一样，这是因为每个数据包都是一个命令，而且每个命令的长度都不一样，所以我们需要找到每个命令的格式，然后才能解析出每个命令的含义。

### 验证数据包

这个时候我们需要写一个测试程序，来分析哪一个包让键盘改变了颜色，然后再分析这个包的格式。这里我们使用python将数据重发到usb设备，然后观察键盘的变化。

```
import logging
import time

import usb
from usb import USBError

class AlienwareUSBDriver:
    VENDOR_ID = 0xd62
    PRODUCT_ID = 0xc2b0

    SEND_BM_REQUEST_TYPE = 0x21
    SEND_B_REQUEST = 0x09
    SEND_W_VALUE = 0x3cc
    SEND_W_INDEX = 0x0

    PACKET_LENGTH = 63

    def __init__(self):
        self._control_taken = False
        self._device = None

    def acquire(self):
        """ Acquire control of the USB controller."""
        if self._control_taken:
            return

        self._device = usb.core.find(idVendor=AlienwareUSBDriver.VENDOR_ID, idProduct=AlienwareUSBDriver.PRODUCT_ID)

        if self._device is None:
            logging.error("ERROR: No AlienFX USB controller found; tried VID {}, PID {}"
                          .format(AlienwareUSBDriver.VENDOR_ID, AlienwareUSBDriver.PRODUCT_ID))

        try:
            self._device.set_configuration()
        except USBError as exc:
            logging.error("Cant set configuration. Error : {}".format(exc.strerror))

        try:
            usb.util.claim_interface(self._device, 0)
        except USBError as exc:
            logging.error("Cant claim interface. Error : {}".format(exc.strerror))

        self._control_taken = True
        logging.debug("USB device acquired, VID={}, PID={}".format(hex(AlienwareUSBDriver.VENDOR_ID),
                                                                   hex(AlienwareUSBDriver.PRODUCT_ID)))

    def release(self):
        if not self._control_taken:
            return

        try:
            usb.util.release_interface(self._device, 0)
        except USBError as exc:
            logging.error("Cant release interface. Error : {}".format(exc.strerror))

        try:
            self._device.attach_kernel_driver(0)
        except USBError as exc:
            logging.error("Cant re-attach. Error : {}".format(exc.strerror))

        self._control_taken = False
        logging.debug("USB device released, VID={}, PID={}".format(hex(AlienwareUSBDriver.VENDOR_ID),
                                                                   hex(AlienwareUSBDriver.PRODUCT_ID)))

    def write_packet(self, pkt):
        if not self._control_taken:
            return

        try:
            num_bytes_sent = self._device.ctrl_transfer(
                self.SEND_BM_REQUEST_TYPE, self.SEND_B_REQUEST,
                self.SEND_W_VALUE, self.SEND_W_INDEX,
                pkt, 0)

            logging.debug("wrote: {}, {} bytes".format(pkt, len(pkt)))
            if len(pkt) != num_bytes_sent:
                logging.error("writePacket: intended to write {} of {} bytes but wrote {} bytes"
                              .format(pkt, len(pkt), num_bytes_sent))

            return num_bytes_sent
        except USBError as exc:
            logging.error("writePacket: {}".format(exc))
```

其中设备信息可以在wireshark中查看

![](https://attach.52pojie.cn/forum/202310/24/145457z95zcdqc9bjsw59q.jpg)

device_info.jpg (489.46 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjY1MTM2MXw2MDcxYTI0ZnwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM2MXw2MDcxYTI0ZnwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:54 上传

`VENDOR_ID = 0xd62`

`PRODUCT_ID = 0xc2b0`

在使用 `device.ctrl_transfer` 发送数据时需要指定 `bmRequestType`, `bRequest`, `wValue`, `wIndex`，这些信息也可以在wireshark中查看

![](https://attach.52pojie.cn/forum/202310/24/145621my4r81aa8w1nn980.jpg)

usb_send_info.jpg (244.08 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjY1MTM2MnwwMmJiNjViMnwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM2MnwwMmJiNjViMnwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 14:56 上传

```
OUT_BM_REQUEST_TYPE = 0x21
OUT_B_REQUEST = 0x09
OUT_W_VALUE = 0x3cc
OUT_W_INDEX = 0x0
```

尝试将wireshark中的数据包发送到键盘，通过测试发现，其中一条数据包发送后，Q键的灯光才会改变

```
if __name__ == '__main__':
    device = AlienwareUSBDriver()
    device.acquire()
    data = bytes.fromhex('cc8c020073072f46121278b56519a6f9661799e568127ab7691aaaff6a2aaaff6c2aaaff6e137fbf7019a6f9711aaaff8608334b8708334b8808334b2bfc0000')
    device.write_packet(data)
```

### 数据包格式分析

通过分析得知，每次改变颜色，awcc 会通过`CC 8C 02 00` 命令把所有按键的颜色发送一遍 ，经过多次测试，发现每个按键的颜色都是由三个字节表示，分别是 R G B，所以我们可以通过改变这三个字节来改变按键的颜色。包格式如下:

![](https://attach.52pojie.cn/forum/202310/24/150010jjp3twplulj0t8zp.jpg)

command_table.jpg (131.04 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY1MTM2M3wwMzZmNWFiZXwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MTM2M3wwMzZmNWFiZXwxNzkzNTcwMTM0fDE2ODA4NnwxODQ4MTAy&nothumb=yes)

2023-10-24 15:00 上传

经过多次尝试后，将整个键盘的对应序号，得到如下表格

Key | 
Code | 
Key | 
Code | 
Key | 
Code | 
Key | 
Code | 
esc
