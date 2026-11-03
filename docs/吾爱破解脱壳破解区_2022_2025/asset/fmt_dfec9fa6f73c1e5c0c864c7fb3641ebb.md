
# [CTF] KCTF 2025 第七题 WriteUP

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2056628
- 抓取时间: 2026-11-01T22:03:53

## 1. 开始

看着是 iOT 的题要跑虚拟机，实则 `IDA` 都能做出来（雾

`IDA` 直接加载完主程序，检验逻辑直接写在 `main()` 里面了，但是有一些不明所以的变量

![](https://attach.52pojie.cn/forum/202508/29/130400qa1oe5zojas4i9i5.png)

image.png (54.52 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=Mjc5OTc0NHw2MmQ3ODc3MHwxNzkzNTcwNjMxfDE2ODA4NnwyMDU2NjI4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=Mjc5OTc0NHw2MmQ3ODc3MHwxNzkzNTcwNjMxfDE2ODA4NnwyMDU2NjI4&nothumb=yes)

2025-8-29 13:04 上传

直接复制粘贴到 `VS Code` 里面改变量名就舒服多了 －O－

## 2. 分析

### 2.1 string_to_code 中 字符串和对应数值的映射

直接看伪代码，可以推出 `act=0`, `con=0`, `abort=1`, `cancel=1`, `enable=2`, `start=2`, `run=2`, `stop=3`, `reset=4`, `reboot=4`

于是我们可以将伪代码中 `string_to_code(x)` 直接换成对应的数值

### 2.2 `v39` 缓冲区填充

```
  p_argc = &argc;
  printf("key:");
  if ( fgets(s, 30, &Stdin) )
  {
    v3 = strcspn(s, "\n"); // 去掉换行符 用\x00代替
    s[v3] = 0; 
    idx_1 = 0;
    // string_to_code("act"); 无意义

    // 将 key 的前 10 个字符 转为 数字 存入 v39[25] ~ v39[34]
    // d0=v39[25], d1=v39[26], ..., d9=v39[34]
    for ( strIdx = 0; s[strIdx] && strIdx  57 ) // 非 '1' ~ '9'
      {
        /*
        a ~ f  转 10 ~ 15
        其他字符转 16
        */
        if ( s[strIdx] == 97 ) // a
        {
          v39[idx_1++ + 25] = 10;
        }
        else if ( s[strIdx] == 98 ) // b
        {
          v39[idx_1++ + 25] = 11;
        }
        else if ( s[strIdx] == 99 ) // c
        {
          v39[idx_1++ + 25] = 12;
        }
        else if ( s[strIdx] == 100 ) // d
        {
          v39[idx_1++ + 25] = 13;
        }
        else if ( s[strIdx] == 101 ) // e
        {
          v39[idx_1++ + 25] = 14;
        }
        else if ( s[strIdx] == 102 ) // f
        {
          v39[idx_1++ + 25] = 15;
        }
        else
        {
          if ( s[strIdx] == 48 ) // '0'
            v39[idx_1 + 25] = 0;
          else // 其他字符兜底为 16
            v39[idx_1 + 25] = 16;
          ++idx_1;
        }
      }
      else // '1' ~ '9'
      {
        v39[idx_1++ + 25] = s[strIdx] - 48; // 转数字
      }
    }
```

### 2.3 `v39` 5x5 矩阵 和 `v38` + 272 5x3 矩阵 的填充

```
    col = 0;

    next = 0;

    v42 = 0;

    v43 = 0;

    v44 = 0;

    v45 = 0;

    v46 = 0;

    v47 = 0;

    for ( row = 0; row  第一行 d0 d2 d3

        v43 -> 第二行 d1 d3 d6 

        v44 -> 第三行 d2 d4 d8

        v45 -> 第四行 d4 d7 d9

        v46 -> 第五行 d5 d6 d7

        */
```

### 2.4 第一次 `v47` 计算

```
        v22 = 4;
        if ( v22 == row ) // row==4
          v47 += v39[5 * row + col];
      }
      if ( stat("/etc/rc.d", &buf) ) // 该目录在 QNX 下是存在的 所以不会给 v47 额外加值
        v47 += *(_DWORD *)&v38[12 * row + 272 + 4 * col]; // row = 0 ~ 4, col=0

      // v47 最终等于 4 + 5 + 7 + 8 + d9 = 24 + d9
```

### 2.5 `v38` + 272 5x3 矩阵 的数值检查

```
    v48 = 1; // 检查成功flag

    // 取 v38 + 272 的行和 应该都为 34

    for ( row = 0; row  每一行的和应该都为 34

        v48 = 0;

    }

    /*

    可得等式

    d0 + d2 + d3 = 34

    d1 + d3 + d6 = 34

    d2 + d4 + d8 = 34

    d4 + d7 + d9 = 34

    d5 + d6 + d7 = 34

    */
```

### 2.6 QNX 系统特定检查

```
    // https://www.qnx.com/developers/docs/7.0.0/com.qnx.doc.neutrino.prog/topic/process_DCMD_PROC_INFO.html
    memset(v38, 0, 0x110u);                 // 在 QNX 这两步成功 所以不会给 v47 额外 +1
    v49 = open("/proc/self/as", (int)"r");      
    if ( v49 == -1 )
      ++v47;
    if ( devctl(v49, 0x41100801, v38, 272, 0) ) // 0x41100801 -> DCMD_PROC_INFO
      ++v47;
```

### 2.7 `d0`,`d9` 定值范围检查

```
    v23 = v47;

    v24 = v39[0] + v39[3] * v39[2] * v39[1] * v39[4] + (v38[8] & 0x80); // 0 + d0 * 2 * 1 * 4 = 8 * d0 | (v38[8]&0x80) 在 QNX 是 0

    v25 = 4 + 1; // 5

    // if ( v23 != v24 + v25 * ~4 - 9 ) 
    // if ( v47 != v24 + 5 * ~4 - 9 )  // 5 * ~4 - 9 = 5*(-5) - 9 = -34

    if ( v47 != v24 - 34 )  // 5 * ~4 - 9 = 5*(-5) - 9 = -34

      v48 = 0;              // v47 应为 8*d0 - 34

                            // 前文已知 v47 = 24 + d9

                            // 24 + d9 = 8*d0 - 34 -> 8*d0 - d9 = 58

                            // 又因为 d0,d9 范围均为 0 ~ 15 | 暴力求解得 (d0,d9)∈{(8,6), (9,14)}
```

### 2.8 末尾字符串检查

```
    // 末 4 位 'a'/'s' 
    v50 = 123;
    v26 = 2 * 4;
    if ( s[v26 + 2] == 97 ) // s[10]: 'a' +45 / 's' -45
      v50 += 45;
    v27 = 2 * 4;
    if ( s[v27 + 2] == 115 )
      v50 -= 45;
    v28 = 2 * 4;
    if ( s[v28 + 3] == 97 ) // s[11]: 'a' +67 / 's' -67
      v50 += 67;
    v29 = 2 * 4;
    if ( s[v29 + 3] == 115 )
      v50 -= 67;
    v30 = 2 * 4;
    if ( s[v30 + 4] == 97 ) // s[12]: 'a'  +8 / 's'  -8
      v50 += 8;
    v31 = 2 * 4;
    if ( s[v31 + 4] == 115 )
      v50 -= 8;
    v32 = 3 * 4;
    if ( s[v32 + 1] == 97 ) // s[13]: 'a'  +9 / 's'  -9
      v50 += 9;
    v33 = 3 * 4;
    if ( s[v33 + 1] == 115 )
      v50 -= 9;
    v34 = 4;
    v35 = 1;
    if ( 20 * (v34 + v35) != v50 ) // 100 != v50 -> v50 应为 100
                                   // 100 = 123 + 45 - 67 + 8 -9 
                                   // 即 s[10]='a', s[11]='s', s[12]='a', s[13]='s'
        v48 = 0;
```

### 2.9 长度和检查成功flag检验

```
    if ( v48 == 1 && strlen(s)  key 结构为 10 个有效字符 + "asas"
      puts("ok");

    else

      puts("no");
```

## 3. 求解

用 `Python` 写出求解器, 符合要求的解一共有 470 个（我勒个多解大王
