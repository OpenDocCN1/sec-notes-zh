
`Image-35.png` (42.94 KB, 下载次数: 1)

[`Image-35.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwOHwyZjYxNjZjMnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

### `svchost`进程创建

接着，再次通过天堂之门实现对X64系统的调用，最终实现对`svchost`的恶意代码注入。

在此过程中，恶意代码切换X64环境分别调用了以下函数：

- `NtMapViewOfSection 0x0003`；
- `NtWriteVirtualMemory 0x003a`；
- `NtProtectVirtualMemory 0x0050`；
- `NtResumeThread 0x0052`。

![](https://attach.52pojie.cn/forum/202503/21/093405rb1unooedopup7o2.png)

`Image-36.png` (201.2 KB, 下载次数: 1)

[`Image-36.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQwOXxhZWIxYzY5ZnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

syscall injet

## 第三阶段

对解密出的内存PE文件分析发现，它是Lumma Stealer的窃密程序。该文件能够获取剪贴板信息、Steam token以及浏览器账号密码数据库等信息。

![](https://attach.52pojie.cn/forum/202503/21/093407j355b60pi45b5jbr.png)

`Image-37.png` (33.92 KB, 下载次数: 2)

[`Image-37.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxMHwwN2IyNTBmNnwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

剪贴板信息

该窃密程序对Steam token以及浏览器账号密码等信息的获取方式与[//www.52pojie.cn/thread-1974684-1-1.html](https://www.52pojie.cn/thread-1974684-1-1.html)中的窃密程序相同，具体细节可查看往期分析报告。

最后，将数据进行`7z`压缩，并上传至域名`sterpickced.digital`，完成信息窃密。

## 二、附录

**C&C：**

![](https://attach.52pojie.cn/forum/202503/21/093409z6q9e9efk6z0lcxk.png)

`Image-38.png` (8.04 KB, 下载次数: 1)

[`Image-38.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxMXwxOGFiYTZlN3wxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传

**HASH：**

![](https://attach.52pojie.cn/forum/202503/21/093411i7jqr7hplefpe78f.png)

`Image-39.png` (26.44 KB, 下载次数: 0)

[`Image-39.png`](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc2NDQxMnw5ZDFkYjNmNHwxNzkzNTc2NzIzfDE2ODA4NnwyMDE3MTQx&nothumb=yes)

2025-3-21 09:34 上传
