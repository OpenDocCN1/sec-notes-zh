
可以看到很长一串，那我们看看请求是什么样子的，能不能去发送假请求去获取远程代码

### 5. 远程代码请求

```
{"uuid": "03000200040005000006000700080009", "ip": "", "code": "UE4GET4CRP9BWDQY20Z1P73GCBYHKB0PQCTH0QCEXUKSSJCTVVUJQZWTM3RRXW0P506D1MD4XYIGSX86", "requests_times": 1, "num": "6156", "sign": "7e2ec0e60d857648a30ccedfec6d294aa1258f88d73b7ea5e0ae7615988354b3", "version": "1.1.0", "filepath": "./single_control_files/1.1.0/resources/utils/mjpeg_tool.py"}
```

在请求里面很明显的看到了`code`注册卡，那这就麻烦了，后台肯定校验了卡对不对，不对就不下发代码。但是我们有XX助手的云代码，先给他填上看看对不对，XX助手云代码解密如下

```
#! /usr/bin/env python

# -*- coding: utf-8 -*-

"""
Updated on 2023-10-22
"""

import os

import socket

import struct

from lk_logger import lk

class SafeSocket(object):

    """safe and exact recv & send"""

    def __init__(self, sock=None):

        if sock is None:

            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

        else:

            self.sock = sock

        self.buf = b""

    def __enter__(self):

        try:

            return self.sock.__enter__()

        except AttributeError:

            return self.sock

    def __exit__(self, exc_type, exc_val, exc_tb):

        try:

            return self.sock.__exit__(exc_type, exc_val, exc_tb)

        except AttributeError:

            self.sock.close()

    # PEP 3113 -- Removal of Tuple Parameter Unpacking

    # https://www.python.org/dev/peps/pep-3113/

    def connect(self, tuple_hp):

        host, port = tuple_hp

        self.sock.connect((host, port))

    def send(self, msg):

        totalsent = 0

        while totalsent  bytes:

        """ return without delimeter """

        while True:

            index = self.buf.find(delimeter)

            if index != -1:

                _return = self.buf[:index]

                self.buf = self.buf[index + len(delimeter):]

                return _return

            self._drain()

    def read_bytes(self, length: int) -> bytes:

        while length > len(self.buf):

            self._drain()

        _return, self.buf = self.buf[:length], self.buf[length:]

        return _return

    def write(self, data: bytes):

        return self.sock.sendall(data)

class MJpegcap(object):

    """重写airtest的mjpegcap，用已知的端口获取数据"""

    def __init__(self, ip='localhost', port=None):

        self.port = int(port)

        self.ip = ip

        self.sock = None

        self.buf = None

        self._is_running = False

        # self.ori_function = ori_function

    # @ready_method

    # def setup_stream_server(self):

    #     if self.port_forwarding:

    #         self.port, _ = self.instruct_helper.setup_proxy(9100)

    #         lk.log(self.port, _)  # 2023-04-21修改

    #     self.init_sock()

    #     reg_cleanup(self.teardown_stream)

    def init_sock(self):

        try:

            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

            # lk.log(self.port+40000)

            # if self.check_port_in_use(self.port+40000):

            #     lk.log('port_in_use', self.port + 40000)

            #     self.relase_port(self.port+40000)

            # self.sock.bind(('127.0.0.1', self.port+40000))

            # lk.log('self.sock.bind', self.port+40000)

            self.sock.connect((self.ip, self.port))

            # local_ip, self.local_port = self.sock.getsockname()

            # lk.log(f"Local IP: {local_ip}, Local Port: {self.local_port}")

            self.buf = SocketBuffer(self.sock)

            self.buf.write(b"GET / HTTP/1.0\r\nHost: localhost\r\n\r\n")

            self.buf.read_until(b'\r\n\r\n')

            self._is_running = True

            # lk.log("mjpegsock is ready")

        except ConnectionResetError:

            # 断开tidevice或是拔线，会导致这个异常，直接退出即可

            lk.log("mjpegsock connection error")

            raise

    def close_sock(self):

        try:

            # 设置SO_REUSEADDR选项

            self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

            # 设置 SO_LINGER 选项，超时时间为 0 秒

            self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_LINGER, struct.pack('ii', 1, 0))

            self.sock.close()

            # self.relase_port(self.port)

        except ConnectionResetError:

            lk.log("sock close error")

            raise

    @staticmethod  # 未使用类中变量的函数为静态函数，删除静态方法中的self，然后在这个方法的用@staticmethod声明一下。

    def check_port_in_use(port):

        """检测端口是否占用"""

        port = int(port)

        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

        sock.settimeout(0.05)

        return sock.connect_ex(('127.0.0.1', port)) == 0

    def get_frame_from_stream(self):

        if self._is_running is False:

            self.init_sock()

        try:

            while True:

                line = self.buf.read_until(b'\r\n')

                if line.startswith(b"Content-Length"):

                    length = int(line.decode('utf-8').split(": ")[1])

                    break

            while True:

                if self.buf.read_until(b'\r\n') == b'':

                    break

            imdata = self.buf.read_bytes(length)

            return imdata

        except IOError:

            # 如果暂停获取mjpegsock的数据一段时间，可能会导致它断开，这里将self.buf关闭并临时返回黑屏图像

            # 等待下一次需要获取屏幕时，再进行重连

            # LOGGING.debug("mjpegsock is closed")

            lk.log("mjpegsock is closed")

            self._is_running = False

            self.buf.close()

            # return self.get_blank_screen()

            return None

    def relase_port(self, _port):

        """释放指定端口"""

        # 查找指定端口对应的pid

        cmd_find = 'netstat -aon | findstr %s' % _port

        # lk.log(cmd_find)

        # 返回命令执行后的结果

        result = os.popen(cmd_find).read()

        # lk.log(result)

        if str(_port) and 'LISTENING' in result:

            # 获取端口对应的pid进程

            i = result.index('LISTENING')

            # LISTENING 与端口号之间相隔7个空格

            start = i + len('LISTENING') + 7

            end = result.index('\n')

            # 提取端口号

            pid = result[start:end]

            # 关闭被占用端口的pid

            cmd_kill = 'taskkill -f -pid %s' % pid

            lk.log(cmd_kill)

            os.popen(cmd_kill)

        else:

            # lk.log('port %s is available' % _port)

            pass

    # def get_frame(self):

    #     # 获得单张屏幕截图

    #     return self.get_frame_from_stream()

    # def snapshot(self, ensure_orientation=True, *args, **kwargs):

    #     """

    #     Take a screenshot and convert it into a cv2 image object

    #

    #     获取一张屏幕截图，并转化成cv2的图像对象

    #     !!! 注意，该方法拿到的截图可能不是队列中最新的，除非一直在消费队列中的图像，否则可能会是过往图像内容，请谨慎使用

    #

    #     Args:

    #         ensure_orientation: True or False whether to keep the orientation same as display

    #

    #     Returns: numpy.ndarray

    #

    #     """

    #     screen = self.get_frame_from_stream()

    #     try:

    #         screen = aircv.utils.string_2_img(screen)

    #     except Exception:

    #         # may be black/locked screen or other reason, print exc for debugging

    #         traceback.print_exc()

    #         return None

    #

    #     if ensure_orientation:

    #         if self.ori_function:

    #             display_info = self.ori_function()

    #             orientation = next(key for key, value in ROTATION_MODE.items() if value == display_info["orientation"])

    #             screen = aircv.rotate(screen, -orientation, clockwise=False)

    #

    #     return screen

    # def get_blank_screen(self):

    #     """

    #     生成一个黑屏图像，在连接失效时代替屏幕画面返回

    #     Returns:

    #

    #     """

    #     if self.ori_function:

    #         display_info = self.ori_function()

    #         width, height = display_info['width'], display_info['height']

    #         if display_info["orientation"] in [90, 270]:

    #             width, height = height, width

    #     else:

    #         width, height = 1080, 1920

    #     img = numpy.zeros((width, height, 3)).astype('uint8')
```
