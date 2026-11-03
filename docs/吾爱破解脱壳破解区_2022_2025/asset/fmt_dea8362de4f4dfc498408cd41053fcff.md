
# [CTF] DefCamp CTF 2025 onigirl 复盘详解

- 原帖：[https://www.52pojie.cn/forum.php?mod=viewthread&tid=2062956](https://www.52pojie.cn/forum.php?mod=viewthread&tid=2062956)
- 抓取时间：`2026-11-01T22:02:18`
- 作者：`selph`（我的另一个 id）
- 文章转载自 [先知社区](https://xz.aliyun.com/news/18992)

唯一一个困难 `pwn` 题，难是真难，为期 2 天的比赛总共只有 11 只队伍解出。该题是 `glibc-2.41` 下的题目，总共 3 个难点：如何绕过图像校验？如何在受限情况下进行堆利用？受限情况下如何劫持执行流？对于前半，本文将介绍完整的深入分析复杂流程的过程；对于后半，本文将结合 `glibc2.41` 源码来介绍 `fastbin` 与 `tcachebin` 的组合利用技巧，和 `exit` 中攻击向量。

![image-20250921234546-vu86vbe.png](https://attach.52pojie.cn/forum/202509/27/135743yppfp9rfi9zu9dkn.png)

`image-20250921234546-vu86vbe.png`（14.59 KB，下载次数：5）

[forum.php?mod=attachment&aid=MjgwNTYxMXxjZTk5NzBlYXwxNzkzNTcwNTM2fDE2ODA4NnwyMDYyOTU2&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwNTYxMXxjZTk5NzBlYXwxNzkzNTcwNTM2fDE2ODA4NnwyMDYyOTU2&nothumb=yes)

## 题目情况

题目来源：`DefCamp CTF 2025`（2025 年 9 月 14 号），困难 `pwn`。

保护全开，`glibc-2.41` 版本：

```
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
```

## 逆向分析（前半）：`privilege` 值生成

`main()` 函数很长，分段来看（这里是重命名后的结果）：

```
  v47 = __readfsqword(0x28u);

  setbuf(stdin, 0);
  setbuf(stderr, 0);
  setbuf(stdout, 0);

  random_seed = time(0);                        // 随机数
  srand(random_seed);

  privilege_value = 4;
  modification_parameters = modification_parameters_1;
  privilege_pointer = &privilege_value;

  printf("Enter image size in bytes: ");
  if ( !fgets(size_input_buffer, 32, stdin) )   // 输入size
  {
    fwrite("Failed to read size\n", 1u, 0x14u, stderr);
    return 1;
  }

  input_size = strtoul(size_input_buffer, 0, 10);// 转换成数字
  if ( !input_size || input_size > 0x10000 )    // 不能是0，不能>0x10000
  {
    fprintf(stderr, "Bad size (1–%d)\n", 0x10000);
    return 1;
  }

  allocated_buffer = malloc(input_size);        // 分配内存
  if ( !allocated_buffer )
  {
    perror("malloc");
    return 1;
  }

  bytes_read = read(0, allocated_buffer, input_size);

  if ( bytes_read 
```

- 读取用户输入的图像数据，手动输入大小和内容，需要自己构造。
- 加载图像并解析为 `RGB` 格式，这里通过 `stbi_load_from_memory()` 函数进行，该函数是静态编译进去的库函数。
- 对图像进行一系列复杂的浮点数运算（问了 AI 知道是像素级的处理与变换），大致如下：
  - 基于像素点到图像中心的距离进行颜色调整。
  - 引入随机扰动影响颜色计算。
  - 应用幂函数和参数矩阵对 `RGB` 分量进行非线性变换。
  - 某些像素进行 `XOR` 或 `NOT` 操作以引入噪声。
  - 在处理后生成一个“权限值”（`privilege`），后续进入一个交互式菜单系统，根据权限值（需要是 `0` 才行）决定用户可执行的操作（如分配、释放内存块等）。

## 利用分析&利用过程（前半）

### 分析如何得到 `privilege == 0`

让 `privilege` 的值为 `0` 的条件如下：

```
    privilege = *privilege_pointer != 4919;
```

需要 `privilege_pointer` 的值是 `4919`。此处我测试了一个只有 1 个像素的 `bmp` 图片，将图片输入，这里解析出图片 `RGB` 三个 `hex` 值，经过一系列运算后，向 `privilege` 指针进行异或操作：

```
        *privilege_pointer ^= (unsigned __int8)(pixel_pointer[1] & *pixel_pointer)
                            & (unsigned __int8)pixel_pointer[2]
                            & 0xF;
```

这个操作进行一系列 `&` 运算，结果就是 `privilege_pointer` 的值和一个 4 位的值进行异或。

然后紧接着的相关操作如下：

```
    *privilege_pointer |= 7u;
    *privilege_pointer &= 0x1FFFu;
    final_privilege_value = *privilege_pointer;
    *privilege_pointer = rand() & 0x3F | final_privilege_value;
    privilege = *privilege_pointer != 4919;
```

这里对 `privilege_pointer` 进行 `|` 运算，最后 3 位恒定设置为 1，然后进行 `& 0x1FFF`，保留最后 2 字节，最后再 `| rand() & 0x3F` 设置最低一字节一个值。

`4919` 的十六进制是 `0x1337`，最后的 7 是无论如何都会设置上的，问题在于中间的两个 3，最后的那个 3 的值取决于 `rand()` 函数的结果。也就是说，通过不断运行，就有概率得到 `0x37` 的最低字节值。

问题在于如何得到 `0x13` 的值？

这一块已经分析不出什么来了。注意啊！这是 `pwn` 问题，就要通过 `pwn` 的方式来完成目标！

回过头来，看看这个 `privilege_pointer` 怎么来的：

```
  privilege_value = 4;
  modification_parameters = modification_parameters_1;
  privilege_pointer = &privilege_value;
```

它们的定义：

```
  __int64 modification_parameters_1[10]; // [rsp+D0h] [rbp-A0h] BYREF
  int privilege_value; // [rsp+120h] [rbp-50h] BYREF
  char size_input_buffer[40]; // [rsp+130h] [rbp-40h] BYREF
```

可见，`privilege_value` 紧挨着 `modification_parameters_1` 数组。局部变量里就这么一个数组，碰巧有 `privilege_value` 紧挨着，是不是有可能这个数组会发生溢出来影响 `privilege_value` 的值呢？

继续深入分析 `modification_parameters_1` 参数：

```
  modification_parameters = modification_parameters_1;
  ...
  loaded_image_data = (char *)stbi_load_from_memory(
                                (__int64)allocated_buffer,
                                input_size,
                                (unsigned int *)&image_width,
                                (unsigned int *)&image_height,
                                (__int64 *)&p_menu_choice_storage,
                                3u,
                                modification_parameters_1);
```

然后是浮点运算用的了，正常使用逻辑下一般不会出现越界溢出，这里可疑的就是 `stbi_load_from_memory()` 函数，进去追踪该参数：

```
__int64 __fastcall stbi_load_from_memory(
        __int64 buf,
        int size,
        unsigned int *p_j,
        unsigned int *p_i,
        __int64 *p_menu_choice_storage,
        unsigned int n3,
        __int64 *modification_parameters)
{
  __int64 v12[30]; // [rsp+30h] [rbp-F0h] BYREF

  v12[29] = __readfsqword(0x28u);
  stbi__start_mem(v12, buf, size);
  return stbi__load_and_postprocess_8bit((int)v12, p_j, p_i, p_menu_choice_storage, n3, modification_parameters);
}
```

继续进入 `stbi__load_and_postprocess_8bit()`：

```
char *__fastcall stbi__load_and_postprocess_8bit(
        unsigned int *a1,
        unsigned int *p_j,
        unsigned int *p_i,
        __int64 *p_menu_choice_storage,
        unsigned int n3,
        __int64 *modification_parameters)
{
  unsigned int n3_1; // eax
  bool v8; // al
  int n3_2; // eax
  char *main; // [rsp+40h] [rbp-20h]
  unsigned int p_n8[3]; // [rsp+4Ch] [rbp-14h] BYREF
  unsigned __int64 v15; // [rsp+58h] [rbp-8h]

  v15 = __readfsqword(0x28u);
  main = stbi__load_main(a1, p_j, p_i, p_menu_choice_storage, n3, p_n8, 8, modification_parameters);
  if ( !main )
    return 0;
  if ( p_n8[0] != 8 && p_n8[0] != 16 )
    __assert_fail(
      "ri.bits_per_channel == 8 || ri.bits_per_channel == 16",
      "stb_image.h",
      0x50Cu,
      "stbi__load_and_postprocess_8bit");
  if ( p_n8[0] != 8 )
  {
    if ( n3 )
...
```

这里还要继续深入 `stbi__load_main()`。
