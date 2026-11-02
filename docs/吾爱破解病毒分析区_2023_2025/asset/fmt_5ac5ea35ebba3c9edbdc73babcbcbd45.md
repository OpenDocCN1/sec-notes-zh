
[href="forum.php?mod=attachment&aid=MjczMDY3MHw0MDNhNGExNHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3MHw0MDNhNGExNHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

打开`\KnownDlls32\ntdll.dll`内存映射句柄

利用`NtSetInformationProcess`传递`ProcessInstrumentationCallback`参数设置回调为空，用于防止被安全软件检测到样本的系统调用。

![](https://attach.52pojie.cn/forum/202410/22/162232fcqik1tq65q7e81y.png)

Image-16.png (19.67 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3MXwyNWE4MzM5ZnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3MXwyNWE4MzM5ZnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

调用`NtSetInformationProcess`

通过遍历模块名计算哈希值判断是否在沙箱中，如果存在就会终止程序。

下图是通过网上比较常见的沙箱会加载的模块名计算比较得出的结果，其中一个没有被识别到。

![](https://attach.52pojie.cn/forum/202410/22/162234magzevghn6avff46.png)

Image-17.png (26.28 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3MnxkYTIyYjQ5OHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3MnxkYTIyYjQ5OHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

沙箱列表

以下是上述检测沙箱时会用到的模块名哈希算法，第一个参数是模块名，第二个参数会传入固定值`0x18D40B1A`。

![](https://attach.52pojie.cn/forum/202410/22/162236sz77uqu81h7b7uh7.png)

Image-18.png (126.61 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3M3w3MTU0N2IzZHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3M3w3MTU0N2IzZHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

计算模块名哈希值

还会通过调用`GetUserDefaultUILanguage`函数检测语言，具体代码为`0x419`：俄罗斯。如果是俄罗斯语言则不会进行窃密操作，如果不是则继续。

![](https://attach.52pojie.cn/forum/202410/22/162239cnc88uqaukcuzkjz.png)

Image-19.png (11.1 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3NHwwODQyNmM5ZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3NHwwODQyNmM5ZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

不支持警告

下载配置信息并开始窃密

获取远程服务器：

该样本通过域名轮询的方式，循环连接多个候选服务器，直到成功建立连接。如果所有候选服务器都无法连接，样本会访问 [//steamcommunity.com/profiles/76561199724331900" target="_blank"](https://steamcommunity.com/profiles/76561199724331900:rel="nofollow noopener" href="https)，获取页面中的 HTML 数据，使用 和 标签定位 Steam 账号名称。随后通过提取的 Steam 账号名称解密出远程服务器域名，目前分析时解密出的域名为 sergei-esenin.com。

由于 Steam 账号名称可以随时修改，木马作者能够通过动态更新 Steam 名称来控制和更新远程服务器的域名，从而保证样本的远程服务器持续存活。

![](https://attach.52pojie.cn/forum/202410/22/162241qk5potle0eo0oezo.png)

Image-20.png (58.77 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3NXw5Njc1NmI0ZnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3NXw5Njc1NmI0ZnwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

动态获取函数地址

![](https://attach.52pojie.cn/forum/202410/22/162243s688dsqq298qbxlx.png)

Image-21.png (121.52 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3Nnw3ZTM4OTk1NXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3Nnw3ZTM4OTk1NXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

获取远程服务流程

![](https://attach.52pojie.cn/forum/202410/22/162245ez9xwbbnxhx69xbf.png)

Image-22.png (326.7 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3N3w0YzAwMDJhMHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3N3w0YzAwMDJhMHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

Steam 名称

![](https://attach.52pojie.cn/forum/202410/22/162247otnn858vn1n8nc1n.png)

Image-23.png (103.38 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY3OHw3OTA2Mzk5N3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY3OHw3OTA2Mzk5N3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

Steam 名称 URL 解密代码

下载配置信息并利用：

样本通过 POST 请求向 [//sergei-esenin.com/api" target="_blank"](https://sergei-esenin.com/api:rel="nofollow noopener" href="https) 发送数据，请求参数包括`act=recive_message`（用于指定服务器行为）、`ver=4.0`（版本）、`lid=xAeOdp--mainteam`与`j=15f7911c5c73e2c263a9b433eb55ff31`。服务器响应返回加密的 JSON 配置数据，包含需要窃取的应用程序配置信息。之后对返回的加密数据进行 Base64 解码，随后通过异或运算与位运算的组合还原解密出 JSON 配置。

![](https://attach.52pojie.cn/forum/202410/22/162250s5b7szvt55gdx5vi.png)

Image-24.png (286.48 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY5OXxmYjUyYjdlOXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY5OXxmYjUyYjdlOXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

JSON 数据解密算法

可以看出 JSON 数据中包含重要文件、比特币钱包、浏览器配置信息，密码数据库等，该木马会按照该配置中的路径获取文件，最后会打包成压缩文件并发送到服务器中。

![](https://attach.52pojie.cn/forum/202410/22/162252d249c8wwgx9w4vms.png)

Image-25.png (146.61 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4MHw2NTZlZDBlZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4MHw2NTZlZDBlZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

部分 JSON 配置信息

下图是其中一个 Chrome 用户信息的压缩包：

![](https://attach.52pojie.cn/forum/202410/22/162254ivi4y9gw4zz67gva.png)

Image-26.png (29.28 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4MXw4NThkZmZlZHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4MXw4NThkZmZlZHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

压缩包

如果解析 JSON 数据时获取`se`的值为`true`则会截屏发送当前画面。

![](https://attach.52pojie.cn/forum/202410/22/162256ftdoztti332vkocn.png)

Image-27.png (38.98 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4MnwyMzU2YWM5N3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4MnwyMzU2YWM5N3wxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

截屏逻辑

获取邮件文件：

利用`RtlExpandEnvironmentStrings`展开`%LocalAppData%`，查看该文件夹下的`Packages`文件夹是否存在。

如果存在则利用通配符`microsoft.windowscommunicationsapps*`遍历文件夹，进入该文件夹递归的遍历`*.eml`邮件文件，最终将文件压缩到`Windows10Mail`压缩包中。

如果`Packages`文件夹不存在则会检查`%LocalAppData%\Microsoft\Windows Mail\Local Folders`文件夹是否存在，如果存在则遍历`*.eml`邮件文件，最终将文件压缩到`Windows10MailAlternative`压缩包中。

![](https://attach.52pojie.cn/forum/202410/22/162258tzih4noc7aohsb7i.png)

Image-28.png (25.32 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4M3xmYWQzNzE0ZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4M3xmYWQzNzE0ZXwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:22 上传

压缩包

下图是利用通配符遍历`*.eml`文件并压缩到`Mails/Windows10Mail`的函数调用图。

![](https://attach.52pojie.cn/forum/202410/22/162300zm85wx8555su5595.png)

Image-29.png (333.43 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjczMDY4NHxlMmZiNDUwNHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczMDY4NHxlMmZiNDUwNHwxNzkzNTc2ODA4fDE2ODA4NnwxOTc0Njg0&nothumb=yes)

2024-10-22 16:23 上传

函数调用图

获取 Outlook：

遍历`HKEY_USERS`注册表，并从中检测以下注册表路径：

- `Microsoft\Windows Messaging Subsystem\Profiles\9375CFF0413111d3B88A00104B2A6676`
- `Microsoft\Windows NT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook\9375CFF0413111d3B88A00104B2A6676`
- `Microsoft\Office\13.0\Outlook\Profiles\Outlook\9375CFF0413111d3B88A00104B2A6676`
- `Microsoft\Office\14.0\Outlook\Profiles\Outlook\9375CFF0413111d3B88A00104B2A6676`
- `Microsoft\Office\15.0\Outlook\Profiles\Outlook\9375CFF0413111d3B88A00104B2A6676`

从这些路径中循环获取`00000001 ~ 00000004`键中的值，最后将敏感信息写入`Applications/Outlook/Profiles.txt`文件中压缩。

获取 ThunderBird 文件：
