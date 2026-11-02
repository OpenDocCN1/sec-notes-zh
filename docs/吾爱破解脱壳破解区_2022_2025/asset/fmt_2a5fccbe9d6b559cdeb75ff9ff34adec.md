
## RSA 流程分析篇

软件没有使用专门的 RSA 库，而是直接对大数进行运算。

简单介绍一下 RSA 算法的过程：

- RSA 为非对称加密，即私钥加密的数据只能使用公钥解密，反之亦然。
- 公、私钥由两个特别大的质数衍生而来，其中公钥由 `e`、`n` 组成，而私钥由 `d`、`n` 组成。
- 当公、私钥的位数足够大，在有限时间内根据其中一个密钥推导出另外一方密钥几乎不可能。
- 加密使用 `C = M ^ e MOD n` 计算，解密使用 `M = C ^ d MOD n` 计算。其中 `M` 与 `C` 分别为明文、密文。

知道了这一点后就可以开始理解原始的代码了：

首先获得一对密钥，对机器码解码为大数，进行 RSA 加密操作得到一个大数；编码该大数，拼接 `ACHECKER` 字样，再解码为一个大数：

```
zhsread("D0330A59", &rsa_E);
zhsread("A7456C12309EAF6BEF610A5B1F408D62B4AF7775E167656C236BC3B8D77F587E92D80DB14AC83281", &rsa_N);
sprintf(buf, "%08X", g_machine_code);
DecodeBase64ToBigInt(buf, &rsa_M);
zexpmod((int)rsa_M, (int *)rsa_E, (int)rsa_N, (int *)&EncryptedMachineCode);
EncodeString(buf, EncryptedMachineCode);
strcat(buf, "ACHECKER");
DecodeBase64ToBigInt(buf, &EncryptedMachineCode);
```

然后对输入的注册码进行相反的操作，将注册码解码回大数，然后进行 RSA 解密操作：

```
// 0075439C  43 39 38 35 46 39 37 41 33 43 34 45 30 44 33 42  C985F97A3C4E0D3B  
// 007543AC  46 37 44 33 35 44 43 34 31 34 38 45 35 43 34 37  F7D35DC4148E5C47  
// 007543BC  37 34 39 30 37 36 44 36 36 38 43 41 38 34 36 34  749076D668CA8464  
// 007543CC  41 36 44 32 43 43 46 42 31 42 32 36 31 38 33 36  A6D2CCFB1B261836  
// 007543DC  32 33 33 31 35 45 35 34 35 30 36 31 30 37 38 34  23315E5450610784  
// 007543EC  44 39 37 34 45 44 35 45 39 37 30 32 41 34 35 31  D974ED5E9702A451  

zhsread(byte_75439C, &rsa_E);                 // 0x75439C = &g_exe_mem[0x80]
DecodeBase64ToBigInt(g_reg_code, &rsa_M); // 变量共用，此处的 rsa_M 应理解为 rsa_C。下同。
zexpmod((int)rsa_M, (int *)rsa_E, (int)rsa_N, (int *)&DecryptedValue);
EncodeString(buf, DecryptedValue);
```

其中 `byte_75439C` 的内容通过动态调试获得，为可执行文件 `0x80` 偏移处开始的内容。

最后对比两边运算得到的数值是否一致：

```
serial_ok = zcompare(EncryptedMachineCode, DecryptedValue) == 0;
```

如果只是为了爆破，此时修改 `zcompare(...) == 0` 的条件为 `!= 0` 即可。

## Python 算法注册机篇

既然搞明白如何运作，那么可以直接使用 Python 将上述代码进行模拟了：

