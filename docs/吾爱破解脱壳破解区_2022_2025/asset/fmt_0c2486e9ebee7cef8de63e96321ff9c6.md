
```
s=0x59C69FE620
m=0x59C69FE520
mh=[]
ms=[]
for i in range(128):
  mh.append(ida_bytes.get_byte(s+i))
for i in range(128):
  ms.append(ida_bytes.get_byte(m+i))
a=int.from_bytes(bytes(mh),"little")
b=int.from_bytes(bytes(ms),"little")
print(hex(pow(a,0x10001,b)))
```

![](https://attach.52pojie.cn/forum/202507/22/111224spbmfqlqu3fbbuje.png)
99.png (5.73 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc5MDQ3OXw4MWJkMDNlYXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQ3OXw4MWJkMDNlYXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:12 上传

![](https://attach.52pojie.cn/forum/202507/22/111226mxeyifsibbzaxi9g.png)
100.png (11.55 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc5MDQ4MHxmMjFjMTQyZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQ4MHxmMjFjMTQyZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:12 上传

得到结果,说明猜想正确.

![](https://attach.52pojie.cn/forum/202507/22/111228qv3t2edv3j2kee6u.png)
101.png (39.58 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDQ4MXw3MjAwZjQ1MnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQ4MXw3MjAwZjQ1MnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:12 上传

分块加密.根据输入输出可得,`1be0`为加密函数.

![](https://attach.52pojie.cn/forum/202507/22/151127e30s0ai3rrxm2ura.png)
103.png (59.4 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDY2NHwzM2ZkNTk0N3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDY2NHwzM2ZkNTk0N3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 15:11 上传

跟进,根据轮函数可判断为`chacha20`.由于是流加密,所以只要将明文异或即为密文,长度不变.加密后即写入文件.

![](https://attach.52pojie.cn/forum/202507/22/111323clb2ejnqqx7oniiy.png)
104.png (23.73 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDUwNHxjODYxMjhlMXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDUwNHxjODYxMjhlMXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:13 上传

写入加密数据到文件后,再次生成三个随机数,组成24字节数据,计算第一个随机数模24的值,作为长度传入`NtWriteFile`写入文件.后文对此有解释.

![](https://attach.52pojie.cn/forum/202507/22/151216zcc4tp2wfys4sik2.png)
105.png (21.58 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDY2NXwxMTc4ZDVmNHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDY2NXwxMTc4ZDVmNHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 15:12 上传

![](https://attach.52pojie.cn/forum/202507/22/111324aabf7kxzcft2t9ms.png)
106.png (28.78 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDUwNXw3NTlmNjRiZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDUwNXw3NTlmNjRiZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:13 上传

解密6字节魔术字符串.可以发现,在被加密文件的尾部有该字符串.

![](https://attach.52pojie.cn/forum/202507/22/111326v08pgnq8dpppdbdd.png)
107.png (20.54 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDUwNnxkNjgwNGYwZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDUwNnxkNjgwNGYwZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:13 上传

拷贝`v17`到一个缓冲区的头部.`v17`为前文获得的原文件大小.所以,解密的时候根据这个值计算大小,即可忽略在加密数据之后,文件信息之前加入的垃圾数据.垃圾数据使得文件大小得到使用,而且让加密后文件大小每次都不一样.

![](https://attach.52pojie.cn/forum/202507/22/151321lsco17nczo1ohlwz.png)
108.png (23.74 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc5MDY2OHw2NTgyNTc4NHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDY2OHw2NTgyNTc4NHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 15:13 上传

跟进`2740`,发现拷贝了之前模幂的结果.将结果拼接在文件大小的后面,再拼接上这个魔术字符串,将头部文件大小与`rsa`加密后的64字节密钥数据(加密前44字节)的第一个`qword`异或,写入文件末尾.

释放资源.

![](https://attach.52pojie.cn/forum/202507/22/151329mpmen1bmbyp6iezm.png)
111.png (4.73 KB, 下载次数: 0)
[href="forum.php?mod=attachment&aid=Mjc5MDY3MXw0ZjEwOTIyYnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDY3MXw0ZjEwOTIyYnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 15:13 上传

可以看到,加密表面上并没有什么漏洞,也使用了非对称加密.所以问题应该就出在`rsa`算法的使用上.提取`n`,尝试分解,成功.所以漏洞就在此.计算出私钥,就可以解密了.私钥计算代码如下:

```
import gmpy2

from Crypto.Util.number import long_to_bytes

p = 33181439397036208526873371281094347304315910469546899478574685327220453068623

q = 33181439397036208526873371281094347304315910469546899478574685327220453068539

e = 65537

n = q*p

d = gmpy2.invert(e, (p - 1) * (q - 1))

print(list(long_to_bytes(d)))

#[18, 194, 136, 240, 17, 86, 207, 217, 37, 12, 248, 163, 28, 7, 228, 151, 159, 228, 128, 54, 21, 110, 162, 164, 43, 104, 235, 118, 63, 44, 99, 32, 46, 218, 156, 29, 178, 135, 155, 220, 180, 229, 243, 76, 101, 127, 82, 45, 176, 15, 201, 195, 248, 197, 24, 187, 226, 22, 23, 182, 202, 161, 242, 29]
```

求出私钥(`d`),提取模数(`n`),就可以解密了.下面是一个c语言写的自带扫描的解密器源码(带注释)和编译好的exe文件:

编写解密脚本后,解密`samples`中所有文件,运行`set_flag_interpreter.exe`,即可获得`flag`.

![](https://attach.52pojie.cn/forum/202507/22/111246frf20e20ygccfcjg.png)
113.png (27.65 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDQ4NHxmNjYyN2I5YnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQ4NHxmNjYyN2I5YnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:12 上传

文件未解密前

![](https://attach.52pojie.cn/forum/202507/22/111248k0tumy4uu3juxrjj.png)
114.png (13.34 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=Mjc5MDQ4NXwyYzAzYmNhNHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQ4NXwyYzAzYmNhNHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes)
2025-7-22 11:12 上传

文件解密后
