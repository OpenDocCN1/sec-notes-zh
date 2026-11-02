
# [CTF] HITCON CTF 2016 SleepyHolder

- 原帖：<https://www.52pojie.cn/forum.php?mod=viewthread&tid=1825577>
- 抓取时间：2026-11-01T22:47:47

## 检查文件信息

![图片.png](https://attach.52pojie.cn/forum/202308/25/151001iw7wy73tkz7p757y.png)

![图片.png](https://attach.52pojie.cn/forum/202308/25/151021uuqlfdyulwryhqiw.png)

`ELF64` 小端动态链接的 `bin`，没有开 `PIE` 和 `FULL RELRO`。

![图片.png](https://attach.52pojie.cn/forum/202308/25/151113d631kakrmtactokt.png)

`ubuntu14.04` 应该用 `glibc_2.19`，我这里用的 `glibc2.23`，如果要用 `2.19` 的 `libc`，可以整一个本地的 `libc-database`，然后下载相应版本即可。

![图片.png](https://attach.52pojie.cn/forum/202308/25/155433nsruko9xr7sovuu1.png)

改 `libc` 为 `glibc2.23`。

## 试运行

![图片.png](https://attach.52pojie.cn/forum/202308/25/151142gddy97gs8lyggl7c.png)

## 逆向分析

```
/* main */

void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)
{
  int v3; // eax
  unsigned int buf; // [rsp+4h] [rbp-1Ch] BYREF
  int fd; // [rsp+8h] [rbp-18h]
  int v6; // [rsp+Ch] [rbp-14h]
  char s[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v8; // [rsp+18h] [rbp-8h]

  v8 = __readfsqword(0x28u);
  sub_400CEB(a1, a2, a3);
  puts("Waking Sleepy Holder up ...");
  fd = open("/dev/urandom", 0);
  read(fd, &buf, 4uLL);
  buf &= 0xFFFu;
  malloc(buf); // 初始化堆
  sleep(3u);
  puts("Hey! Do you have any secret?");
  puts("I can help you to hold your secrets, and no one will be able to see it :)");
  while ( 1 )
  {
    puts("1. Keep secret");
    puts("2. Wipe secret");
    puts("3. Renew secret");
    memset(s, 0, 4uLL);
    read(0, s, 4uLL);
    v3 = atoi(s);
    v6 = v3;
    switch ( v3 )
    {
      case 2:
        wipe();
        break;
      case 3:
        renew();
        break;
      case 1:
        keep();
        break;
    }
  }
}
```

```
/* keep */

unsigned __int64 keep()
{
  int v0; // eax
  char s[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  puts("What secret do you want to keep?");
  puts("1. Small secret");
  puts("2. Big secret");
  if ( !huge_in_use )  // 只可申请一次huge
    puts("3. Keep a huge secret and lock it forever");
  memset(s, 0, 4uLL);
  read(0, s, 4uLL);
  v0 = atoi(s);
  if ( v0 == 2 )
  {
    if ( !big_in_use )
    {
      big_ptr = calloc(1uLL, 0xFA0uLL);
      big_in_use = 1;
      puts("Tell me your secret: ");
      read(0, big_ptr, 0xFA0uLL); // 可能泄露信息
    }
  }
  else if ( v0 == 3 )
  {
    if ( !huge_in_use )
    {
      huge_ptr = calloc(1uLL, 0x61A80uLL);
      huge_in_use = 1;
      puts("Tell me your secret: ");
      read(0, huge_ptr, 0x61A80uLL); // 可能泄露信息
    }
  }
  else if ( v0 == 1 && !small_in_use )
  {
    small_ptr = calloc(1uLL, 0x28uLL);
    small_in_use = 1;
    puts("Tell me your secret: ");
    read(0, small_ptr, 0x28uLL); // 可能泄露信息
  }
  return __readfsqword(0x28u) ^ v3;
}
```

```
/* wipe */

unsigned __int64 wipe()
{
  int v0; // eax
  char s[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  puts("Which Secret do you want to wipe?"); // 不可释放huge
  puts("1. Small secret");
  puts("2. Big secret");
  memset(s, 0, 4uLL);
  read(0, s, 4uLL);
  v0 = atoi(s);
  if ( v0 == 1 )
  {
    free(small_ptr);  // 未置零，UAF
    small_in_use = 0;
  }
  else if ( v0 == 2 )
  {
    free(big_ptr); // 未置零，UAF
    big_in_use = 0;
  }
  return __readfsqword(0x28u) ^ v3;
}
```

```
/* renew */

unsigned __int64 renew()
{
  int v0; // eax
  char s[8]; // [rsp+10h] [rbp-10h] BYREF
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  puts("Which Secret do you want to renew?");
  puts("1. Small secret");
  puts("2. Big secret");
  memset(s, 0, 4uLL);
  read(0, s, 4uLL);
  v0 = atoi(s);
  if ( v0 == 1 )
  {
    if ( small_in_use )
    {
      puts("Tell me your secret: ");
      read(0, small_ptr, 0x28uLL);  // 可能存在信息泄露
    }
  }
  else if ( v0 == 2 && big_in_use )
  {
    puts("Tell me your secret: ");
    read(0, big_ptr, 0xFA0uLL); // 可能存在信息泄露
  }
  return __readfsqword(0x28u) ^ v3;
}
```

## 漏洞利用

因为存在一次 `large bin` 的申请，会触发 `__malloc_consolidate`，而且存在 `UAF` 漏洞，可以通过 `fastbin_dup_consolidate` 来形成 `double free`，而且这个过程会把后一个 `chunk` 的 `PREV_INUSE` 位设为 0。而 `small_ptr` 的大小为 `0x28`，对齐时可以修改后一个 `chunk` 的 `prev_size` 位，可控空间大小为 `0x28`，可以构造 `fake_chunk` 进而形成 `unlink`，达到任意地址写。

### 首先进行 `unlink`