```
def from_machine_code(mc: str, edition: str = 'ACHECKER') -> int:
    rsa_e = 0xD0330A59
    rsa_n = 0xA7456C12309EAF6BEF610A5B1F408D62B4AF7775E167656C236BC3B8D77F587E92D80DB14AC83281
    bi_machine_code = decode_bi(mc)
    v6 = pow(bi_machine_code, rsa_e, rsa_n)
    buf1 = encode_bi(v6) + edition
    return decode_bi(buf1)

def decode_serial_code(serial: str) -> int:
    e = 0xD0330A59
    n = 0xC985F97A3C4E0D3BF7D35DC4148E5C47749076D668CA8464A6D2CCFB1B26183623315E5450610784D974ED5E9702A451
    bi_serial = decode_bi(serial)
    return pow(bi_serial, e, n)
```

但是此时我们遇到了一个问题：从机器码加密数据这一步还算简单，但要生成一个能正常解码的序列号却不容易。

那么就只能把公钥里的 `N` 给替换掉了。因为已知该数据来自可执行文件头，随便算一个进行替换即可。

从网上抄了一个密钥对生成的代码修改而成，详细请参见附件；最终生成的密钥对：

```
found pair of prime number:
p = 0x843639E2D71CC0ECE759D7A98E69262D9ACF5C8248FE46B9
q = 0xAAEAD7348A1D8C131206CE282598DCF7FD5013093C06B151
derived keys:
e = 0xD0330A59
d = 0x446D2FDBB8D1256A48CCFA14080E07D9340AAC319A6ADFBAE38D2AC9963618CABB819B2B2ACB77CEA0086027F3404E9
n = 0x58454B2639277AD2D9B0D34C190725824FB7F154308F80B56287FF315F671FF9EF1EB76184FC16DB6DBBDFBAEBB04989
```

将新的 `N` 带入脱壳前的可执行文件后，就可以正常算号了：

```
$ python ./sxtq/generate_serial.py 8B4BB9C0
ACHECKER --> Lsf10IlAOFtvm6giwVXsBh4ayfuDuV3wgGPSGCIX2UWrflt$BX0AJIg0zCQHo0El
```

直接将生成的序列号拿去测试，成功激活；可以设定电脑的思考时间。

