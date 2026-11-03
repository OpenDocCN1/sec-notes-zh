
```
mov eax, 0x6499F8A9
ret
```

调试过后，可以知道`requestData_fromUrl`主要作用是：将字符串"Bananabot 5000"解码，传递给`InternetOpenA`函数。然后，将字符串"http://127.0.0.1:57328/2/summary"解码并传递给`InternetOpenUrlA`函数，最后调用`InternetReadFile`函数获取数据，在游览器访问"http://127.0.0.1:57328/2/summary"可以看到。这其实是`miner.exe`进程的http服务器，其定义可以在`config.json`文件中看到。随后回到`sub_62F43070`函数，该函数后面通过从`url`读取的数据作为密钥来解码shellcode，shellcode使用密钥"version": "6.20进行AES ECB解码，其实不用关心，因为程序没蹦。。  

调试到`call rdi`，使用`scylla`插件或者`savedata`命令从内存dump `0x4470`数据，保存为`bin1`，接下来就是分析`bin1`。

## `bin1`分析

将`bin1`放入`ida`，结合动态调试，可以知道第一个函数`resolve_function`为了解析函数地址，写入到`word_836`所在位置。也可以通过汇编代码`mov rax,gs:[60]`知道是在获得PEB地址，恶意软件技术获取函数地址的常用技术开头。

![](https://attach.52pojie.cn/forum/202405/06/204618yidawe4wcll3cii8.png)
QQ图片20240505003651.png (39.22 KB, 下载次数: 3)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3Nnw3YWUxNmRkYnwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:46 上传

![](https://attach.52pojie.cn/forum/202405/06/204259fbnkbfznbrk6oryj.png)
QQ图片20240504234712.png (59.14 KB, 下载次数: 2)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3MXw1NWUyZjA3N3wxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:42 上传

`bin1`的主要逻辑在`bin1_main`, 该函数逻辑：读取`C:\Program Files (x86)\Steam\config\config.vdf`打开的文件（虚拟机没有steam，伪造一下），然后从中获得`SentryFile`字段对应的文件的路径（SentryFile对应值是路径），并将`SentryFile`文件复制到`C:\depot\ssfn_steam`。最后，该函数使用`config.vdf`文件的前16个字节作为`key`使用`rc4`算法来解码额外的shellcode并跳转。

![](https://attach.52pojie.cn/forum/202405/06/204407gke1rh61g4z914rr.png)
QQ图片20240506010416.png (91.29 KB, 下载次数: 2)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3Mnw4NjZmYjYzY3wxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:44 上传

![](https://attach.52pojie.cn/forum/202405/06/204739dwp77cr00rqrv8za.png)
QQ图片20240505014042.png (72.8 KB, 下载次数: 3)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3N3xjNWQxYmZmZnwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:47 上传

使用同样的方法dump `bin2`，接着分析`bin2`。

## `bin2`分析

`bin2`与`bin1`差不多，第一步也是解析函数，然后这次针对的是`discard`平台，主要逻辑：在`C:\Users\invoker\AppData\Roaming\Discord\Local Storage\leveldb`中搜索`.ldb`文件（Microsoft Access 锁定信息文件），然后在每个`.ldb`文件的内容中搜索`dQw4w9WgXcQ`（youtube）（ Never Gonna Give You Up 经典名曲。。) 复制到`C:\depot`。最后从`Cookies`文件中读取16个字节（`SQLite format 3\0`）并将其用作密钥解码下一阶段的shellcode。

![](https://attach.52pojie.cn/forum/202405/06/211725y2ni0l72lbm37cxc.png)
QQ图片20240506010416.png (91.29 KB, 下载次数: 4)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3OXxmODkxNDk3ZXwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 21:17 上传

## `bin3`分析

`bin3`，和`bin2`一样，经典套娃。。这次针对的是`Sparrow`数字钱包文件（得下载），主要逻辑：在`C:\Users\three\AppData\Roaming\Sparrow\wallets`中搜索数据库文件，将文件复制到`C:\depot`，最后用17字节`recentWalletFiles`解码下一个阶段shellcode。

![](https://attach.52pojie.cn/forum/202405/06/204425jrrfj4k6kwcyyk4d.png)
QQ图片20240506014250.png (60.87 KB, 下载次数: 2)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3M3w5ZjM3NTdmMnwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:44 上传

## `bin4`分析

`bin4`将之前偷的东西发给服务器，主要逻辑：函数收集前面步骤中保存在`C:\depot`文件夹中的所有文件，并将它们组合成一个名为`output`的文件。并通过HTTP POST转发到`https://bighackies.flare-on.com/stolen`。然后，它读取对发送的HTTP请求的响应，对输出内容与响应接收到的字节数执行多次32CRC。如果所有比较都成功通过，则将执行解码阶段。这次解密用的`xor`，`xor`的`key`与`http_response_size`相关，我们可以直接推出`key`，不用执行http请求。因为解码后的`bin`前4个字节为`"the "`，即`0x20656874`，而加密的`bin`前4个字节为`0x32513E04`，两者异或为`0x12345670`，所以`key=0x12345670`，即`http_response_size=0x10`。

![](https://attach.52pojie.cn/forum/202405/06/204438rcxkqt6q7wxzncwn.png)
QQ图片20240506120524.png (52.73 KB, 下载次数: 2)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3NHw2ZmUxMTkwZXwxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:44 上传

## `bin5`分析

`bin5`最后的阶段，方式和之前一样，主要逻辑：从当前进程（即游戏）的内存中读取字节，比较当前地图名称的前4个字节是`"spcr"`，从地图的`spcr2.cfg`文件中读取数据，因为选完地图可能改变内存或配置文件，我又重新来了一遍（在选完地图后附加，都是泪），然后忽略`if`条件（没有影响数据变化）。后面会计算`flag`的`CRC32`并将结果与常量`0xA5561586`进行比较，最后得到`flag`。

![](https://attach.52pojie.cn/forum/202405/06/204453umdxuu14djz7cjor.png)
QQ图片20240506145334.png (62.54 KB, 下载次数: 2)
[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5NDA3NXxmN2E4OTM1N3wxNzkzNTcyMTg2fDE2ODA4NnwxOTIxMDUx&nothumb=yes)
2024-5-6 20:44 上传

Flag: computer_ass1sted_ctfing@flare-on.com

## 参考链接

[https://anti-debug.checkpoint.com/](https://anti-debug.checkpoint.com/)
[https://blog.maikxchd.com/a-shitty-flare-on-10-writeup-for-challenge-4-aimbot](https://blog.maikxchd.com/a-shitty-flare-on-10-writeup-for-challenge-4-aimbot)
