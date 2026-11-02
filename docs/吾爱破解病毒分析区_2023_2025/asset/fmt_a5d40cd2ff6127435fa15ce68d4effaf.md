
# [PC样本分析] Go语言低门槛背后的风险，初级勒索攻击或致解密无门

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1990340
- 抓取时间: 2026-11-01T23:46:22

Go 语言凭借其高效的并发处理能力和强大的标准库支持等特点，在开发效率和性能上具有一定的优势，备受软件开发者青睐。然而，这种优势也吸引了部分初学者以及勒索组织的目光，他们利用 Go 语言编写勒索病毒并进行传播，催生了各种基于 Go 语言的勒索实验样本和勒索组织样本。

近期，火绒工程师在日常关注安全动态时发现一个基于 Go 语言的勒索样本。分析发现，该样本会利用 AES 进行数据加密，并将加密秘钥发送至 `youtube.com`，导致被加密文件完全无法恢复。推测该样本属于基于 Go 语言勒索样本中的实验样本。随后对基于 Go 语言的勒索样本进行调查分析发现，实验样本在该类勒索样本中较为常见，而勒索组织样本除了勒索步骤比较完善，使用的技术手段与实验样本仍有一定相似。火绒安全产品可对上述勒索木马进行拦截查杀，请广大用户及时更新病毒库以提高防御能力。

