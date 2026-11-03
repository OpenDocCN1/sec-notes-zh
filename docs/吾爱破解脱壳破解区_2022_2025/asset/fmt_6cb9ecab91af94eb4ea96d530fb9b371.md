
# 代码分析

## 返回值构造
`v30 = PyNumber_Power(v28, *((_QWORD *)off_18000B688 + 38), Py_NoneStruct);//查表+38为65547`
`v31 = (int *)PyNumber_Remainder(v30, *((_QWORD *)off_18000B688 + 49));//查表+49为4294967293`
`v32 = PyTuple_New(2LL);//创建返回元组`
`*(_QWORD *)(v32 + 24) = v31;//元组第一个元素为v31`
`v15 = sub_180004360(v12, *((_QWORD *)off_18000B688 + 33), 5LL, 0LL);//查表+33为5`
`v3 = (int *)v15;`
`v24 = sub_180004360(v3, *((_QWORD *)off_18000B688 + 37), 23LL, 0LL);//查表+37为23`
`v21 = PyNumber_Lshift(v12, *((_QWORD *)off_18000B688 + 32));//查表+32为4`
`v12 = v21;`
`v16 = off_18000B688;`
`v22 = (int *)PyNumber_And(v12, v16[48], v17, v18);//查表v16[48]为4198170623`
`v25 = PyNumber_Add(v22, v24);`
`v3 = (int *)v25;`
`*(_QWORD *)(v32 + 32) = v3;//元组第二个元素为v3`

分析`sub_180004360`可知该函数为右移函数：

```
__int64 __fastcall sub_180004360(__int64 a1, __int64 a2, char a3, int a4)
{
  unsigned __int64 v4; // r9
  __int64 result; // rax
  unsigned __int64 v6; // r9
  __int64 v7; // rcx
  __int64 v8; // rcx
  __int64 (*v9)(void); // rax

  if ( *(_QWORD *)(a1 + 8) == PyLong_Type[0] )
  {
    v4 = *(_QWORD *)(a1 + 16);
    if ( (v4 & 1) != 0 )
    {
      if ( *(_DWORD *)a1 != -1 )
        ++*(_DWORD *)a1;
      return a1;
    }
    else if ( v4 >= 0x10 )
    {
      v6 = v4 >> 3;
      switch ( v6 * (1 - (*(_QWORD *)(a1 + 16) & 3LL)) )
      {
        case 0xFFFFFFFFFFFFFFFEuLL:
          v7 = -(__int64)(*(unsigned int *)(a1 + 24) | ((unsigned __int64)*(unsigned int *)(a1 + 28) > a3;
          result = PyLong_FromLongLong(v7, v7, 0x180000000uLL, v6);
          break;
        case 2uLL:
          v8 = (__int64)(*(unsigned int *)(a1 + 24) | ((unsigned __int64)*(unsigned int *)(a1 + 28) > a3;
          result = PyLong_FromLongLong(v8, v8, 0x180000000uLL, v6);
          break;
        default:
          result = (*(__int64 (__fastcall **)(__int64, __int64))(PyLong_Type[12] + 96LL))(a1, a2);
          break;
      }
    }
    else
    {
      return PyLong_FromLong((unsigned int)((int)(*(_DWORD *)(a1 + 24) * (1 - (v4 & 3))) >> a3));
    }
  }
  else
  {
    v9 = (__int64 (*)(void))PyNumber_Rshift;
    if ( a4 )
      v9 = (__int64 (*)(void))PyNumber_InPlaceRshift;
    return v9();
  }
  return result;
}
```

## `rand0m` 算法推导
猜测`rand0m`函数中`v12`为16进制字符串所表示的数字，可得到`rand0m`函数算法：

```
第一个返回值：((int(n,16)^2654435769)>>11)**65537%4294967293

第二个返回值：((int(n,16)>5)>>23)
```

经验证算法正确

