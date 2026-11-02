
# 注入劫持模块

## 传奇登陆器劫持模块

该模块主要负责劫持用户电脑中的传奇登陆器，来推广指定的传奇。模块启动之后，会先从C&C服务器中获取相关配置文件，再根据配置文件进行检测，如果存在指定的传奇客户端就进行劫持。获取配置信息相关代码，如下图所示：

![Image-34.png](https://attach.52pojie.cn/forum/202312/21/172509vh77krt2xtzzt1bd.png)

### 获取配置信息代码

由于安全工程师在调试的过程中并未获取到相关配置信息，无法进行动态调试。以下是分析人员通过对恶意文件的静态分析的结果，分析发现配置信息中存在几个字段：`icomd5`，`filemd5`等字段，病毒通过这几个字段信息来确定要劫持的登陆器，以`icomd5`为例，模块会遍历系统中的进程并计算图标文件的MD5值来判断是否为劫持的登陆器，计算图标MD5相关代码，如下图所示：

![Image-35.png](https://attach.52pojie.cn/forum/202312/21/172517mfcmac68g6mfog39.png)

### 计算图标MD5

确认是要劫持的登陆器之后，会将劫持模块注入到目标进程中，相关代码，如下图所示：

![Image-36.png](https://attach.52pojie.cn/forum/202312/21/172527j6akgigrwkiilzim.png)

### 注入劫持模块

在劫持模块中会对`ShowWindow`函数进行HOOK，相关代码，如下图所示：

![Image-37.png](https://attach.52pojie.cn/forum/202312/21/172536kqtmsdmvxg55jgm5.png)

### HOOK `ShowWindow`

在`HookShowWindow`中，会创建一个新的窗口来替代原窗口，相关代码，如下图所示：

![Image-38.png](https://attach.52pojie.cn/forum/202312/21/172544fk4kk418usukl21k.png)

### 劫持登陆器界面

# 反调试模块

反调试模块启动后，会通过枚举Windows窗口的类名和标题来检测是否存在特定的调试软件，相关代码，如下图所示：

![Image-39.png](https://attach.52pojie.cn/forum/202312/21/172551r2ouo95mu5ip5957.png)

### 会检查的窗口标题和类名列表

如下图所示：

![Image-40.png](https://attach.52pojie.cn/forum/202312/21/172601oyd1d1dy3r1z137l.png)

# 附录

## C&C

![Image-41.png](https://attach.52pojie.cn/forum/202312/21/172608pcmjws6cmyy62sbs.png)

## HASH

![Image-42.png](https://attach.52pojie.cn/forum/202312/21/172616wvkdtb1133bambck.png)
