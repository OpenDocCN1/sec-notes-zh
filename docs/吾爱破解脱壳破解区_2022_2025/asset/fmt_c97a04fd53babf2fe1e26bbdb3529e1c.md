
# [CTF] 0CTF2017 babyheap

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1817311
- 抓取时间：2026-11-01T22:48:37

## 前言

新注册的论坛账号，我目前在做ctf pwn和reverse，未来打算做android和iot，在吾爱这边准备更新一些实战记录和一些比赛的wp。

本题知识点我在这里有写：[链接](https://bbs.kanxue.com/thread-277530.htm)。

## 0ctf2017:babyheap

### 检查文件信息

![图片.png (151.35 KB, 下载次数: 1)](https://attach.52pojie.cn/forum/202308/04/143948efskeeoxky8eqkyr.png)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzAwNXwyOTZiODBmN3wxNzkzNTczMzE1fDE2ODA4NnwxODE3MzEx&nothumb=yes) 2023-8-4 14:39 上传

![图片.png (118.43 KB, 下载次数: 3)](https://attach.52pojie.cn/forum/202308/04/144022dgtggae1hehdkb1a.png)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzAwNnw1MmJlNTcxY3wxNzkzNTczMzE1fDE2ODA4NnwxODE3MzEx&nothumb=yes) 2023-8-4 14:40 上传

`elf64`小端序程序，保护全开。

### 修改`rpath`

检测到编译环境为`GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609`

![图片.png (119.56 KB, 下载次数: 1)](https://attach.52pojie.cn/forum/202308/04/144054pzujukjnyjun1jfg.png)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzAwN3w1MGRjNzljZnwxNzkzNTczMzE1fDE2ODA4NnwxODE3MzEx&nothumb=yes) 2023-8-4 14:40 上传

所以修改`rpath`为`glibc2.23`

![图片.png (252.08 KB, 下载次数: 2)](https://attach.52pojie.cn/forum/202308/04/144130zl05a371xb4r4p06.png)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzAwOHxiZDZkMTY5Y3wxNzkzNTczMzE1fDE2ODA4NnwxODE3MzEx&nothumb=yes) 2023-8-4 14:41 上传

### 试运行

![图片.png (106.26 KB, 下载次数: 1)](https://attach.52pojie.cn/forum/202308/04/144146etddy5syswua5yfr.png)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzAwOXwyMThmMjRlOHwxNzkzNTczMzE1fDE2ODA4NnwxODE3MzEx&nothumb=yes) 2023-8-4 14:41 上传

### 逆向分析

```c
//分配堆块函数
char *sub_B70()
{
  int fd; // [rsp+4h] [rbp-3Ch]
  char *addr; // [rsp+8h] [rbp-38h]
  unsigned __int64 v3; // [rsp+10h] [rbp-30h]
  __int64 buf[4]; // [rsp+20h] [rbp-20h] BYREF

  buf[3] = __readfsqword(0x28u);
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(_bss_start, 0LL, 2, 0LL);
  alarm(0x3Cu);
  puts("===== Baby Heap in 2017 =====");
  fd = open("/dev/urandom", 0);
  if ( fd 
```

```c
//Allocate函数
void __fastcall Allocate(__int64 a1)
{
  int i; // [rsp+10h] [rbp-10h]
  int v2; // [rsp+14h] [rbp-Ch]
  void *v3; // [rsp+18h] [rbp-8h]

  for ( i = 0; i  0 )
      {
        if ( v2 > 4096 )
          v2 = 4096;
        v3 = calloc(v2, 1uLL);//堆块内容初始化为1
        if ( !v3 )
          exit(-1);
        *(_DWORD *)(24LL * i + a1) = 1; //标记是否使用
        *(_QWORD *)(a1 + 24LL * i + 8) = v2;//标记大小Size
        *(_QWORD *)(a1 + 24LL * i + 16) = v3;//堆块地址
        printf("Allocate Index %d\n", (unsigned int)i);
      }
      return;
    }
  }
}

//结构体如下
struct Heap {
    double in_use;
    double size;
    void *heap;
};
```

```c
//Fill函数
__int64 __fastcall Fill(__int64 a1)
{
  __int64 result; // rax
  int v2; // [rsp+18h] [rbp-8h]
  int v3; // [rsp+1Ch] [rbp-4h]

  printf("Index: ");
  result = sub_138C();
  v2 = result; //序号
  if ( (unsigned int)result  0 )//存在堆溢出漏洞
      {
        printf("Content: ");
        return sub_11B2(*(_QWORD *)(24LL * v2 + a1 + 16), v3);//分配的heap在结构体偏移16的位置。
      }
    }
  }
  return result;
}
```

```c
//Free函数
__int64 __fastcall Free(__int64 a1)
{
  __int64 result; // rax
  int v2; // [rsp+1Ch] [rbp-4h]

  printf("Index: ");
  result = sub_138C();
  v2 = result;
  if ( (unsigned int)result 
```
