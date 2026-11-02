
```
        if (!*name2)
        {
            printf("正确\n");
        }
    }
    system("pause");
    return 0;
}
```

注册机的话就需要通过用户名计算序列号，从算法中可知验证的时候是对用户名和序列号中的字符依次进行验证，而运算过程又是异或运算，因此要从用户名得到序列号，只要将通过用户名字符串算法处理的结果，再用序列号字符串算法进行一次运算，就能得到正确的序列号。完整的注册算法程序如下：

```
#include "stdafx.h"
#include 
#include 
#include 

int _tmain(int argc, _TCHAR* argv[])
{
    char name[20],serial[20];
    int i,j;
    BYTE *name1, *serial1;
    printf("请输入用户名（长度6~20）：");
    scanf_s("%s", name, 20);
    name1 = (BYTE*)name;
    i = 1;
    do
        *name1++ ^= i++;
    while (*name1);
    //printf("name为：%s\n", name);
    serial1 = (BYTE*)name;
    j = 10;
    do
        *serial1++ ^= j++;
    while (*serial1);
    printf("序列号为：%s\n", name);
    system("pause");
    return 0;
}
```

注册结果如下：

![](https://attach.52pojie.cn/forum/202207/20/130326b1eg1ww3wabn1bdh.png)

`02.png` (11.25 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjUzODI3NHxjODZmOGFmNnwxNzkzNTc0NzU4fDE2ODA4NnwxNjYzNjkx&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzODI3NHxjODZmOGFmNnwxNzkzNTc0NzU4fDE2ODA4NnwxNjYzNjkx&nothumb=yes)

2022-7-20 13:03 上传

## 3. 总结

这是一个比较简单的注册算法题，前面的题能做出来的话，这道题应该也没有什么难度。
