
# [原创] Navicat 17.3.2 公钥拼接函数纯静态分析及提取

- 原帖：`https://www.52pojie.cn/forum.php?mod=viewthread&tid=2057483`
- 抓取时间：2026-11-01T22:03:38

## 定位公钥拼接函数

参考 [相关帖子](https://www.52pojie.cn/thread-2056733-1-1.html)，不难定位到公钥碎片地址，交叉引用查到公钥拼接函数 `sub_18215B240`。

![image-20250902151542015.png](https://attach.52pojie.cn/forum/202509/02/155647q5zvkosj8ojr15k1.png)
`image-20250902151542015.png` (48.83 KB，下载次数: 4)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM1N3xjODE3MDE4N3wxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

![image-20250902144831113.png](https://attach.52pojie.cn/forum/202509/02/155657mq0x2mcwpfw8p2ql.png)
`image-20250902144831113.png` (15.76 KB，下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM2MXxmOGYzOWU1ZnwxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

公钥拼接函数很大，差不多有 80 KB。IDA 默认会拒绝反编译大小 64 KB 以上的函数，需要调整 `MAX_FUNCSIZE`。

![image-20250902144118453.png](https://attach.52pojie.cn/forum/202509/02/155659pggeee9b9rg22p99.png)
`image-20250902144118453.png` (48.92 KB，下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM2Mnw5ZjkxMTFkYXwxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

## 处理不透明谓词

`F5` 生成的伪代码有一万多行，且经过不透明谓词混淆。搞半天还要自己拼。

![image-20250902145156478.png](https://attach.52pojie.cn/forum/202509/02/155654onn245v5ovvn3jko.png)
`image-20250902145156478.png` (60 KB，下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM2MHw3MGZkMzlhOHwxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

其中 `v1811`、`v1524`、`v1670` 为不透明谓词，由一类相似的函数返回固定的 `bool` 值。

![image-20250902145916367.png](https://attach.52pojie.cn/forum/202509/02/155652paffrfrvo1vrvdih.png)
`image-20250902145916367.png` (13.67 KB，下载次数: 4)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM1OXxlOTQ4MDQ5Y3wxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

不透明谓词求值用到的常量字符串如下：

![image-20250902151454029.png](https://attach.52pojie.cn/forum/202509/02/155650addwpopdemanzage.png)
`image-20250902151454029.png` (61.95 KB，下载次数: 2)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM1OHxkNzI4MzEyNnwxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

`Patch` 掉不透明谓词后程序的结构就很清晰了，只有七百多行。不难看出 Navicat 应该是使用了 C++ 标准库的 `std::string`，适当重命名库函数可以得到以下结果：

![image-20250902152057377.png](https://attach.52pojie.cn/forum/202509/02/155645lg938z3xt5cs9gxy.png)
`image-20250902152057377.png` (31.93 KB，下载次数: 3)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM1Nnw4ZjA5ZGNhOXwxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

## 分析公钥拼接逻辑

不难看出这是 `base64` 编码的 `der` 格式，但是拼接时复用了部分公钥碎片，且拼接过程中存在前向引用公钥进行解密的操作。用正则表达式对伪代码进行适当替换可以得到以下代码：

```python
f = lambda x: x.to_bytes((x.bit_length() + 7) // 8, 'little')
t = bytearray()
t.extend(b'MI')
t.extend(b'IB')
t.extend(b'I')
t.extend(b'jAN')
t.extend(b'B')
t.extend(b'g')
v5 = t[2]
t.append(v5 + 34)
v139 = f(113)
v215 = v139
t.extend(v215)
v157 = b'hki'
v216 = v157
t.extend(v216)
v175 = f(813119815)
v217 = v175
t.extend(v217)
t.extend(b'BAQE')
# ...
t.extend(b'x')
v119 = t[86]
t.append(v119 + 3)
t.extend(b'ju')
v120 = t[381]
t.append(v120)
t.extend(b'Qa')
t.append(119)
v184 = f(1363231817)
v212 = v184
t.extend(v212)
v137 = b'A'
v213 = v137
t.extend(v213)
v123 = b'B'
v214 = v123
t.extend(v214)
print(t.decode())  # MIIBIjANBgkqhkiG9w0BAQE...xljuuQawIDAQAB
```

得到的结果能被完整解析，并可提取出 `N` 和 `e` 等常数。后续 `patch` 及 `keygen` 不在叙述。

## Patch 点位选择

由于公钥中存储的 `N` 前有一段 `header`，导致不好在伪代码中定位 `patch` 点位，故考虑尽可能修改靠近 `LSB` 的半字节，此时将 `Qa` 修改为 `Ta` 为最佳选项。

同时，如果从 `MSB` 出发，前几项结果会出现 `base64` 编码将三字节映射到四字节时半字节 `patch` 仍出现 `base64` 编码后出现两字节不同的问题，从 `LSB` 出发时第一个小质数（`0x1FD`）结果即满足 `base64` 编码后仅有一字节不同。

![image-20250902153821964.png](https://attach.52pojie.cn/forum/202509/02/155643wjour9nfswwfcw9n.png)
`image-20250902153821964.png` (14.3 KB，下载次数: 4)  
[附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjgwMDM1NXw1ZWIxOTE4MXwxNzkzNTcwNjE2fDE2ODA4NnwyMDU3NDgz&nothumb=yes)（2025-9-2 15:56 上传）

## 附录：移除不透明谓词后的伪代码

p.s. 移除不透明谓词后伪代码如下：

```c
__int64 *__fastcall sub_18215B240(__int64 a1, __int64 *result)
{
  __int64 v3; // rax
  __int64 v4; // rax
  char *v5; // rax
  __int64 v6; // rax
  __int64 v7; // rax
  __int64 v8; // rax
  char *v9; // rax
  char *v10; // rax
  char *v11; // rax
  char *v12; // rax
  __int64 v13; // rax
  char *v14; // rax
  char *v15; // rax
  char *v16; // rax
  char *v17; // rax
  char *v18; // rax
  __int64 v19; // rax
  char *v20; // rax
  char *v21; // rax
  char *v22; // rax
  __int64 v23; // rax
  __int64 v24; // rax
  char *v25; // rax
  __int64 v26; // rax
  char *v27; // rax
  char *v28; // rax
  char *v29; // rax
  __int64 v30; // rax
  __int64 v31; // rax
  char *v32; // rax
  __int64 v33; // rax
  __int64 v34; // rax
  char *v35; // rax
  char *v36; // rax
  char *v37; // rax
  __int64 v38; // rax
  char *v39; // rax
  __int64 v40; // rax
  char *v41; // rax
  __int64 v42; // rax
  char *v43; // rax
  __int64 v44; // rax
  __int64 v45; // rax
  char *v46; // rax
  char *v47; // rax
  __int64 v48; // rax
  char *v49; // rax
  char *v50; // rax
  char *v51; // rax
  char *v52; // rax
  char *v53; // rax
  char *v54; // rax
  __int64 v55; // rax
  char *v56; // rax
  char *v57; // rax
  char *v58; // rax
  char *v59; // rax
  __int64 v60; // rax
  char *v61; // rax
  __int64 v62; // rax
  char *v63; // rax
  char *v64; // rax
  char *v65; // rax
  __int64 v66; // rax
  __int64 v67; // rax
  __int64 v68; // rax
  char *v69; // rax
  char *v70; // rax
  char *v71; // rax
  char *v72; // rax
  __int64 v73; // rax
  char *v74; // rax
  __int64 v75; // rax
  __int64 v76; // rax
  __int64 v77; // rax
  __int64 v78; // rax
  char *v79; // rax
  char *v80; // rax
  __int64 v81; // rax
  char *v82; // rax
  __int64 v83; // rax
  char *v84; // rax
  char *v85; // rax
  __int64 v86; // rax
  char *v87; // rax
  char *v88; // rax
  char *v89; // rax
  __int64 v90; // rax
  char *v91; // rax
  char *v92; // rax
  char *v93; // rax
  __int64 v94; // rax
  __int64 v95; // rax
  char *v96; // rax
  __int64 v97; // rax
  char *v98; // rax
  char *v99; // rax
  __int64 v100; // rax
  __int64 v101; // rax
  char *v102; // rax
  char *v103; // rax
  char *v104; // rax
  char *v105; // rax
  char *v106; // rax
  char *v107; // rax
  char *v108; // rax
  char *v109; // rax
  char *v110; // rax
  __int64 v111; // rax
  char *v112; // rax
  char *v113; // rax
  __int64 v114; // rax
  char *v115; // rax
  __int64 v116; // rax
  char *v117; // rax
  __int64 v118; // rax
  char *v119; // rax
  char *v120; // rax
  __int64 v121; // rax
  char v123; // [rsp+37h] [rbp-C9h] BYREF
  char v124; // [rsp+38h] [rbp-C8h] BYREF
  char v125; // [rsp+39h] [rbp-C7h] BYREF
  char v126; // [rsp+3Ah] [rbp-C6h] BYREF
  char v127; // [rsp+3Ch] [rbp-C4h] BYREF
  char v128; // [rsp+3Dh] [rbp-C3h] BYREF
  char v129; // [rsp+3Eh] [rbp-C2h] BYREF
  char v130; // [rsp+3Fh] [rbp-C1h] BYREF
  char v131; // [rsp+40h] [rbp-C0h] BYREF
  char v132; // [rsp+58h] [rbp-A8h] BYREF
  char v133; // [rsp+59h] [rbp-A7h] BYREF
  char v134; // [rsp+5Ah] [rbp-A6h] BYREF
  char v135; // [rsp+5Bh] [rbp-A5h] BYREF
```
