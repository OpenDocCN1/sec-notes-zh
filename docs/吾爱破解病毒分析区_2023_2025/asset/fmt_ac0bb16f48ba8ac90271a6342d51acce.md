
# [PC样本分析] 【病毒分析】独家揭秘LIVE勒索病毒家族之2.0（全版本可解密）

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1934176
- 抓取时间: 2026-11-01T23:47:52

## 1.背景

### 1.1 来源

公司从客户现场提取出来了该勒索样本，通过分析发现该样本存在LIVE家族的特征，但与LIVE1.0的加密特征略有不同，故称为LIVE2.0版本。该文我们会对LIVE2.0加密流程进行分析，并在接下来的文章发布LIVE2.0勒索病毒的解密器。

### 1.2 介绍

LIVE2.0勒索病毒是LIVE家族的第二代恶意软件，该恶意软件会加密受害者的文件，并且将受害者的ID、文件名称和文件名称的长度写入到加密文件的末尾，最后会将文件名重命名成带`.LIVE`的文件 ，尤其是针对较重要的文本类文件会采用全部数据加密的方式。此外，它还会提供给受害者一封“`FILE RECOVERYID+受害者ID.txt`”的勒索信。

加密前：

![](https://attach.52pojie.cn/forum/202406/13/092938v415sbm5j0400x70.png)

1.2.1.png (139.95 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcwMTYxNHxiZTBkMjFlZnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYxNHxiZTBkMjFlZnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

加密后：

![](https://attach.52pojie.cn/forum/202406/13/092944cmk7e2b3sr8h47h6.png)

1.2.2.png (25.77 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjcwMTYxNXxmZmU2NmFjOXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYxNXxmZmU2NmFjOXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

## 2.文件分析

### 2.1 文件结构

与LIVE1.0版本不同的地方是密钥的长度从16byte变到了8byte，勒索壁纸换成了png格式的新图片。

![](https://attach.52pojie.cn/forum/202406/13/092946yzhzda6eouhzdddd.png)

2.1.png (19.03 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjcwMTYxNnwwZmU1NDAwMHwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYxNnwwZmU1NDAwMHwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

### 2.2 程序配置信息

```
  MinDate: int64 1672531200
  MaxDate: int64 1707955200
  AppendedExtension: 'LIVE'
  NoteFilename: 'FILE RECOVERY_ID_124188258340.txt'
  DeleteSystemRestorePoints: false
  StopServices: true
  ServicesToStop: ['MSSQLFDLauncher', 'MSSQLSERVER', 'SQLSERVERAGENT', 'SQLBrowser', 'SQLTELEMETRY', 'SQLWriter', 'MSSQL', 'SQLAgent', 'MSSQLServerOLAPService']
  StopProcesses: true
  ProcessesToStop: ['postgresql', 'sqlbrowser', 'sqlwriter', 'sqlservr', 'sqlceip', 'SQLAGENT', 'sqlservr', 'pg_ctl', 'postgres']
  DropNoteInSpecificDirectories: true
  DirectoriesToDropNoteIn: ['C:\Users\Public\Documents', 'C:\Users\Public\Desktop']
  IncludeFiles: false
  FileSet: ['C:\bootmgr', 'FILE RECOVERY_ID_124188258340.txt']
  IncludeDirectories: false
  DirectorySet: ['C:\Program Files', 'C:\Program Files (x86)', 'C:\ProgramData', 'C:\Windows', 'C:\msys64', 'C:\Users\All Users']
  IncludeExtensions: false
  NoneSet: ['exe', 'dll', 'ink', 'ini', 'lnk', 'ico', 'sys', 'desktop', 'mui', 'live']
  FastSet: @as []
  IntermittentSet: @as []
  FullSet: ['txt', 'log', 'doc', 'docx', 'msg', 'rtf', 'dat', 'ppt', 'pptx', 'xml', 'csv', 'xls', 'xlsx']
  EncryptHiddenFiles: true
  EncryptHiddenDirectories: false
  BufferSize: 8192
  Percent: 0.10000000000000001
  Segmentation: 512.0
  ChangeWallpaper: true
  WallpaperName: '124188258340.png'
```

### 2.3 勒索信

Hello....Your file has been encrypted and cannot be used..When you see this letter, your privacy data has been backed up by us. If you do not handle it, we will publish your privacy data after the 7th.....Don't try to change or restore the file yourself, which will destroy them..If necessary, you can decrypt a test file for free. Free test decryption is only available for files less than 3MB in size.....To restore files, you need a decryption tool. Please contact us by email...Please add the file name of this document to the email and send it to me. ..¡¾FILE RECOVERY_ID xxxxxx¡¿..I will tell you the amount you need to pay. After the payment is completed, we will make the decryption tool and send it to you.....Customer service mailbox:..locked@onionmail.org..Spare mailbox: (use this mailbox after no reply in 24 hours)..liveteam@onionmail.org....You can also contact us through intermediary agencies (such as data recovery companies)....If you refuse to pay, you will be attacked constantly. Your privacy -sensitive data will also be announced on Internet.....!! We are a team that pays attention to credibility, so you can pay safely and restore data.

### 2.4 勒索壁纸

![](https://attach.52pojie.cn/forum/202406/13/092949hu18pg4p3eme2uvp.png)

2.4.png (364.92 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjcwMTYxN3w4MGJkOTJjYXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYxN3w4MGJkOTJjYXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

### 2.5 密钥

![](https://attach.52pojie.cn/forum/202406/13/092951ydxsox8dds84dk7s.png)

2.5.png (213.54 KB, 下载次数: 6)

[href="forum.php?mod=attachment&aid=MjcwMTYxOHwwMzI5OTA3M3wxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYxOHwwMzI5OTA3M3wxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

### 2.6 加密文件结构

其中原始文件名长度和`progarm_id`都是以小端序的方式存储的。

![](https://attach.52pojie.cn/forum/202406/13/092954pgpkoa6pwykpwyfa.png)

2.6.png (8.39 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjcwMTYyMHw2Y2E5Y2Q2OXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyMHw2Y2E5Y2Q2OXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

### 2.7 加密文件样式

加密文件名特征：随机数+`.LIVE`

加密文件目录样式：

勒索信+加密文件

![](https://attach.52pojie.cn/forum/202406/13/092956je1hc0lyycyhcjee.png)

2.7.1.png (75.41 KB, 下载次数: 3)

[href="forum.php?mod=attachment&aid=MjcwMTYyMXwyYzEyZjFhOHwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyMXwyYzEyZjFhOHwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

解密后：

![](https://attach.52pojie.cn/forum/202406/13/092958v3o11ed8xrzvxvvd.png)

2.7.2.png (139.95 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjcwMTYyMnw3YzhhZDVmNnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyMnw3YzhhZDVmNnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:29 上传

## 3.程序运行分析

初步来看加密器并未使用保护措施，并且没有移除符号表，入口点为`main`。

![](https://attach.52pojie.cn/forum/202406/13/095507gmcnm0zp7o7rrffn.png)

3.png (204.58 KB, 下载次数: 4)

[href="forum.php?mod=attachment&aid=MjcwMTY5MXxkODU5MzEzYnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTY5MXxkODU5MzEzYnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:55 上传

### 3.1 Main函数

从`Main`函数中可以知道主要逻辑在`progarm_main`函数中，跟LIVE1.0版本一致。

![](https://attach.52pojie.cn/forum/202406/13/093000rwlgyqhvyjqzvxxv.png)

3.1.png (11.34 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjcwMTYyM3xmM2Y1NDBiYnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyM3xmM2Y1NDBiYnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 3.2 Progarm_main函数

首先程序会获取程序运行中的命令行，并且根据其参数内容进行其他功能的使用。

![](https://attach.52pojie.cn/forum/202406/13/093002ulbprza6lu4f2nfj.png)

3.2.png (16.77 KB, 下载次数: 5)

[href="forum.php?mod=attachment&aid=MjcwMTYyNHxkNjViZTc0Y3wxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyNHxkNjViZTc0Y3wxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 3.3 program_print_help函数（打印使用说明）

具体实现在函数`program_print_help`函数中有所说明，如：指定加密的目录、删除系统还原点、停止服务等，使用方法是`--参数` `true/false`，跟LIVE1.0版本一致。

![](https://attach.52pojie.cn/forum/202406/13/093005brrqj0lrjpnoj1rl.png)

3.3.1.png (299.31 KB, 下载次数: 4)
