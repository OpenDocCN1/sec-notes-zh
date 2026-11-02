
```
__asm
{   //内联汇编直接修改寄存器的值
    mov byte ptr [r13+99h], 0
}
```

按下 `Command + F9` 狠狠的编译出 `dylib` 文件。

![](https://attach.52pojie.cn/forum/202301/27/215433jmjhdeowshs5m5d8.jpg)

`16748244457951.jpg` (28.98 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyMHw5ZmQ1NDkxZXwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

好耶！

现在拿出我们的注入文件去注入app:

![](https://attach.52pojie.cn/forum/202301/27/215436vjudd3hp8zhkl3fr.jpg)

`16748245600178.jpg` (557.51 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyMXwxMjM3YjBjMXwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

`insert_dylib` 文件是编译自项目：`insert_dylib`，代码在开头自己编译。我不提供的原因是老有阴谋论在那边发癫说这个可执行文件有后门 让我们关SIP一定是你的阴谋 我要控制所有人的MacBook 统治整个世界！我是淫魔是吧？阴谋论歇歇吧别搁那发癫了。

注入完成打开app：

![](https://attach.52pojie.cn/forum/202301/27/215439f2rm6k3wopokw3b2.jpg)

`16748247844169.jpg` (151.64 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyMnxiMTBmZmY3NnwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy&nothumb=yes)

2023-1-27 21:54 上传

成功。

## 第四步 注意事项

我们想修改内存的是主程序二进制文件，所以我们千万不要注入任何影响到偏移地址有关的文件，如 `AirCore` 和 `AirBuddy` 二进制文件，为什么呢？因为这样会造成文件偏移错误。我们的偏移地址是根据 IDA 读出的原始二进制来算的，如果你注入了这些文件地址就会发生移位，这就无法通过原地址进行正确的注入了。所以我挑了个好欺负的小文件。也别去注入几 MB 的大文件，手动来回复制文件很麻烦的。

## 注入文件和补丁

仅供参考学习使用，有些人不要发癫！

试试注入 CleanMyMac 4.12.3 中国特供版，有彩蛋哦。

源码：`insert_dylib`

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyNHw2ZWVlNjUzY3wxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy)

(13 KB, 下载次数: 40)

2023-1-27 22:01 上传

点击文件名下载附件
下载积分: 吾爱币 -1 CB

`rd_toute`

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyNnw1ZGM0MGNiNXwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy)

(10.34 KB, 下载次数: 36)

2023-1-27 22:01 上传

点击文件名下载附件
下载积分: 吾爱币 -1 CB

编译好的二进制文件：

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyNXw4YzM2YWY3ZnwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy)

(8.47 KB, 下载次数: 44)

2023-1-27 22:01 上传

点击文件名下载附件
下载积分: 吾爱币 -1 CB

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU4NjcyOHxlYTdhYTliMnwxNzkzNTc0MTAzfDE2ODA4NnwxNzM5MTEy)

(10.44 KB, 下载次数: 50)

2023-1-27 22:03 上传

点击文件名下载附件
下载积分: 吾爱币 -1 CB

仅供参考学习使用。

注入方法：

```
sudo insert_dylib libInlineInjectPlugin.dylib /Applications/AirBuddy.app/Contents/Frameworks/LetsMove.framework/Versions/A/LetsMove /Applications/AirBuddy.app/Contents/Frameworks/LetsMove.framework/Versions/A/LetsMove
```

`libInlineInjectPlugin.dylib` 文件建议固定放在某个位置，注入成功后移动或删除文件都会导致注入失败。

`sudo insert_dylib`

## 参考资料 Reference&Credit

- 2023, Search Engine By Google.
- 2023, Bing Global Search Engine By Microsoft.
- 2023, MacOS动态注入的三种方式及hook方案, [https://blog.csdn.net/tangsilian/article/details/89442802](https://blog.csdn.net/tangsilian/article/details/89442802)
- 2023, macOS 逆向從入門到破解 Frida + Hopper + `dylib` 注入, [https://beeeeemo.dev/2021/08/macos-%E9%80%86%E5%90%91%E5%BE%9E%E5%85%A5%E9%96%80%E5%88%B0%E7%A0%B4%E8%A7%A3-frida-hopper-dylib-%E6%B3%A8%E5%85%A5/](https://beeeeemo.dev/2021/08/macos-%E9%80%86%E5%90%91%E5%BE%9E%E5%85%A5%E9%96%80%E5%88%B0%E7%A0%B4%E8%A7%A3-frida-hopper-dylib-%E6%B3%A8%E5%85%A5/)
- 2023, 404 WebSite by code-examples.net, [https://code-examples.net/zh-CN/q/204273](https://code-examples.net/zh-CN/q/204273)
- 2023, JianShu by [https://www.jianshu.com/p/3c8a9a6cee8d](https://www.jianshu.com/p/3c8a9a6cee8d)
- 2023, 404 WebSite by [https://github.com/rodionovd/rd_route](https://github.com/rodionovd/rd_route)
- 2023, 使用Frida优雅调试010 Editor [https://www.chinapyg.com/thread-134972-1-1.html](https://www.chinapyg.com/thread-134972-1-1.html)
- 2023，Objective-C的hook方案（一）: Method Swizzling by [https://blog.csdn.net/yiyaaixuexi/article/details/9374411](https://blog.csdn.net/yiyaaixuexi/article/details/9374411)
- 2023，运行时注入方式破解最新Mac版010 Editor v9.0.1 [https://www.52pojie.cn/forum.php?mod=viewthread&tid=861809&extra=page%3D1%26filter%3Dtypeid%26typeid%3D377](https://www.52pojie.cn/forum.php?mod=viewthread&tid=861809&extra=page%3D1%26filter%3Dtypeid%26typeid%3D377)
- 还有好多，懒癌犯了不想写了。
