
# [CTF] `qiling` framework + `qilingLab` `x86_64` 题解

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1826353
- 抓取时间: 2026-11-01T22:47:38

## `qiling` framework + `qilingLab` `x86_64` 题解

### 环境

- mac m1
- `Qiling` framework `1.47dev0`
- git 地址: [https://github.com/qilingframework/qiling](https://github.com/qilingframework/qiling)
- 文档地址: [https://docs.qiling.io](https://docs.qiling.io)
- `ida` `7.5`
- `qilingLab`
- `x86_64` 版本: [https://www.shielder.com/attachments/qilinglab-x86_64](https://www.shielder.com/attachments/qilinglab-x86_64)
- `aarch64` 版本: [https://www.shielder.com/attachments/qilinglab-aarch64](https://www.shielder.com/attachments/qilinglab-aarch64)

#### 先模拟运行

```python
ql = Qiling(["qilinglab-x86_64"], "rootfs/x8664_linux",
            verbose=QL_VERBOSE.OFF, multithread=True)
ql.run()
```

- 运行后发现报错。
- `unicorn.unicorn.UcError: Invalid memory read (UC_ERR_READ_UNMAPPED)`
- 说明内存读取不到。不用慌，这是因为第一题我们还没解出来。解出来就解决了。原因下面会讲。

### 第1题

先看下 `ida` 反编译的伪 C 代码。

![](https://cdn.db-kj.com/markdown/image-20230828014725142.png)

比较简单，就是要地址 `0x1337` 的值等于 `1337` 即可。但是我们之前一运行就报错，是因为这里的 `0x1337` 不存在，所以报错。

接下来我们直接写代码：

```python
def challenge1():
    ql.mem.map(0x1000, 0x1000)
    ql.mem.write(0x1337, ql.pack16(1337))
```

### 第2题

![](https://cdn.db-kj.com/markdown/image-20230828020206027.png)

逻辑解释：

- 调用 `uname` 函数赋值给 `name` 变量。
- `name.sysname` 要等于 `"QilingOS"`。
- `name.version` 要等于 `"ChallengeStart"`。
- 所以我们只需要直接修改 `uname` 函数即可。

```python
def challenge2():
    def my_syscall_uname(q: Qiling, *args):
        rdi = ql.arch.regs.rdi
        ql.mem.write(rdi, b'QilingOS\x00')
        ql.mem.write(rdi + 65 * 3, b'ChallengeStart\x00')

    ql.os.set_api("uname", my_syscall_uname, QL_INTERCEPT.EXIT)
```

可能各位会疑问，为什么写 `rdi` 和 `rdi + 65 * 3`，大家看张图即可明白了。

![](https://cdn.db-kj.com/markdown/image-20230828021355516.png)

### 第3题

![](https://cdn.db-kj.com/markdown/image-20230828021512165.png)

逻辑解释：

- 打开 `/dev/urandom`。
- 取 `0x20` 个字符放到 `buf`。
- 取 `1` 个字符放到 `v5`。
- 生成一个 32 位的字符放到 `v7`。
- 对比 `buf` 的每个字符等于 `v7` 的每个字符，且 `buf` 的字符不等于 `v5`。
- 所以解题方式就是做一个假的 `/dev/urandom` 和一个我们自己的 `getrandom`。

```python
def challenge3():
    class FKUrandom(QlFsMappedObject):
        def read(self, expected_len):
            if expected_len > 1:
                return b"\x65" * expected_len
            else:
                return b"\x00"

        def fstat(self):
            return -1

        def close(self):
            return 0

    def my_get_random(q: Qiling, *args):
        p = q.os.resolve_fcall_params({"buf": POINTER, "count": SIZE_T, "flags": INT})
        if p["count"] == 32:
            q.mem.write(p["buf"], b"\x65" * 32)
        return 0

    ql.add_fs_mapper("/dev/urandom", FKUrandom())
    ql.os.set_api("getrandom", my_get_random, QL_INTERCEPT.CALL)
```

### 第4题

该题反编译看不到伪 C 代码，所以我们直接看汇编码。

![](https://cdn.db-kj.com/markdown/image-20230828022258784.png)

![](https://cdn.db-kj.com/markdown/image-20230828022514993.png)

逻辑解释：

- 初始化 `[rbp+var_8] = 0` 和 `[rbp+var_4] = 0`。
- `0xe43` 处进行 `cmp` 判断，如果相等则死循环。
- 所以我们只需要让他们两的值不同即可。

两种解法：

第一种：直接在 `0xe43` 处修改 `eax` 的值。

```python
def challenge4():
    base_ptr = ql.mem.get_lib_base("qilinglab-x86_64")
    hook_addr = base_ptr + 0xe43

    def update_eax(q: Qiling, *args):
        q.arch.regs.eax = 1

    ql.hook_address(update_eax, hook_addr)
```

第二种：直接在赋值后进行修改，让其不同。

```python
def challenge4_1():
    base_ptr = ql.mem.get_lib_base("qilinglab-x86_64")
    hook_addr = base_ptr + 0xe33

    def update_var8(q: Qiling, *args):
        rbp = q.arch.regs.rbp
        q.mem.write(rbp - 0x8, b"\x01")

    ql.hook_address(update_var8, hook_addr)
```

### 第5题

![](https://cdn.db-kj.com/markdown/image-20230828023259185.png)

逻辑解释：

- `v5[0-4] = 0`
- `V5[8-12]=rand()` 随机数
- 所以解题方式就是直接改 `rand` 让其返回 `0` 即可。

```python
def challenge5():
    def my_rand(q: Qiling, *args):
        q.arch.regs.eax = 0

    ql.os.set_api("rand", my_rand, QL_INTERCEPT.EXIT)
```

### 第6题

![](https://cdn.db-kj.com/markdown/image-20230828024829697.png)

逻辑解释，直接一个死循环……参考第4题解题思路。

```python
def challenge6():
    base_ptr = ql.mem.get_lib_base("qilinglab-x86_64")
    hook_addr = base_ptr + 0xf16

    def update_rax(q: Qiling, *args):
        q.arch.regs.rax = 0

    ql.hook_address(update_rax, hook_addr)
```

### 第7题

![](https://cdn.db-kj.com/markdown/image-20230828025012040.png)

逻辑解释，直接睡死……

解题思路，修改 `sleep` 即可。

```python
def challenge7():
    def my_sleep(q: Qiling, *args):
        q.arch.regs.edi = 0
        return 0

    ql.os.set_api("sleep", my_sleep, QL_INTERCEPT.ENTER)
```

### 第8题

![](https://cdn.db-kj.com/markdown/image-20230828025151930.png)

逻辑解释：

- `v2` 申请 `0x18` 大小的字节，就是 `24` 个字节。
- 而 `dword = 4` 字节，所以 `v2` 是 `6` 个 `dword`。
- `qword = 8` 个字节。
- 为 `v2` 起始位置指向的地址申请 `0x1e` 的空间，这里相当于 `v2[0] + v2[1] = malloc(0x1euLL)`。
- `v2[2]` 和 `v2[3]` 分别等于 `1337` 和 `1039980266`。
- 所以 `v2` 的第二个指向地址的值是 `1039980266`，为 `v2` 的起始位置指向的地址赋值成 `"Random data"`。
- `v2+2` 相当于 `v2` 的第三个指向的地址的值 `= a1`，这个就是 `check` 值。
- 所以解题方式就是找到这个结构，然后修改第三个值为 `1` 即可。

```python
def challenge8():
    def hook(q: Qiling, *args):
        magic = (1039980266
```

### 第9题

![](https://cdn.db-kj.com/markdown/image-20230828031401868.png)

逻辑解释：

- 对 `src` 赋值 `"aBcdeFghiJKlMnopqRstuVWxYz"`。
- `dest = src`，即 `dest = "aBcdeFghiJKlMnopqRstuVWxYz"`。
- 循环对 `dest` 做小写处理。
- 判断 `src` 要等于 `dest`。

解题思路：

- 所以我们只需要处理 `strcmp` 或者 `tolower` 函数即可。
- 建议不要处理 `strcmp`，因为第10题也用到了这个，不利于我们学习，会直接把第10题也过了。
- 所以我们处理 `tolower`。

```python
def challenge9():
    def hook(q: Qiling, *args):
        p = q.os.resolve_fcall_params({"s": BYTE})
        # print(p["s"])
        q.arch.regs.eax = p["s"]

    ql.os.set_api("tolower", hook, QL_INTERCEPT.EXIT)
```

### 第10题

![](https://cdn.db-kj.com/markdown/image-20230828031752756.png)

逻辑解释：

- 打开文件 `/proc/self/cmdline`。
- 读取 `0x3f` 字节，赋值给 `buf`。
- 对 `buf` 进行处理，`0` 值赋值成 `32`。
- 判断 `buf` 是否等于 `"qilinglab"`。
- 所以解题思路就是，直接模拟该文件，让 `read` 函数直接读到 `"qilinglab"` 即可。

```python
def challenge10():
    class FSHook(QlFsMappedObject):
        def read(self, expected_len):
            return "qilinglab".encode()

        def close(self):
            return 0

    ql.add_fs_mapper("/proc/self/cmdline", FSHook())
```

### 第11题

![](https://cdn.db-kj.com/markdown/image-20230828032049842.png)

逻辑解释：

- 执行汇编指令 `cpuid`。
- 然后判断 `rbx+rcx` 组合后的值要等于 `0x696C6951614C676ELL`。
- 同时 `rdx` 的值要等于 `538976354`。
- 这题比较关键的是 `cpuid`，不清楚的小伙伴可以自行查询资料。
- 所以我们的解题思路就是在执行 `cpuid` 的时候直接不执行，同时对 `rbc`、`rcx`、`rdx` 进行赋值即可。

```python
def challenge11():
    def hook1(q: Qiling, address, size):
        if ql.mem.read(address, size) == b'\x0f\xa2':
            regs = ql.arch.regs
            regs.ebx = 0x696C6951
            regs.ecx = 0x614C676E
            regs.edx = 0x20202062
            regs.rip += 2

    ql.hook_code(hook1)
```

### 到此为止，11 题全部拿下。受益匪浅，今后准备在有空的情况下，尝试用 `qiling` 框架刷一下 CTF 的逆向题，然后分享给大家一起学习 `qiling` 这个框架。该框架真的非常好。

### 其中第 8 题和第 11 题确实学到了很多的东西……
