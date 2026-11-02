
# WriteFile 函数逆向分析

## 5. WriteFile 函数跟踪

`WriteFile` 函数，那决定最终写入 `Your_input` 文件的字符串重点就是在这两个函数了，所以我们就跟进 `401000`、`401140` 两个函数进行分析, 此时为了方便分析我们采用 `IDA` 来分析

### 拖入 IDA Pro 跳转 `401000`

拖进 `IDA Pro` 直接按 `G` 跳转到 `401000`（如下图）

![](https://attach.52pojie.cn/forum/202203/09/194002tufyaatwxfhfwfwh.png)

image-20220309180546210.png (30 KB, 下载次数: 262)

[href="forum.php?mod=attachment&aid=MjUwMDgyOHwyN2VkOGZmYXwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgyOHwyN2VkOGZmYXwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes)

### `401000` 函数代码

```
int __cdecl sub_401000(int a1, int a2)
{
  char i; // al
  char v3; // bl
  char v4; // cl
  int v5; // eax

  for ( i = 0; i = a2 )
      return 1;
  }
  return 0;
}
```

### 参数 `a1`、`a2` 解析

`401000` 两个参数 `a1`、`a2` 是什么呢？如下图

![](https://attach.52pojie.cn/forum/202203/09/194020cs3znu8088smu20t.png)

image-20220309181313579.png (53.64 KB, 下载次数: 265)

[href="forum.php?mod=attachment&aid=MjUwMDgyOXwxY2YyYzM5ZHwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgyOXwxY2YyYzM5ZHwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes)

可以看到 `esi` 是我们输入字符的长度，`edi` 是我们输入的字符长度，但是这里有个入栈细节

```
0040108C   .  56                   push esi
0040108D   .  57                   push edi
0040108E   .  E8 6DFFFFFF          call 1c40a4a4.00401000
```

这里的推入参数入栈是从参数的左边开始向右边推入，例如：

```
//c调用约定
void add(a,b){}
//对应汇编
push b
push a
call add
```

所以 `esi` 是 `IDA` 中伪代码的 `a2`，也就是我们输入字符的长度，`edi` 是 `IDA` 中伪代码的 `a1`，也就是我们输入字符所在的地址，那么此时我们阅读 `401000` 函数伪代码，就可以得到逻辑：

## 6. 深入逻辑分析

1. 循环 `a2`（`0x13`）次，循环体内：判断此时的循环次数是否为19次，如果是第19次循环的话则将我们输入的字符串第19位字符 `a[18]` 与 `0x13` 异或再返回第19位，如果循环次数取模2不为0的话则将 `a[i+a1]-i` 的值赋值给 `v3`, 否则将 `a[i+a1+2]` 赋值给 `v3`，最后，无论当前循环次数是否取模2等于0都将 `v3` 异或循环次数 `i` 的值赋值给 `a[i]`

2. 循环结束后判断 `a2`（我们输入字符的长度）是否小于等于0，满足的话则推出函数返回1，但是看来这个判断是毫无意义的

3. 判断我们输入的字符串的每一位字符是否等于 `byte_40A030` 数组中的每一个对应的值，等于的话就一直循环，知道循环了 `strlen(byte_40A030)` 后退出函数返回1，那我们查看一下 `byte_40A030` 数组是多少？（如图）

![](https://attach.52pojie.cn/forum/202203/09/194036ild72lnxl6fybca2.png)

image-20220309183642625.png (24.88 KB, 下载次数: 272)

[href="forum.php?mod=attachment&aid=MjUwMDgzMHxiZmNjZmNhOXwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgzMHxiZmNjZmNhOXwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes)

`strlen(byte_40A030)` 就是等于我们输入字符串的正确长度（`0x13`）, 其实这里可以有很大程度可以确定 `byte_40A030` 就是 `flag` 最后的加密结果，但是为了严谨我们还是通过函数调用来具体分析

![](https://attach.52pojie.cn/forum/202203/09/194051async8s3v4bs5sqk.png)

image-20220309183957081.png (24.13 KB, 下载次数: 264)

[href="forum.php?mod=attachment&aid=MjUwMDgzMXwxYjlmOGE2N3wxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgzMXwxYjlmOGE2N3wxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes)

如上图，找到了 `401000` 函数的调用者 `401080`，我们分析一下 `401080` 的逻辑：

定义整数变量 `v5` 接受 `401000` 的返回值（刚才分析过，只有0或1），接下来执行 `401140` 函数，跟进去看以后就只是一个 `HOOK` 相关的功能，不影响数据

最下面有一个判断，如果 `v5` 不为0则将 `lpNumberOfBytesWritten` 指向的值赋值为1

```
if ( v5 )
    *lpNumberOfBytesWritten = 1;
```

往上一看 `lpNumberOfBytesWritten` 就是 `WriteFile` 中的第三个参数，也就是我们设定写入文件的字节数，从这里就得不到更多信息了，所以我们现在需要找到 `WriteFile` 函数的调用者，如下图

![](https://attach.52pojie.cn/forum/202203/09/194110ooeir9romcm9az8c.png)

image-20220309190345090.png (49.56 KB, 下载次数: 266)

[href="forum.php?mod=attachment&aid=MjUwMDgzMnw1YWMxZWY0NHwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgzMnw1YWMxZWY0NHwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes)

主函数调用了 `WriteFile` 函数，我们先理一下主函数的逻辑是什么？

```
if ( NumberOfBytesWritten == 1 )
      sub_401370(aRightFlagIsYou);
    else
      sub_401370(aWrong);
```

通过跟入 `sub_401370` 函数发现该函数就是一个 `printf` 函数，参数就是输出的字符串变量，`aRightFlagIsYou` 变量是提示我们输入的 `flag` 正确的字符串，`aWrong` 字符串是提示我们输入的 `flag` 是错误的字符串，而要想执行提示我们输入正确则需要使得 `NumberOfBytesWritten == 1`，这个 `NumberOfBytesWritten` 就是 `HOOK` 了 `WriteFile` 的函数中做出的赋值更改，但是在主函数中查案代码发现 `NumberOfBytesWritten` 还参与了 `sub_401240` 函数，而且还是在 `hook` 之后，也就是说有可能这个 `sub_401240` 函数更改了 `NumberOfBytesWritten` , 那到底 `sub_401240` 函数是否对 `NumberOfBytesWritten` 做出更改还是得跟进去才知道如下图

![](https://attach.52pojie.cn/forum/202203/09/194130ez8p34b9crvovchc.png)

image-20220309191048516.png (29.82 KB, 下载次数: 264)

[href="forum.php?mod=attachment&aid=MjUwMDgzM3xjOGEyN2I5NHwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgzM3xjOGEyN2I5NHwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0&nothumb=yes)

`a1` 是我们输入得字符串所在的地址，`a2` 是 `NumberOfBytesWritten` 所在的地址，阅读代码逻辑：

将字符串 `v4="This_is_not_the_flag"` 中 `v4[a1 - v4 + result]` 元素与 `v4[result]` 对比，如果相等则循环以下代码：

```
if ( ++result >= (int)(v3 - 1) )
      {
        if ( result == 21 )
        {
          result = (int)a2;
          *a2 = 1;
        }
        return result;
      }
```

从上面代码中可以看出唯一对 `NumberOfBytesWritten` 操作且等于1的地方只有 `for` 中的 `if` 满足条件执行，那我们就想办法去执行这个 `*a2 = 1`，但是再仔细读代码后发现这个函数中我们输入的字符串未参与任何运算，也得不到一个中继的加密值，所以我们无法从这个函数中获得任何切确的信息，而且一看 `v4` 字符串是 `"This_is_not_the_flag"`，这很有可能说明作者写这个函数是用来误导我们的，所以综上所述我们只能去找 `NumberOfBytesWritten` 的另一个函数，也就是之前我们分析过的 `sub_401000` 函数，因为刚才我们上面已经分析过了，所以先通过 `sub_401000` 中的写出 `byte_40A030` 变量来反推出我们输入的 `flag` 因该是多少，解密脚本如下：

```
#include 
#include 

int main()
{
        int v3 = 0;
        unsigned char a1[] =
        {
                0x61, 0x6A, 0x79, 0x67, 0x6B, 0x46, 0x6D, 0x2E, 0x7F, 0x5F,
                0x7E, 0x2D, 0x53, 0x56, 0x7B, 0x38, 0x6D, 0x4C, 0x6E, 0x00
        };
        for (size_t i = 19; i >0; i--)
        {
                if (i == 18)
                {
                        *(BYTE *)(a1 + 18) ^=  0x13;

                }
                else
                {
                        v3 = *(BYTE *)(i + a1) ^ i;
                        if (i%2)
                        {
                                *(BYTE *)(i + a1) = v3 + i;
                        }
                        else
                        {
                                *(BYTE *)(i + a1 + 2) = v3;
                        }
                }
        }
        printf("%s",a1);
        system("pause");
        return 0;
}
```

此时我们得到了我们应该输入的 `flag`，此时 `NumberOfBytesWritten` 的值已经是1了，那我们继续带入下面的 `sub_401240` 中看这个函数是否会将 `NumberOfBytesWritten` 改为0就可以了，经过测试，`sub_401240` 带入我们推出的 `flag` 也是使得 `NumberOfBytesWritten` 变为1，所以 `main` 函数执行了输出提示正确的功能。

cm下载地址

![](https://static.52pojie.cn/static/image/filetype/zip.gif)

[href="forum.php?mod=attachment&aid=MjUwMDgzN3w3ZWRhZjY4YXwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMDgzN3w3ZWRhZjY4YXwxNzkzNTc2MDU5fDE2ODA4NnwxNjAwMDg0)

(21 KB, 下载次数: 10)

2022-3-9 19:49 上传

点击文件名下载附件

下载积分: 吾爱币 -1 CB
