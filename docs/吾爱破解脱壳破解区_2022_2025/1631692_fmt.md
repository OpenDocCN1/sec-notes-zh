
# [原创] 练习笔记之160Crackme-010

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1631692
- 抓取时间: 2026-11-01T23:26:56

## 160CM-010

### 1.破解过程

这道题看难度提示为“★★★”，以为会是一道难题，结果VB Decompiler打开一看，发现和009基本相同，只是算法不同。爆破就不搞了，分析了一下算法，也比较简单，就是输入的字符串每个字符ASCII码加10后的字符串与`"kXy^rO|yXom\kMuOn*+"`比较，相同就成功，完全没有三星的难度。

![02.png](https://attach.52pojie.cn/forum/202205/05/114156q436gdn984ttjedt.png)

`02.png`（24 KB，下载次数：2） · [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTY4N3xmMGMyZjAyZXwxNzkzNTc1NjE0fDE2ODA4NnwxNjMxNjky&nothumb=yes) · 2022-5-5 11:41 上传

VS中开搞，码入代码：

```
#include "stdafx.h"
#include 
#include

int _tmain(int argc, _TCHAR* argv[])
{
    char key[30] = "kXy^rO|*yXo*m\kMuOn*+";
    char code[30] = "";
    int i;
    for (i=0;i
```

运行后，得到结果如下：

![03.png](https://attach.52pojie.cn/forum/202205/05/114158j3s4l422u4u4gs2u.png)

`03.png`（10.24 KB，下载次数：2） · [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTY4OHxjNDM0YjQzM3wxNzkzNTc1NjE0fDE2ODA4NnwxNjMxNjky&nothumb=yes) · 2022-5-5 11:41 上传

输入文本框，点击OK，发现居然没成功。然后，OD调试、VB Decompiler重新反编译。。。一顿操作猛如虎，结果还是没发现问题出在哪里，算法看起来没有任何问题，一度怀疑`vbaVarTstEq`函数是不是不光比较字符串内容，还要变量体的数据类型代码完全相同，因为一个是`08`，一个是`8008`，但是程序代码里面根本没有修改变量体数据类型的其他操作。

瞎忙活了半天，结果还是在打印出来的字符串中发现了问题，因为打印出来的字符和输入到程序代码里面的字符`"kXy^rO|yXom\kMuOn*+"`相比，少了一个`\`。呃……，这才想起`\`在C语言好像是有特殊含义的转义符。

在字符串中增加一个`\`，新的算法代码如下：

```
#include "stdafx.h"
#include 
#include

int _tmain(int argc, _TCHAR* argv[])
{
    char key[30] = "kXy^rO|*yXo*m\kMuOn*+";
    char code[30] = "";
    int i;
    for (i=0;i
```

编译，运行。

![01.png](https://attach.52pojie.cn/forum/202205/05/114154lbs0wp88asn6b0tw.png)

`01.png`（10.16 KB，下载次数：1） · [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTY4Nnw3ZWRlM2VhOHwxNzkzNTc1NjE0fDE2ODA4NnwxNjMxNjky&nothumb=yes) · 2022-5-5 11:41 上传

复制注册码输入文本框，确定，成功！

![04.png](https://attach.52pojie.cn/forum/202205/05/114200ffikpgow8wgrfuoz.png)

`04.png`（14.01 KB，下载次数：3） · [附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUxOTY4OXw4YWE1ZjhmNnwxNzkzNTc1NjE0fDE2ODA4NnwxNjMxNjky&nothumb=yes) · 2022-5-5 11:42 上传

### 2.总结

这道题实际感觉应该比009还简单一些，但因为粗心大意，碰到一个比较SB的问题，也花费了不少时间。

- C语言中，单引号`'`、双引号`"`、反斜杠`\`都是特殊的字符，不能直接表示，需要在前面加一个反斜杠`\`。
