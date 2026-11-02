
之后，将病毒的主要文件复制到`C:\Users\Admin\AppData\Roaming\aah`路径下，并创建进程`C:\Users\Admin\AppData\Roaming\aah\TiVoDiag.exe`（前提是该进程不存在）。

![](https://attach.52pojie.cn/forum/202503/21/093332sd4wwgd2avghot4z.png)

Image-17.png (50.95 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDM5NHxkMmQxNDAyYnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDM5NHxkMmQxNDAyYnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

复制恶意文件

然后，再次获取关键API，其中包含网络相关的API。

![](https://attach.52pojie.cn/forum/202503/21/093335uizcsqqspsm5q975.png)

Image-18.png (81.28 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDM5NXw2ZjcyNjUzOXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDM5NXw2ZjcyNjUzOXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

网络相关API

接着，通过进程Token查询进程的完整性级别。

![](https://attach.52pojie.cn/forum/202503/21/093337o3std53ptz4vvnzt.png)

Image-19.png (27.15 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDM5NnxhNzBjNTFmZnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDM5NnxhNzBjNTFmZnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

查询进程的完整性级别

随后，通过com组件{00021401-0000-0000-C000-000000000046}创建快捷方式，进一步实现持久化操作。

![](https://attach.52pojie.cn/forum/202503/21/093339hg57x0znm3nge0mk.png)

Image-20.png (47.8 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDM5N3xiM2UxOTkxNnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDM5N3xiM2UxOTkxNnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

com组件快捷方式

之后，恶意代码再次利用进程镂空技术，将其自身注入到`shdocvw.dll`中，并创建`more.com`挂起进程（`more.com`为查看文本命令，类似于Linux系统中的`cat`命令）。

![](https://attach.52pojie.cn/forum/202503/21/093341htaymdy0k83c0jnv.png)

Image-21.png (50.14 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDM5OHwyOGQ2Nzk1ZXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDM5OHwyOGQ2Nzk1ZXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

![](https://attach.52pojie.cn/forum/202503/21/093343pqdzq2is6mq4bwbs.png)

Image-22.png (45.67 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDM5OXxmMDRhZThlM3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDM5OXxmMDRhZThlM3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

进程镂空创建挂起进程

接着，通过天堂之门对挂起的`more.com`进程进行代码注入。

![](https://attach.52pojie.cn/forum/202503/21/093345kjo4bbq42tz5owt5.png)

Image-23.png (70.7 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQwMHxmZmZhZWM3NXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwMHxmZmZhZWM3NXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

天堂之门

对X64代码进行分析发现，该恶意代码主要通过syscall系统调用实现API调用。

![](https://attach.52pojie.cn/forum/202503/21/093348r8oaa8v00buaa5ag.png)

Image-24.png (90.24 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDQwMXxjODdhY2M2Y3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwMXxjODdhY2M2Y3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

syscall

通过分析syscall code可知，该恶意代码主要来源于参数`[ebp+8]`。经分析，其主要利用的syscall code如下。

![](https://attach.52pojie.cn/forum/202503/21/093353lolg4gssgg4s7og6.png)

Image-25.png (74.21 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc2NDQwM3w1ZjM4OTM0NnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwM3w1ZjM4OTM0NnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

syscall code

之后，通过天堂之门进行系统调用`NtCreateSection`与`NtMapViewOfSection`，最终实现恶意代码对`more.com`进程的内存映射。

![](https://attach.52pojie.cn/forum/202503/21/093354akqzwtmqx00i722z.png)

Image-26.png (38.91 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQwNHwyOWM5MTc4N3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwNHwyOWM5MTc4N3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

恶意代码映射

随后，将其最终shellcode进行加密，并写入到临时文件夹。

![](https://attach.52pojie.cn/forum/202503/21/093352xmg9xfg69qh97hog.png)

Image-27.png (56.09 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQwMnwyYzE3ZDhhMXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwMnwyYzE3ZDhhMXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

写入本地临时文件夹、恶意代码

之后，再次通过天堂之门进行系统调用`NtSetContextThread`、`ZwSetThreadContext`、`ResumeThread`，以恢复`more.com`的进程挂起。

![](https://attach.52pojie.cn/forum/202503/21/093356um1i1i4ccdw2i4qo.png)

Image-28.png (36.24 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQwNXxlZTUzMzQ1NHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwNXxlZTUzMzQ1NHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

恢复挂起进程more.com

## 第二阶段

恶意代码通过获取大量API的方式进行初始化，接着解密第一阶段生成的临时恶意代码文件，随后将解密后的恶意代码注入`svchost.exe`，以完成恶意代码的利用。

![](https://attach.52pojie.cn/forum/202503/21/093413va72zok6of5a2s2q.png)

Image-29.png (43.78 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQxM3xiYmE3ODBhZHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxM3xiYmE3ODBhZHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

运行流程

首先，恶意代码通过获取大量API的方式，完成其运行环境的初始化。

![](https://attach.52pojie.cn/forum/202503/21/093415d4av6kontvg461zt.png)

Image-30.png (120.33 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQxNHw1ZWJhOGQzMHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxNHw1ZWJhOGQzMHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

获取API

接着，将计算机名字生成的密钥作为解密密钥，对临时文件进行解密。

![](https://attach.52pojie.cn/forum/202503/21/093417tracjdore6tpcdc3.png)

Image-31.png (79.38 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQxNXxmZGUwYTA4ZXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxNXxmZGUwYTA4ZXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

解密临时文件

解密出来的恶意代码包含配置信息和用于第三阶段的内存PE文件，该文件属于Lumma Stealer。

![](https://attach.52pojie.cn/forum/202503/21/093419wcgie60ilw5pzeee.png)

Image-32.png (45.55 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQxNnw5MDE2NmVhZnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxNnw5MDE2NmVhZnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

解密出配置信息

![](https://attach.52pojie.cn/forum/202503/21/093359s1dduwwerxuxxu47.png)

Image-33.png (108.58 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQwNnw1MDA2Mzg0OXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwNnw1MDA2Mzg0OXwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:33 上传

解密出内存PE

之后，调用天堂之门完成对svchost进程的注入。此次注入所使用的syscall系统调用代码与第一阶段不同，其syscall code已更改为`[rbp+0C]`。

![](https://attach.52pojie.cn/forum/202503/21/093401gi5t8gtb7m7559wi.png)

Image-34.png (64.81 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=Mjc2NDQwN3w4ZmI1MGIxMHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwN3w4ZmI1MGIxMHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

天堂之门

恶意代码通过进程镂空`rpcrt4.dll`的方式创建`svchost.exe`进程。

![](https://attach.52pojie.cn/forum/202503/21/093403v1o31bvt3cn6sobx.png)
