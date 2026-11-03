
# [Android 分享] Game修改器2.7优化界面，支持Frida双模式执行脚本

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1764607
- 抓取时间: 2026-11-01T21:45:43

https://www.123pan.com/s/VDcDVv-n1rfA.html

请手动打开通知权限，`toast`提示需要

请手动打开 后台弹出界面 权限

经测试个别系统可能没有弹出权限会崩溃

这个权限无法动态申请。。。

![](https://attach.52pojie.cn/forum/202303/25/141611dh1h5gf5fzgchplf.jpg)
IMG_20230325_133833.jpg (112.15 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjYwMTYxNXwxNjE3Mzg1MHwxNzkzNTY5NTQxfDE2ODA4NnwxNzY0NjA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMTYxNXwxNjE3Mzg1MHwxNzkzNTY5NTQxfDE2ODA4NnwxNzY0NjA3&nothumb=yes)
2023-3-25 14:16 上传

![](https://attach.52pojie.cn/forum/202303/25/141619gam0nf39cncajpp9.jpg)
IMG_20230325_133905.jpg (105.8 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYwMTYxNnw2ZDUyYzdkM3wxNzkzNTY5NTQxfDE2ODA4NnwxNzY0NjA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMTYxNnw2ZDUyYzdkM3wxNzkzNTY5NTQxfDE2ODA4NnwxNzY0NjA3&nothumb=yes)
2023-3-25 14:16 上传

![](https://attach.52pojie.cn/forum/202303/25/141624fomfherou8cou2zt.jpg)
IMG_20230325_133924.jpg (46.51 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjYwMTYxN3xlM2IzYzg1OHwxNzkzNTY5NTQxfDE2ODA4NnwxNzY0NjA3&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYwMTYxN3xlM2IzYzg1OHwxNzkzNTY5NTQxfDE2ODA4NnwxNzY0NjA3&nothumb=yes)
2023-3-25 14:16 上传

## Frida脚本指令使用注意

`[pack]`代表包名输入框内的文本

`[pid]`代表通过包名输入框内文本获取指定程序`pid`

`[path]`代表选择脚本按钮所选择的脚本路径

### 指令使用方式一

`-e -f [pack] -s [path]`

### 指令使用方式二

`-n com.myapp -s /sdcard/hook.js`

注意指令中间必须用空格隔开，与电脑端命令行`Frida`指令一样

脚本执行记录会在`/sdcard/Hook/log.txt`生成脚本执行结果日志

暂时没测试出其他问题，发个新帖说明下更新`Frida`脚本使用方式
