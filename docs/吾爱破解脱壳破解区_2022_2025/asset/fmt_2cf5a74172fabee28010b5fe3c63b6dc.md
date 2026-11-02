
# [分享] WinRAR去广告已注册版打包方法及相关教程工具

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1835038
- 抓取时间: 2026-11-01T22:45:18

WinRAR去广告已注册版打包方法及相关教程工具：

## 第一步：安装 WinRAR

1. 安装 `winrar-x64-623sc.exe`，记住安装路径，后面会用到（比如安装在 `C:/Program Files/WinRAR`）。
2. 在 D 盘创建“WinRAR”文件夹，把 WinRAR 安装包解压到这个文件夹里面，或复制 WinRAR 安装目录下的 `WinRAR.exe` 到 D 盘，并把 WinRAR 注册文件 `rarreg.key` 放进去。

## 第二步：去除非商业个人版 WinRAR 广告

用授权文件注册后无广告版 WinRAR 可跳过此步。

用 Resource Hacker（[http://4275.com/qv4lne](http://4275.com/qv4lne)）或 Resource Tuner（[http://4275.com/kyubxr](http://4275.com/kyubxr)）打开刚刚复制的 `WinRAR.exe`，修改 `WinRAR.exe`，注意先关闭 WinRAR，点击字符串表 `String Table` -> 80：2052 -> 最长的那行 1277 行（有的 WinRAR 版本不一定是 1277 行）-> 为广告代码 -> 选定广告字串代码：

```
[Asm] 纯文本查看 复制代码
```
