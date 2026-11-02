
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyNXxkNTBlZTM2NnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

## `Progarm_main`函数逻辑分析

最开始仅仅只检测`cmd`命令行的第一个参数是不是`--self`，如果是需要将第二个参数，即程序本身路径作为`cmdline`的第一个参数，以防止后续读取自身文件时，使用第一个参数不是文件本身路径而是`--self`参数，导致后续读取密钥与程序配置内容失败。

![](https://attach.52pojie.cn/forum/202406/13/093007zabodllbah8bddnt.png)

3.3.2.png（15.88 KB，下载次数：6）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyNnw5OWQzNWQ3YnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

接下来进行了读取系统时间，以用作后续的日期判断，来对程序状态进行设置。

![](https://attach.52pojie.cn/forum/202406/13/093009uqlfjqjznw9vfqrg.png)

3.3.3.png（9.4 KB，下载次数：6）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyN3xiYmYxZDJlZHwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

再往下是初始化，初始化主要分为两部分，密钥初始化和程序初始化。这里再次提一下文件的结构组成：

![](https://attach.52pojie.cn/forum/202406/13/093011dz0032pipvl4fcih.png)

3.3.4.png（19.03 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyOHxjOWFiM2ZiOXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

其中初始化和LIVE1.0版本一样，都是两个初始化由`encryption_Initialize`函数和`program_Initialize`函数来实现。

![](https://attach.52pojie.cn/forum/202406/13/093014tjrrukp4uhkhga00.png)

3.3.5.png（8.21 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYyOXw1YmFiZTRkYnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

## 3.4 `encryption_Initialize`函数（密钥初始化）

密钥初始化核心为两部分，一个是从程序文件本身读取KEY，另外是从将硬编码的IV赋值。

### 读取KEY

原理和LIVE1.0版本一样，先从末尾读取4字节长度，再用`fseek`来不断地移动文件指针到key处。

![](https://attach.52pojie.cn/forum/202406/13/093016jqjbbuby1jbbbpvf.png)

3.4.1.png（29.14 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzMHwxMWUxNGMwMnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 赋值IV

![](https://attach.52pojie.cn/forum/202406/13/093018s5aidregp7z4yd57.png)

3.4.2.png（10.94 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzMXw5ZDMwZWE2N3wxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

整体的密钥初始化结束。

## 3.5 `program_Initialize`函数（程序初始化）

其功能是读取勒索信前面部分程序配置信息，并且将其转换为`key:value`的形式，以便于后续功能的使用。

### 读取配置项

原理和LIVE1.0版本一样，先读取末尾的勒索信长度，再依次移动文件指针到配置信息的开头。

![](https://attach.52pojie.cn/forum/202406/13/093020x49po6n7m0wmzm09.png)

3.5.1.png（29.05 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzMnwyNjUzMGY0NnwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 转化为`key:value`的形式

![](https://attach.52pojie.cn/forum/202406/13/093022dl8hfxve1i3tizpe.png)

3.5.2.png（19.38 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzM3xiOWNkY2VlYXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 文件内配置信息

```text
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

整体的程序初始化结束。

接下来是一个死循环，主要针对于文件本身的第一个参数以后的其他参数进行遍历，并且判断其功能是否开启等操作，具体功能介绍在`program_print_help`函数有介绍，如果参数未在其中，会触发`program_print_help`函数。

![](https://attach.52pojie.cn/forum/202406/13/093025gsz6mgc97vb63g5v.png)

3.5.3.png（149.12 KB，下载次数：4）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzNHxjZDRkZDBmZXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 时间比较

参数遍历完毕后，就是针对于时间的比较，从文件配置信息中可以得到一个最大与最小的日期信息，是个时间戳，转换一下可以得到其具体时间。

```text
  Mindate: 2023-01-01 08:00:00

  Maxdate: 2024-02-15 08:00:00
```

![](https://attach.52pojie.cn/forum/202406/13/093027plq3lhxd1asx3vhk.png)

3.5.4.png（6.78 KB，下载次数：5）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzNXwyZTg2ODMyZXwxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

### 逻辑实现

原理与LIVE1.0版本一样，都是根据Unix时间戳的形式来进行的判断，作用就是来判断软件是否在2023-01-01 08:00:00 ～ 2024-02-15 08:00:00之间，不在则不能够运行。

![](https://attach.52pojie.cn/forum/202406/13/093029s4pys8998z4uvm41.png)

3.5.5.png（12.44 KB，下载次数：4）

[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjcwMTYzNnw2MDg3OTVjN3wxNzkzNTc2ODY5fDE2ODA4NnwxOTM0MTc2&nothumb=yes)

2024-6-13 09:30 上传

接下来会从配置文件中读取勒索信的文件名称，并且从文件名中以`_`字符进行分割包含的ID，给到`progarms_id`变量以便后续的使用，最后获取更换壁纸功能的状态，如果是`true`的话就会进入到`progarm_ChangeWallpaper`函数进行勒索壁纸的更换。

### 配置中的变量内容