![](https://attach.52pojie.cn/forum/202207/07/063847zow9yso5opotby5o.png)

[reg_ok_proof.png (23.38 KB, 下载次数: 232)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTEzNXwxNDU4YTEyZXwxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:38 上传

## 固定机器码

在分析算法阶段已经得知一个全局变量 `g_machine_code`，直接查找交叉引用定位到来源。

因为是一个 `DWORD` 数值类型，使其每次都计算出一致的数值即可。

```
g_machine_code = 46091 * VolumeSerialNumber;
GenerateAlternativeMachineCode(); // 00401BC0
if ( g_alt_machine_code ) // 若是找到“更好”的替代机器码，则使用该值。
    g_machine_code = g_alt_machine_code;
```

其中一个方案为直接更改“次选”机器码计算过程：

![](https://attach.52pojie.cn/forum/202207/07/063912sfqvo311vw717dqf.png)

[alt_machine_code_gen.png (28.03 KB, 下载次数: 227)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTE0NXxkYTIwY2M0YnwxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:39 上传

直接将 `00401C57` 处的 `mov ecx, ebx` 替换为 `mov ecx, edx`即可无视读取到的特征，从而固定机器码。

若需要固定为特定一个值，可以直接暴力修改赋值过程为 `mov dword[g_alt_machine_code], 0x12345678`。

## 软件自校验

该软件分别实现了文件大小、文件头、以及运行时代码区段的内存校验。

### 文件校验

文件校验非常容易发现，直接对 `CreateFileW` 下断点，等待堆栈出现当前可执行文件路径为止：

![](https://attach.52pojie.cn/forum/202207/07/063906uuvot6ee1etuofo7.png)

[create_file_breakpoint.png (5.64 KB, 下载次数: 234)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTE0M3xhMzI3OWE2Y3wxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:39 上传

然后根据调用堆栈，一路回溯到验证文件头处：

```
00401B1D    call  _fopen
```

该代码差不多类似于这样：

```
f_exe = fopen(g_exe_path, "rb");

if ( f_exe ) {
  DWORD size_of_buffer = fread(g_exe_mem, sizeof(char), 0x80000, f_exe); // 读取前 8k 内容（包括公钥所在处）
  fclose(f_exe);

  // 0x20 处的数值应等于读入数据量的 13 倍
  if ( 13 * size_of_buffer != *(DWORD *)&g_exe_mem[0x20] )
    g_file_tampered = 1;

  // 从缓冲区 0x80 处开始读取，直到结束
  DWORD hash = 0;
  DWORD* p_data = (DWORD*)&g_exe_mem[0x80];
  for (int i = 0; i 
```

Python 实现请参见附件，可以传入一个文件来计算应有的校验值。

当逆向到此处的时候，发现该校验非常脆弱。因此生成 RSA 密钥对的脚本会针对该特性来生成校验值一致的 `N` 值。

### 内存校验

该程序还有内存校验。具体的表现为… 暗桩触发后，AI 会变得异常迟钝。

因为我们更改了程序的字节码，所以直接对更改的地方下一个硬件访问断点即可；

在转储窗口跳转到该代码地址，右键更改处，选择「断点」→「硬件访问断点」：

![](https://attach.52pojie.cn/forum/202207/07/063859aojyb2t018yotco8.png)

[find_what_access_our_code.png (31.63 KB, 下载次数: 235)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTE0MHxiZjhiZDY0M3wxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:38 上传

回到游戏继续操作，待断点触发后将该地址复制到 IDA 继续观察：

```
int ValidateMemory() {

  int hash = 0;

  for ( byte* p = (byte *)ValidateExecutableHeader; p 
```

和刚才一样朴实无华的 `XOR` 校验；在偏移是 `4` 的倍数的位置 `xor 1` 即可让校验码不变：

![](https://attach.52pojie.cn/forum/202207/07/063857sv72nlccbt40ckz0.png)

[fix_checksum_2008_memory.png (33.44 KB, 下载次数: 240)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTEzOXxiMjA1MGE3NHwxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:38 上传

## 带壳补丁固定机器码

没什么好说的，在 UPX 解压完成后直接照着上述的思路更改对应内存区块即可。

![](https://attach.52pojie.cn/forum/202207/07/063849n2iim40i4b4sv0z3.png)

[patch_sxtq_loader_fixed_mc.png (18.55 KB, 下载次数: 237)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTEzNnw2NzQ2YTY0MnwxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:38 上传

需要注意此处能利用到的字节数有限。

如果需要更改大量数据可尝试插入加载 DLL 的代码后在 DLL 进行修改。

## 子跳专版（2016）

目标和之前分析的版本的一样，即搞懂软件的注册算法并编写算法注册机，同时避免触发校验错误暗桩。

### 试探软件注册

![](https://attach.52pojie.cn/forum/202207/07/063839u4r31rq9fnzf4f93.png)

[sxtq2_regform.png (7.42 KB, 下载次数: 233)](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjUzNTEzMnwxZTc2ZDFiN3wxNzkzNTc0Nzc4fDE2ODA4NnwxNjU4NDMx&nothumb=yes)

2022-7-7 06:38 上传

与上作相比，多了一个「收官训练模块」注册码。从这界面上来看，盲猜一手注册部分的代码基本没大改。

### 算法初探

直接脱壳，然后查找字符串引用，发现非常熟悉的 RSA 参数：

```
0041C290  push sxtq_dump_scy.795788        "A7456C1..."
0041C2CD  push sxtq_dump_scy.795808        "C985F97..."
```

把地址放到 IDA 里分析一番，然后交叉引用看看都拿来干什么了：

```
Dir Address                                   Text

Up  Serial__ValidateTiaoQiVersion+C3          push serial_public_n; "C985F97A3C"...

Up  feat1_validate_checksum+505               push serial_public_n; "C985F97A3C"...

Up  feat1_validate_checksum+511               push serial_public_n; "C985F97A3C"...
```
