
# [原创] 新手学破解记录贴【160个CrackMe】之001

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1984687
- 抓取时间: 2026-11-01T22:20:43

本人刚刚入门汇编逆向[52pojie.cn](https://www.52pojie.cn)分析，本帖将记录我对160个CrackMe程序中的第001程序的分析过程。在学习的过程中，我会边学边记录。如果有不正确之处，还请各位指正。

## 初步尝试与字符串搜索

首先打开程序后，输入假的用户名和密码，会直接弹出错误提示：`Sorry,The serial is incorect!`

![](https://attach.52pojie.cn/forum/202411/23/094527dd0i28yileo99szy.png)

[1.png (267.81 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQyNHw4MjVlY2Q0YnwxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:45 上传

现在把程序拖入到`x32dbg`里面来，来到程序模块进行搜索当前模块的字符串。

![](https://attach.52pojie.cn/forum/202411/23/094557etjuzt7h8sxaj2ay.png)

[2.png (276.82 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQyNXw1ODJiOWE3N3wxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:45 上传

![](https://attach.52pojie.cn/forum/202411/23/094613zgdrg37mbb38irb3.png)

[3.png (61.48 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQyNnxhN2JhODJkZnwxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:46 上传

## 断点与暴力破解

搜索完成，看到有两个输入不正确的提示，在这两个字符串上都下断点，然后重新输入用户名和密码进行注册，看是否会断下来。

![](https://attach.52pojie.cn/forum/202411/23/094636o4dxckq5ocd4qvkr.png)

[4.png (365.36 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQyN3xkYWFlMWNjNnwxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:46 上传

断在了第二个字符串的断点，一眼就看到了上方一个关键跳转，跳过了`Good job dude`这行代码。把上面的`jne`改成`je`，或者直接`nop`掉，叫他不进行跳转，然后再重新输入用户名和密码进行注册，这样就已经暴力破解成功了。

![](https://attach.52pojie.cn/forum/202411/23/094723u9vuvrabxuqpvhhp.png)

[5.png (383.15 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQyOXxmMmQ4ZWZkM3wxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:47 上传

## 注册码算法分析

通过之前断点可以看到一关键数据，其中`CW-7954-CRACKED`就是正确的注册码，这个注册码是怎么生成的具体再来分析一下。在地址头部`0042FA79`地址下一个断点，然后逐行分析一下代码。

![](https://attach.52pojie.cn/forum/202411/23/095012e8ezea1j1zmv87fq.png)

[6.png (416.87 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQzMHw3ZGJlNjNlN3wxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:50 上传

### 取用户名长度

首先，第一个操作是取了一下用户名的长度。

```asm
0042FA79 | 8D55 F0                 | lea edx,dword ptr ss:[ebp-10]                       | [ebp-10]:"a123456789"
0042FA7C | 8B83 DC010000           | mov eax,dword ptr ds:[ebx+1DC]                      | eax:"a123456789", [ebx+1DC]:&"d稝"
0042FA82 | E8 D1AFFEFF             | call acid burn.41AA58                               | 取用户名长度
```

然后把用户名`"a123456789"`放到了`eax`当中，`movzx eax, byte ptr ds:[eax]`会读取内存地址`eax`处的字节`'a'`，它的ASCII值是`0x61`。

接着执行`imul dword ptr ds:[0x00431750]`，`0x00431750`地址的值为`0x29`，也就是相当于`0x61 * 0x29 = F89`，这个时候的`eax`的值为`F89`。

下面又把`eax`放到了内存地址当中，然后又取出来再加上`eax`的值，相当于`eax * 2 = 1F12`，对应十进制就是`7954`，对应正确的激活码中间的哪个字段。

### 常量拼接

再往下看就是一些常量拼接的操作。

```asm
0042FAA3 | 8D45 FC                 | lea eax,dword ptr ss:[ebp-4]                        | [ebp-4]:"CW"
0042FAA6 | BA ACFB4200             | mov edx,acid burn.42FBAC                            | edx:"asasas", 42FBAC:"CW"
0042FAAB | E8 583CFDFF             | call acid burn.403708                               |
0042FAB0 | 8D45 F8                 | lea eax,dword ptr ss:[ebp-8]                        | [ebp-8]:"CRACKED"
0042FAB3 | BA B8FB4200             | mov edx,acid burn.42FBB8                            | edx:"asasas", 42FBB8:"CRACKED"
0042FAB8 | E8 4B3CFDFF             | call acid burn.403708                               |
0042FABD | FF75 FC                 | push dword ptr ss:[ebp-4]                           | [ebp-4]:"CW"
0042FAC0 | 68 C8FB4200             | push acid burn.42FBC8                               |
0042FAC5 | 8D55 E8                 | lea edx,dword ptr ss:[ebp-18]                       | [ebp-18]:"7954"
0042FAC8 | A1 50174300             | mov eax,dword ptr ds:[431750]                       | eax:"CW-7954-CRACKED"
0042FACD | E8 466CFDFF             | call acid burn.406718                               |
0042FAD2 | FF75 E8                 | push dword ptr ss:[ebp-18]                          | [ebp-18]:"7954"
0042FAD5 | 68 C8FB4200             | push acid burn.42FBC8                               |
0042FADA | FF75 F8                 | push dword ptr ss:[ebp-8]                           | [ebp-8]:"CRACKED"
0042FADD | 8D45 F4                 | lea eax,dword ptr ss:[ebp-C]                        | [ebp-C]:"CW-7954-CRACKED"
0042FAE0 | BA 05000000             | mov edx,5                                           | edx:"asasas"
0042FAE5 | E8 C23EFDFF             | call acid burn.4039AC                               | 传入 CW常量  计算出来的 7954 和 CRACKED常量 用 - 拼接出来密码
0042FAEA | 8D55 F0                 | lea edx,dword ptr ss:[ebp-10]                       | [ebp-10]:"asasas"
0042FAED | 8B83 E0010000           | mov eax,dword ptr ds:[ebx+1E0]                      | eax:"CW-7954-CRACKED", [ebx+1E0]:&"d稝"
0042FAF3 | E8 60AFFEFF             | call acid burn.41AA58                               |
0042FAF8 | 8B55 F0                 | mov edx,dword ptr ss:[ebp-10]                       | [ebp-10]:"asasas"
0042FAFB | 8B45 F4                 | mov eax,dword ptr ss:[ebp-C]                        | [ebp-C]:"CW-7954-CRACKED"
0042FAFE | E8 F93EFDFF             | call acid burn.4039FC                               | 真假码对比
```

到此就已经分析完了，总结一下具体的注册码生成算法，还是比较简单的：首先取用户名的第一个字节，然后乘以`0x29`再乘以2，算出来的值，前面拼接上`"CW-"`，后面再拼接上`"-CRACKED"`。

## 注册机实现

下面来写一下注册机。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main()
{
    //获取输入字符串
    string str;
    cout << "请输入用户名: ";
    cin >> str;

    //取字符串的第一个字符
    char ch = str[0];
    int num = ch * 0x29 * 2;

    //把num和字符串拼接起来
    string key = "CW-" + to_string(num) + "-CRACKED";
    cout << key << endl;
    return 0;
}
```

![](https://attach.52pojie.cn/forum/202411/23/095216fr6zoliqyr3z8r4q.png)

[7.png (18.72 KB, 下载次数: 0)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQzMXwzNmU5OTg5MXwxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:52 上传

![](https://attach.52pojie.cn/forum/202411/23/095217fh9k2yphkkv2zzzy.png)

[8.png (132.81 KB, 下载次数: 1)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjczODQzMnxhNDFlZWFlZHwxNzkzNTcxNjQwfDE2ODA4NnwxOTg0Njg3&nothumb=yes) 2024-11-23 09:52 上传
