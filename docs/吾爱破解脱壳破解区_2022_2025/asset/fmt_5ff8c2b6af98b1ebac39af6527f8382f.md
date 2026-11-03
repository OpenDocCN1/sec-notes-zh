
# [CTF] 2025-6月Solar应急响应月赛WP！

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2047518
- 抓取时间: 2026-11-01T22:06:50

## 6月月赛WP

### RSA_LOCK

题目文件结构：

![](https://attach.52pojie.cn/forum/202507/22/110904q6p67plq3q6opkl0.png)
`7.png` (1.37 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDM5NnxjYTY3Y2YxN3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110914b8ogffojs4f64dza.png)
`8.png` (9.64 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDM5N3w2NmE2ZWUyNHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110916shal2hae6ihgill6.png)
`9.png` (49.43 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDM5OHw2NDZmMTBlMHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

ida 打开 `set_202506_chal_main_final.exe`。

![](https://attach.52pojie.cn/forum/202507/22/110919c4lfgpkgdgrrkcfp.png)
`10.png` (99.66 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDM5OXwxMGIyYWRlNXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

没有发现导入表，说明导入方式是动态获取地址。

![](https://attach.52pojie.cn/forum/202507/22/110922wds8d1s2m6dpaofo.png)
`11.png` (67.6 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwMHwxNGU1MzI3Y3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

跟进 `140001150` 函数。

![](https://attach.52pojie.cn/forum/202507/22/110924q2j6l148zr8zownj.png)
`12.png` (14.54 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwMXw0OWUzMDdjZnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

再跟进 `1400010f0` 函数，可以发现读取了 `peb` 中的 `ldr` 中的第一项。

由此可得，`140001150` 从 `ntdll.dll` 模块获取了一些东西。

![](https://attach.52pojie.cn/forum/202507/22/110927a8rg56qdgbk8rrlk.png)
`13.png` (40.97 KB, 下载次数: 2) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwMnw0NjdhOTA5Y3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

根据魔术数字可得，`140001000` 为 `crc64`，输入字符串，返回一个 qword。

直接根据 pe 结构一个个去找程序获取的东西是很费时间的。所以开始动调，目标是判断 `140001150` 做了什么。

![](https://attach.52pojie.cn/forum/202507/22/110929dun1zz8ndd9tf7ta.png)
`14.png` (21.52 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwM3xiZmY4ZmMzYXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

可以看到该函数只有一个输入，为 qword。

![](https://attach.52pojie.cn/forum/202507/22/110932odr7r7ywrryzy4o4.png)
`15.png` (47.26 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwNHw0NTBlMjM3NHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110935ly570xlvloxylhpq.png)
`16.png` (4.22 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwNXw5NGI3ZjkxY3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110937yogjolr1rjkx1otq.png)
`17.png` (3.93 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwNnwwNmYxZmNkOHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

随着输入的 qword 的变化，返回不同的值，都是函数的地址，都为 Nt 函数。结合前面对函数的操作是获取 `ntdll` 中的内容和字符串 `crc64`，可以合理地判断，输入的是 `crc64` 值，输出的是 `ntdll` 中对应函数名的函数地址。

![](https://attach.52pojie.cn/forum/202507/22/110940sjhno57hfdjlowoe.png)
`18.png` (50.52 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwN3w3YTBkMDFiMHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110943xdsz5pki3z7ubdbc.png)
`19.png` (29.72 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwOHw0NDZjMTMxN3wxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

获取了一些 nt 函数。

![](https://attach.52pojie.cn/forum/202507/22/110945iv7fxtpazdvwnv9p.png)
`20.png` (23.66 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQwOXw3MzNhMGZhYXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110948yaiz4iksq4avsq1j.png)
`21.png` (55.22 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQxMHxhNzUzMzQyNHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

观察区段表，发现可疑区段，看到他的名字和数据分布，此时应该把这些函数的获取、这段数据和加载 shellcode 联系起来。

再次看向主函数，可以重命名一些函数。

![](https://attach.52pojie.cn/forum/202507/22/110950x1z0mkrncmecgmri.png)
`22.png` (27.55 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQxMXw0NGVkZmRiZnwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

![](https://attach.52pojie.cn/forum/202507/22/110953pw5jqhfpdij765ie.png)
`23.png` (27.16 KB, 下载次数: 0) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQxMnw2NDFjM2JiNXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

注意到 `memcpy()` 函数，发现疑似读取区段内容的代码。

![](https://attach.52pojie.cn/forum/202507/22/110955xxezewe6sxytrz81.png)
`24.png` (1.84 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQxM3w0YjZmZmFkMXwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

找到 `v7` 和 `v2` 的定义。

![](https://attach.52pojie.cn/forum/202507/22/110958pl5o5bd1gt115bly.png)
`25.png` (23.67 KB, 下载次数: 1) [附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5MDQxNHw4OWEyYzhjZHwxNzkzNTcwODA4fDE2ODA4NnwyMDQ3NTE4&nothumb=yes) 2025-7-22 11:09 上传

根据区段表，发现读取的为 `.c0de` 中的数据，拷贝了 0x4c00 字节。

![](https://attach.52pojie.cn/forum/202507/22/111000kivogd9gmtzglbl7.png)
`26.png` (3.3 KB, 下载次数: 1)
