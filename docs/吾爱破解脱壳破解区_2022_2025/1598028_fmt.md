
# [分享] The043. 吾爱培训，《09：深入浅出探讨脱壳细节》讲师：L4Nce [OD脚本编写]

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=1598028`
- 抓取时间: `2026-11-01T23:34:47`

关于吾爱论坛官方入门教学培训第一期，我的学习记录，如有疑问欢迎交流指正。^_^

好几天没发帖了，换了台电脑，折腾各种系统和软件，终于完美了。

这次直接来到第九课，解释一下：第7课珈蓝夜雨的网络验证，我真没听懂，能力不足。第8课是脱壳铺垫，没有具体内容，所以没有记录。第9课讲的`OD`脱壳脚本编写，挺有意思，找回了以前写程序的感觉。

`第八九课作业.exe` -> `OD`脱壳脚本编写。

## 0x0 首先查壳

首先查壳，`tElock 0.98b1` -> `tE!`。在编写`OD`脱壳脚本之前，我们得先自行手脱这个壳，然后将脱壳的步骤用`OD`脚本语言表达出来，才能达到目的。我是用最后一次异常法脱壳的，之前的学习记录里面有方法：`https://www.52pojie.cn/thread-1575502-1-1.html`。

## 0x1 编写脚本

掌握了脱壳方法后，开始编写脚本。下面是`OD`的调试界面，脚本可以边写边调试，实时测试脚本效果。

## 0x2 脚本记录

下图是我在`txt`文件里的脚本记录，`txt`文件里有详细注释，载入到`OD`时屏蔽了注释了。

## 0x3 文字版脚本

文字版的如下：

```
// Chuiyan tElock 0.98b1 -> tE! OEP Finder
// 此脚本将迅速带你到达OEP

msg "请Alt+O设置OD异常设置不要忽略任何异常，然后Ctrl+F2重载程序"  //消息框

var cbase                                //定义变量, 记录代码段基地址(00401000)
gmi eip, CODEBASE                //获取代码段基地址(gmi地址信息, CODEBASE代码段基地址)
mov cbase, $RESULT                //将基地址传递给cbase变量   
log cbase                                //将cbase输出到OllyDbg的记录窗口[Script Log window]中, 记录用

var csize                                //定义变量, 记录代码段大小(0007C000)
gmi eip, CODESIZE                //获取代码段大小(gmi地址信息, CODESIZE代码段大小)
mov csize, $RESULT                //将大小传递给csize变量
log csize                                //将csize输出到OllyDbg的记录窗口[Script Log window]中, 记录用

esto                                        //Shift+F9  1次
esto                                        //Shift+F9  2次
esto                                        //Shift+F9  3次
esto                                        //Shift+F9  4次
esto                                        //Shift+F9  5次
esto                                        //Shift+F9  6次
esto                                        //Shift+F9  7次
esto                                        //Shift+F9  8次
esto                                        //Shift+F9  9次
esto                                        //Shift+F9  10次
esto                                        //Shift+F9  11次
esto                                        //Shift+F9  12次
esto                                        //Shift+F9  13次
esto                                        //Shift+F9  14次
esto                                        //Shift+F9  15次
esto                                        //Shift+F9  16次, 最后一次异常停在这里

bp 004C6817                                //下F2断点
esto                                        //Shift+F9运行
bc 004C6817                                //取消F2断点
bprm cbase, csize                //下内存访问断点
esto                                        //Shift+F9运行
bpmc                                        //取消内存访问断点

cmt eip,"OEP在此, By Chuiyan!"  //添加注释
```

## 0x4 运行效果

空格键运行脚本，自动到达`OEP`，通过脚本可以提高工作效率，特别是繁琐的重复工作。