![查杀图](https://attach.52pojie.cn/forum/202412/12/094719rh1vmu4m1hu4v44v.png)

`Image-0.png` (30.56 KB, 下载次数: 0) · 2024-12-12 09:47 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA3OHxhY2ZhNDg1OHwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

勒索病毒通常是黑客攻击链的最后一个环节，属于直接可以获利的病毒。在前期攻击中，黑客已将目标环境改造成可以执行任意程序而不被检测的状态。因此，基于 Go 语言的勒索病毒无需采用免杀等特殊技术手段，便能直接进行加密和上传操作。本文将主要对勒索实验样本以及勒索组织样本分别进行简要分析。基于 Go 语言的勒索样本进行勒索的步骤概括流程图如下：

![基于 Go 语言的勒索样本进行勒索的步骤概括](https://attach.52pojie.cn/forum/202412/12/094738eqtrmfweppttpetj.png)

`Image-1.png` (52.34 KB, 下载次数: 0) · 2024-12-12 09:47 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA3OXxkMDUwZGQ3OXwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

## 一、样本分析

基于 Go 语言的勒索样本经常利用到的库函数按照使用先后顺序排列，有以下几种：

- `crypto/rand.Read`：获取随机数
- `path/filepath.Walk`：遍历文件
- `crypto/aes.NewCipher`：初始化 AES 加密器
- `os.WriteFile`：写勒索信或者将加密数据写入原文件
- `encoding/json.Marshal`：转化为 JSON 字符串
- `net/http.Get/Post`：接收或发送数据
- `os/exec.Run`：打开勒索信

### 勒索实验样本

一些初学者或第一次开发勒索病毒的作者可能会制作出实验样本，这类样本的特点为勒索步骤不完整，例如缺少勒索信、勒索信无意义，或在对称加密过程中未能上传必要的数据等，最终导致受害者的数据无法被恢复，除非能够在内存中及时捕获密钥或在用户界面中获取相应密钥。而对于采用 RSA 非对称加密的样本，只有在私钥被保存至本地的情况下，才有可能实现数据恢复。

此次发现的勒索样本（下称 A 样本）即为实验样本，由于该样本最终将秘钥和用户标识符等数据发送至 `youtube.com`，使得被加密文件无法恢复。此外，本文还将对同类型样本 B 和 C 进行简要阐述，展示实验样本的大致步骤和勒索信的无意义。

#### A 样本

A 样本调用 `crypto/rand.Read` 函数生成 1 字节的随机数，并通过计算从 a-z、A-Z 和 0-9 的字符集中选取一个字符，循环此操作九次，生成一个长度为 9 的 `UserId` 字符串，用于识别用户。

![获取随机 UserId 字符串](https://attach.52pojie.cn/forum/202412/12/094753e292pp6ae2zmme3z.png)

`Image-2.png` (270.24 KB, 下载次数: 0) · 2024-12-12 09:47 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4MHxkOTUyOGZhY3wxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

随后，使用一个 32 字节的随机密钥和 AES 加密算法，对 `C:\Users` 文件夹下的所有文件进行加密。加密完成后，创建以 `[源文件名].mtxv21` 命名的加密文件，并删除源文件。

![加密 Users 目录](https://attach.52pojie.cn/forum/202412/12/094809xd5s6tn5e5udeett.png)

`Image-3.png` (69.89 KB, 下载次数: 0) · 2024-12-12 09:48 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4MXxkMzc3ODc2OHwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

还原后的加密算法如下。

![AES 加密算法](https://attach.52pojie.cn/forum/202412/12/094820xjf8sn8uo0junijz.png)

`Image-4.png` (131.6 KB, 下载次数: 0) · 2024-12-12 09:48 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4MnwyMDA0NjM0MHwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

接着，将 `UserId`、加密目录以及 32 位加密秘钥通过 JSON 数据发送至 `youtube.com`。这一操作使得勒索病毒的作者无法获取加密密钥，因此他们无法解密被加密的文件。即使受害者支付赎金也无法成功恢复数据。

![发送 JSON 数据至 youtube.com](https://attach.52pojie.cn/forum/202412/12/094832zee6jj0gje6jrr6u.png)

`Image-5.png` (34.99 KB, 下载次数: 0) · 2024-12-12 09:48 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4M3w5NmZhNmZlOHwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

该 JSON 数据的内容格式与 Discord Webhook 请求体完全一致。如果将目标 URL 替换为 Discord Webhook 链接，那么信息将直接被链接指向的频道接收。在后续处理中，可以通过 ID 分辨用户，并利用加密目录和加密秘钥对加密文件进行恢复。

![JSON 数据](https://attach.52pojie.cn/forum/202412/12/094843g7k2wmw99pl9wfwb.png)

`Image-6.png` (155.27 KB, 下载次数: 0) · 2024-12-12 09:48 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4NHw5OGY3M2Q2ZXwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

其中的部分 JSON 数据键说明如下：

- `avatar_url`：Webhook 消息的头像 URL
- `embeds`：嵌入对象
- `fields`：额外数据数组

在测试过程中存在桌面变黑的现象。分析发现，在 Win10 系统中，桌面背景图通常保存在 `C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Themes\TranscodedWallpaper` 路径中，该路径文件被加密则会导致桌面变黑的现象发生。

最后，该样本会打开勒索信 `README-NOW.txt`，其中显示的 Your ID 即是 `UserId`。

![勒索信](https://attach.52pojie.cn/forum/202412/12/094855a37mez6zjb7bpf57.png)

`Image-7.png` (168.5 KB, 下载次数: 0) · 2024-12-12 09:48 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4OHw5ZGY0MGRlZHwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

#### B 样本

B 样本首先获取随机 32 字符秘钥和 16 字符初始化向量。随后利用 AES 算法的 CFB 加密模式对文件进行加密，并且在该过程中并未保存秘钥。加密完成后，将当前文件移动至临时目录中，同时打开名为 `readme.txt` 的勒索信，但该勒索信中的内容并没有意义。

![获取随机秘钥和初始化向量](https://attach.52pojie.cn/forum/202412/12/095107map9twcbi9fbff69.png)

`Image-8.png` (35.96 KB, 下载次数: 0) · 2024-12-12 09:51 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA4OXw1MzUxMDU3NnwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

![AES 算法 CFB 加密模式加密](https://attach.52pojie.cn/forum/202412/12/095118x22p2pppsixwpsgh.png)

`Image-9.png` (72.39 KB, 下载次数: 0) · 2024-12-12 09:51 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA5MHw5OTRhMjBkMnwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

![移动至临时目录](https://attach.52pojie.cn/forum/202412/12/095132qtpdwt2p3id9p6m2.png)

`Image-10.png` (29.15 KB, 下载次数: 0) · 2024-12-12 09:51 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA5MXw2Y2M4ZmYxN3wxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

![readme.txt](https://attach.52pojie.cn/forum/202412/12/095144cblb1iv1ylxgbzl8.png)

`Image-11.png` (14.88 KB, 下载次数: 0) · 2024-12-12 09:51 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA5Mnw5Zjg5ZWZlYnwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

#### C 样本

C 样本首先获取 `Desktop`、`Documents`、`Pictures` 目录路径。随后获取随机秘钥，同时利用 RC4 算法将获取的目录下的文件进行加密。在加密过程中，将生成的秘钥保存至 `C:\Users\Administrator` 目录下。加密完成后，将勒索信创建在用户桌面上。

![获取目录路径](https://attach.52pojie.cn/forum/202412/12/095155a4030nc1phjpj3pp.png)

`Image-12.png` (50.02 KB, 下载次数: 0) · 2024-12-12 09:51 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA5M3wzNDY1ZjdhM3wxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

![RC4 加密](https://attach.52pojie.cn/forum/202412/12/095357k4f49e45n27dhq23.png)

`Image-13.png` (20.5 KB, 下载次数: 0) · 2024-12-12 09:53 上传 · [下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0MzA5NnxkOTRiN2E4NXwxNzkzNTc2Nzc5fDE2ODA4NnwxOTkwMzQw&nothumb=yes)

![Image-14.png](https://attach.52pojie.cn/forum/202412/12/095411it6vaagqjajgp9j7.png)

`Image-14.png` (54.17 KB, 下载次数: 0)