![](https://attach.52pojie.cn/forum/202412/18/201553trlxxlqyr6xqlxam.png)

6.png (47.7 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0NDU3NnxmNTg2NjcxYnwxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDU3NnxmNTg2NjcxYnwxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes)

2024-12-18 20:15 上传

## `check` 函数分析
接着分析`check`函数，该函数一定调用了`rand0m`函数，于是在`rand0m`函数中调用`PyNumber_Xor`函数处下断点（其中一个参数（`ecx`）为`rand0m`接受的参数），输入`rand0m.check("123456789abcdefedcba98765432123456789")`，共断下来4次，查看`ecx`指向的地址：

```
Py_long(0x12345678):
0000010DB87AB070  01 00 00 00 00 00 00 00 10 28 F5 E1 FD 7F 00 00  .........(õáý...  
0000010DB87AB080  08 00 00 00 00 00 00 00 78 56 34 12 00 00 00 00  ........xV4.....  

Py_long(0x9ABCDEFE):
0000010DB8A67FD0  01 00 00 00 00 00 00 00 10 28 F5 E1 FD 7F 00 00  .........(õáý...  
0000010DB8A67FE0  10 00 00 00 00 00 00 00 FE DE BC 1A 02 00 00 00  ........þÞ¼.....  

Py_long(0xDCBA9876):
0000010DB87AB230  01 00 00 00 00 00 00 00 10 28 F5 E1 FD 7F 00 00  .........(õáý...  
0000010DB87AB240  10 00 00 00 00 00 00 00 76 98 BA 1C 03 00 00 00  ........v.º.....  

Py_long(0x54321234):
0000010DB8A67E50  01 00 00 00 00 00 00 00 10 28 F5 E1 FD 7F 00 00  .........(õáý...  
0000010DB8A67E60  10 00 00 00 00 00 00 00 34 12 32 14 01 00 00 00  ........4.2.....  
```

可以看出`check`读取输入的前32位（bug: ，它并没有判断输入长度，所以`flag`后面加任意数字都可以），每8位为一组调用`rand0m`函数

同时观察堆栈：

![](https://attach.52pojie.cn/forum/202412/18/201549nt6id4nrijh5ntit.png)

4.png (46.48 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0NDU3NHw0ZGFhZjRkN3wxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDU3NHw0ZGFhZjRkN3wxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes)

2024-12-18 20:15 上传

可以找到`check`函数调用`rand0m`的地方：

```
v52 = sub_180004660(v39, &v85[-v51], (unsigned int)(v51 + 1));
```

后面使用该返回值：

```
v34 = sub_180004790(v52, 1LL, v53, 0LL);
v39 = (int *)sub_180004790(v55, 0LL, v63, 0LL);
```

分析`sub_180004790`可知该函数用来根据索引值取出元素：

```
int *__fastcall sub_180004790(_QWORD *a1, __int64 a2, __int64 a3, int a4)

{

  __int64 v4; // rsi

  __int64 v5; // rbx

  BOOL v7; // eax

  unsigned __int64 v8; // rax

  int *result; // rax

  bool v10; // zf

  int v11; // ecx

  __int64 v12; // rbp

  __int64 v13; // rsi

  __int64 v14; // rax

  int *v15; // rbx

  __int64 Item; // rax

  __int64 v17; // rdi

  __int64 v18; // rax

  BOOL v19; // eax

  __int64 v20; // rax

  v4 = a1[1];

  v5 = a2;

  if ( v4 == PyList_Type )

  {

    v19 = a2 >= 0;

    if ( !a4 )

      v19 = 1;

    if ( !v19 )

      a2 += a1[2];

    if ( (unsigned __int64)a2 = 0;

    if ( !a4 )

      v7 = 1;

    if ( v7 )

      v8 = a2;

    else

      v8 = a2 + a1[2];

    if ( v8 = 0 )

        return (int *)(*(__int64 (__fastcall **)(_QWORD *, __int64))(v13 + 24))(a1, v18 + v5);

      if ( !(unsigned int)PyErr_ExceptionMatches(PyExc_OverflowError) )

        return 0LL;

      PyErr_Clear();

    }

    return (int *)(*(__int64 (__fastcall **)(_QWORD *, __int64))(v13 + 24))(a1, v5);

  }

  v14 = PyLong_FromSsize_t(a2);

  v15 = (int *)v14;

  if ( !v14 )

    return 0LL;

  Item = (*(__int64 (__fastcall **)(_QWORD *, __int64))(v12 + 8))(a1, v14);

LABEL_14:

  v17 = Item;

  if ( *v15 >= 0 )

  {

    v10 = (*(_QWORD *)v15)-- == 1LL;

    if ( v10 )

      Py_Dealloc(v15);

  }

  return (int *)v17;

}
```

## 期待值获取
继续分析可知去除两个元素后调用了`PyObject_RichCompare`进行比较，由于懒得静态分析期待值是什么，可以直接在调用`PyObject_RichCompare`处函数下断点查看内存，由于第二处调用需要第一处返回`True`才会执行，所以需要将第一处返回值修改为`Py_TrueStruct`

![](https://attach.52pojie.cn/forum/202412/18/201551v69bnkqk3cc10ekp.png)

5.png (49.64 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0NDU3NXw4MDYzZjNjOHwxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDU3NXw4MDYzZjNjOHwxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes)

2024-12-18 20:15 上传

之后观察`rdx`所指向的`Py_long`对象可以得到`rand0m`函数的期待值：

```
(0x98D24B3A,0x12287F38)
(0xE0F1DB77,0x4A30F74D)
(0xADF38403,0x23A1268)
(0xD8499BB6,0x88108807)
```

## 暴力求解与 Flag
由于用c语言写幂模会溢出不知道怎么处理，所以先用c语言暴力找出满足第二个参数的输入，然后再用python筛选满足第一个参数的输入（只给出第一部分的求解代码）：

```
#include 

#include 

int main() {

    uint64_t y = 0x12287F38;

    printf("a=[");

    for (uint64_t i = 0; i > 5) >> 23) == y) {

            printf("\"%08X\",", i);

        }

    }

    printf("]\n");

    return 0;

}
```
a=["812287F3","812297F3","8122C7F3","8122D7F3","812687F3","812697F3","8126C7F3","8126D7F3","812A87F3","812A97F3","812AC7F3","812AD7F3","812E87F3","812E97F3","812EC7F3","812ED7F3","813287F3","813297F3","8132C7F3","8132D7F3","813687F3","813697F3","8136C7F3","8136D7F3","813A87F3","813A97F3","813AC7F3","813AD7F3","813E87F3","813E97F3","813EC7F3","813ED7F3","816287F3","816297F3","8162C7F3","8162D7F3","816687F3","816697F3","8166C7F3","8166D7F3","816A87F3","816A97F3","816AC7F3","816AD7F3","816E87F3","816E97F3","816EC7F3","816ED7F3","817287F3","817297F3","8172C7F3","8172D7F3","817687F3","817697F3","8176C7F3","8176D7F3","817A87F3","817A97F3","817AC7F3","817AD7F3","817E87F3","817E97F3","817EC7F3","817ED7F3",]

for n in a:
    if(((int(n,16)^2654435769)>>11)**65537%4294967293==0x98D24B3A):
        print(n)

最后可得到满足条件的输入：

```
813A97F3D4B34F74802BA12678950880
```

但由于前文所说没有限制长度，所以输入可以是：
```
813A97F3D4B34F74802BA12678950880+任意16进制数字
```

最终`flag`为

```
flag{813A97F3D4B34F74802BA12678950880}
```

![](https://attach.52pojie.cn/forum/202412/18/201555r9xl5j2ugg9q89xs.png)

7.png (275.06 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc0NDU3N3w3ZDQ3YzY3NnwxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc0NDU3N3w3ZDQ3YzY3NnwxNzkzNTcxNDY0fDE2ODA4NnwxOTkyNDQ4&nothumb=yes)

2024-12-18 20:15 上传
