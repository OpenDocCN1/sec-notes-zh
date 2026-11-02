
# [CTF] OLLVM 与去平坦化 & [RoarCTF2019] polyre 详细WP

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1601573
- 抓取时间: 2026-11-01T23:33:58

## OLLVM

llvm是一个底层虚拟机，OLLVM（Obfuscator-LLVM）是瑞士西北应用科技大学安全实验室于2010年6月份发起的一个项目，这个项目的目标是提供一个LLVM编译套件的开源分支，能够通过代码混淆和防篡改，增加对逆向工程的难度，提供更高的软件安全性。目前，OLLVM已经支持`LLVM-4.0.1`版本;所使用的编译器是`clang`。
llvm保护的大概方法是：程序主要使用一个主分发器来控制程序基本块的执行流程进而模糊每个模块之间的联系。

## 平坦化程序特征

```
__int64 __fastcall check(__int64 a1)
{
  size_t v1; // rax@13
  signed int v2; // ecx@13
  char v3; // ST04_1@16
  signed int v4; // eax@18
  signed int v5; // eax@21
  signed int v6; // eax@24
  signed int v7; // eax@27
  signed int v9; // [sp+44h] [bp-1Ch]@1
  int v10; // [sp+48h] [bp-18h]@1
  signed int v11; // [sp+5Ch] [bp-4h]@0

  srand(0x64u);
  v10 = 0;
  v9 = 706310565;
  while ( 1 )
  {
    while ( 1 )
    {
      while ( 1 )
      {
        while ( 1 )
        {
          while ( 1 )
          {
            while ( 1 )
            {
              while ( 1 )
              {
                while ( 1 )
                {
                  while ( v9 == -2109444161 )
                  {
                    v7 = 1322041670;
                    if ( *(_BYTE *)(a1 + 3) == 100 )
                      v7 = 867560817;
                    v9 = v7;
                  }
                  if ( v9 != -2069803162 )
                    break;
                  ++v10;
                  v9 = 706310565;
                }
                if ( v9 != 306556692 )
                  break;
                v4 = 1322041670;
                if ( *(_BYTE *)a1 == 97 )
                  v4 = 564228819;
                v9 = v4;
              }
              if ( v9 != 341947172 )
                break;
              v6 = 1322041670;
              if ( *(_BYTE *)(a1 + 2) == 99 )
                v6 = -2109444161;
              v9 = v6;
            }
            if ( v9 != 564228819 )
              break;
            v5 = 1322041670;
            if ( *(_BYTE *)(a1 + 1) == 98 )
              v5 = 341947172;
            v9 = v5;
          }
          if ( v9 != 682671478 )
            break;
          v3 = *(_BYTE *)(a1 + v10);
          *(_BYTE *)(a1 + v10) = rand() % 5 + v3 - 97 + 97;
          v9 = -2069803162;
        }
        if ( v9 != 706310565 )
          break;
        v1 = strlen((const char *)a1);
        v2 = 306556692;
        if ( v10 
```
![](https://attach.52pojie.cn/forum/202203/12/130138zk1fffueoifuffuz.jpg)
1.jpg (113.45 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjUwMTY1OHxiOGUzZDMyNXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1OHxiOGUzZDMyNXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:01 上传

### 尝试进行函数的恢复
- 函数的开始地址为序言的地址
- 序言的后继为主分发器
- 后继为主分发器的块为预处理器
- 后继为预处理器的块为真实块
- 无后继的块为`retn`块
- 剩下的为无用块

### deflat 去平坦化脚本使用方法

#### deflat.py
最早版本 [python2] [//github.com/SnowGirls/deflat" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/SnowGirls/deflat:rel="nofollow noopener" href="https)
修改版 [python3] [//github.com/cq674350529/deflat" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/cq674350529/deflat:rel="nofollow noopener" href="https)

#### 安装 angr 模块
脚本需要 `angr` 模块，官方建议使用创建虚拟环境后使用
![](https://attach.52pojie.cn/forum/202203/12/130004olt30130s91l1eem.jpg)
2.jpg (238.7 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjUwMTY1MXw3ZWY1MGI2MnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1MXw3ZWY1MGI2MnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传
进入虚拟环境后 `pip install angr`
![](https://attach.52pojie.cn/forum/202203/12/130002r3btx2z9pdpzbbtp.jpg)
3.jpg (435.3 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjUwMTY1MHwwNzliNjg0MHwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1MHwwNzliNjg0MHwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传

#### 示例 1
这是脚本自带的 sample `check_passwd_x8664_flat`
![](https://attach.52pojie.cn/forum/202203/12/130010ir92xb0r9a3xa7e2.jpg)
4.jpg (116.39 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjUwMTY1M3w3MmQ3ODM4MnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1M3w3MmQ3ODM4MnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传
拖入 `IDA` 看看，发现是平坦化处理了
![](https://attach.52pojie.cn/forum/202203/12/130013nd00iw3lq3g0q999.jpg)
5.jpg (125.51 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjUwMTY1NHw2MzhiNTI3MXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1NHw2MzhiNTI3MXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传
关键之处在于寻找程序的 address
注意到“函数的开始地址为序言的地址”，因此我们需要在序言中找函数起始位置
![](https://attach.52pojie.cn/forum/202203/12/130016csin6cucz5y9ccn9.jpg)
6.jpg (367.14 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjUwMTY1NXwyYTFhN2FiNXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1NXwyYTFhN2FiNXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传
如上图，地址为 `0x4009A0`
运行脚本 (注意需要把下图两个文件从上层目录复制过来)
`python deflat.py -f path/to/binary --addr hexaddress`
```
python deflat.py -f check_passwd_x8664_flat --addr 0x4009A0
```
![](https://attach.52pojie.cn/forum/202203/12/130141fqkghhgiczbxohxr.jpg)
77.jpg (110.62 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjUwMTY1OXw0YTAwMTRhM3wxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1OXw0YTAwMTRhM3wxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:01 上传
运行结果
![](https://attach.52pojie.cn/forum/202203/12/130021sdnzt3g3ngwzw4zg.jpg)
8.jpg (643.33 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjUwMTY1Nnw4MWNlNGY0M3wxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1Nnw4MWNlNGY0M3wxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传
发现新生成的程序已经成功去平坦化
![](https://attach.52pojie.cn/forum/202203/12/130007xr3v97d9ekm33k3m.jpg)
9.jpg (207.47 KB, 下载次数: 3)
[href="forum.php?mod=attachment&aid=MjUwMTY1MnwxYTc2Njk4OXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1MnwxYTc2Njk4OXwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传

### 示例 2 —— [RoarCTF2019] polyre WP

#### 去平坦化
`IDA` 打开程序
![](https://attach.52pojie.cn/forum/202203/12/130037bjpcfz2j7zlpzp7p.jpg)
10.jpg (315.35 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjUwMTY1N3xkY2IzMzBkY3wxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY1N3xkY2IzMzBkY3wxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:00 上传
找序言的函数起始位置
![](https://attach.52pojie.cn/forum/202203/12/130144t5ofo6u77a5i7mia.jpg)
11.jpg (432.8 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjUwMTY2MHwyOGI3MmQ4YnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY2MHwyOGI3MmQ4YnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:01 上传
运行程序
`python deflat.py -f polyre --addr 0x400620`
运行结果，成功去平坦化
![](https://attach.52pojie.cn/forum/202203/12/130146c8jaar49wx9erkak.jpg)
12.jpg (200.42 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjUwMTY2MXwxMWEwOTI0ZnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUwMTY2MXwxMWEwOTI0ZnwxNzkzNTc2MDM2fDE2ODA4NnwxNjAxNTcz&nothumb=yes)
2022-3-12 13:01 上传

#### 分析代码
