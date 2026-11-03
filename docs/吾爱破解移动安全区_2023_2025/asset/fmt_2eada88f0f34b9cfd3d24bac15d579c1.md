
```python
        out.append(ch ^ S[(S[i] + S[j]) % 256])
    return out

def RC4decrypt(ciphertext, key):
    return RC4(ciphertext, key)

wrap_elf_start = 0x1e270
wrap_elf_size = 0xb8010
key = b"vUV4#\x91#SVt"

with open('C:/Users/19817/Desktop/libjiagu_a64.so', 'rb') as f:
    wrap_elf = f.read()

# 对密文进行解密
dec_compress_elf = RC4decrypt(wrap_elf[wrap_elf_start:wrap_elf_start + wrap_elf_size], key)
dec_elf = zlib.decompress(bytes(dec_compress_elf[4::]))

with open('wrap_elf', 'wb') as f:
    f.write(dec_elf)

with open('wrap_elf', 'rb') as f:
    wrap_elf = f.read()

ELF_magic = bytes([0x7F, 0x45, 0x4C, 0x46])
for i in range(len(wrap_elf) - len(ELF_magic) + 1):
    if wrap_elf[i:i + len(ELF_magic)] == ELF_magic:
        print(hex(i))
        with open('wrap_elf_part1', 'wb') as f:
            f.write(wrap_elf[0:i])
        with open('wrap_elf_part2', 'wb') as f:
            f.write(wrap_elf[i::])
        break
```

得到了另外一个 `ELF` 文件

![](https://attach.52pojie.cn/forum/202412/30/134329jj6y2gtgcygtp24y.png)

![](https://attach.52pojie.cn/forum/202412/30/134345upr2vrqrdrp22yad.png)

这里的条目的大小和数量和之前一样，那么我们再看看有没有类似的数据，说不定能找到相关的大小。

跟着调用链，我们找到了 `sub_5B08`，这里有 `0x38`，以及传入参数的 `6`。

![](https://attach.52pojie.cn/forum/202412/30/134347m1bwfznvolzlcfcz.png)

这个循环的异或确实是很可疑的，我们只能去 `inlineHOOK` 一下对应的数据了。

```javascript
function inline_HOOK() {
    console.log("[inline_HOOK]");

    // 获取模块和目标地址
    var module = Process.getModuleByName("libjiagu_64.so");
    var soaddr = module.base.add(0x5B34);  // 假设偏移地址是 0x5B34

    // 修改内存权限为只读，以确保能够读取该地址
    Memory.protect(ptr(module.base.add(0x5B34)), 0x100, 'r');

    // 附加钩子
    Interceptor.attach(soaddr, {
        onEnter: function(args) {
            console.log("HOOK STARTED");
            // 读取 ARM 架构的寄存器值（比如 x23）
            var x23 = this.context.x23;  // 读取 x23 寄存器值
            console.log("x23寄存器值: " + x23);
            console.log(hexdump(ptr(x23),{
                offset: 0, // 相对偏移
                length:256,
                header: true, // 显示头部信息
                ansi: true // 使用 ANSI 颜色
            }));
        },
        onLeave: function(retval) {

            // 可以在这里处理函数返回值等操作
        }
    });
}
```

这里的代码都要在 `libjiagu_64.so` 加载之后，并且进行延时 `dump` 才能触发类似于以下这种：

```javascript
if(path.indexOf("libjiagu_64") !== -1) {
    this.is_can_hook = true;
    console.log("hook hook_proc_self_maps started ......");
    // hook_uncompress_res();
    setTimeout(function() {
        // hook_5F20();
        // hook_6044();
        // hook_uncompress_res();
        // HOOKuncompree();
        inline_HOOK();
    }, 1);
}
```

![](https://attach.52pojie.cn/forum/202412/30/140922kffppot8fea3fept.png)

这里 `dump` 的数据和我们之前分割出来的不是 `ELF` 文件那部分是一模一样的。

![](https://attach.52pojie.cn/forum/202412/30/140924gppevpswze93pdep.png)

### 四个数据块

同时对于函数链中的 `sub_5B08` 进行分析，`wrap_elf_part1` 的读取方式是第一个字节表示被异或的数字，这里是 `0xD3`，后面的四个字节表示一个段的长度，随后读取指定长度的字节并异或，之后再读取四个字节获取到下一个段的长度，以此类推，直到读取到文件末尾。这里原作者是已经把这里的异或过程给逆向出来了的，具体的已经涉及到了向量之类的了。

![](https://attach.52pojie.cn/forum/202412/30/140925qksuoz4ylok6l6ox.png)

同时这个函数的最后会有一个数据的相加：

```c
a1[19] = v31 + v19 + v43 + v7 + 17;
```

我们随便找一个变量进行分析，比如 `v19`。

![](https://attach.52pojie.cn/forum/202412/30/140931afx7fh7gbzgbrmpj.png)

`v19` -> `v20` -> `v6`    `v19` 是由 `v6` 赋值而来的，那么 `v6` 是什么？

![](https://attach.52pojie.cn/forum/202412/30/140933aqb8kt4aqw0ak6xa.png)

`v6` 是之前我们 `inlineHOOK` 的 `v4` 的下一个 `byte`，同时还是四字节的。

![](https://attach.52pojie.cn/forum/202412/30/140935tsoggx8qg4xsp9z5.png)

那就是对应的一个数据块的大小。

```c
a1[19] = v31 + v19 + v43 + v7 + 17;
```

那么就是说明这里有四个数据块，而 `17` == （四个对应数据块的 `addr`）`4 * 4 + 1`（异或的 `D3`）。

### 结构体编写

再根据相应的位置，可以创建一个相应的结构体。这里的 `a1` 是 `QWORD` 的指针是八字节的，数组前面的 `0-8` 都被使用了所有要填充 `9 * 8 = 72` 字节，相应的后面也要填充 `4 * 8 = 32` 字节。

```c
struct deal_extra
{
  char blank[72];
  int phnum;
  int *extra_part1;
  int phdr_size;
  char blank2[32];
  int *extra_part2;
  int *extra_part3;
  int *extra_part4;
  int *main_elf;
};
```

所以这里作者应该是写错了，作者写的 36 字节的 `char`，不过是差了 4 字节，不是八字节就无所谓。这里会自动补的。

同理，回溯到之前调用这里的函数也要去补齐对应的结构体，于是到了 `sub_49F0` 的位置。

![](https://attach.52pojie.cn/forum/202412/30/140937a25ks5svozot35el.png)

看着这里把对应的数据块的指针传递出去了，那么就要去给 `v7` 创建一个结构体，看看他怎么个事。

注意这里是 `char` 指针，所以这里是 `29 * 8 = 232` 字节的空。

```c
struct deal_extra_B
{
  char blank[232];
  int *extra_part1;
  char blank1[8];
  int phnum;
  int *extra_part4;
  char blank2[24];
  int *extra_part2;
  char blank3[8];
  int *extra_part3;
};
```

这里已经被 `v7` 的结构体给上去了，而这里的函数，正是在最初判断是自定义 `Linker` 加固 so 的调用的地方，而且传入的参数正是 `v7`，于是我们来看看自定义 `Linker` 的位置。

![](https://attach.52pojie.cn/forum/202412/30/140941dxhkjkzqs1hh8jej.png)

可以看到的是 `v9` 作为 `switch` 的条件判断，是 `v7` 的 `extra_part4` 也就是第四个数据块的数据传入的，是用于处理动态链接库的，那么通过源码比对，`extra_part4` 对应 `Section Header Table`（节头表）的 `.dynamic` 段。

之后再来看看自定义 `Linker` 的处理动态链接库的下面的函数 `sub_4918`。

![](https://attach.52pojie.cn/forum/202412/30/134416rrbs3kklkkk6krvs.png)

这里去处理了 `extra_part0` 到 `extra_part3` 这三个数据块的数据，我们再去看看，这三个数据块是干嘛的。

这里先普及一下重定向的东西：

### 重定位类型（R_*）

在重定位操作中，数字如 `0x401`、`0x408` 可能也与重定位类型（如 `R_386_32`、`R_X86_64_64` 等）相关，这些类型定义了如何修改程序中指定地址的值。具体的重定位类型由操作系统和体系结构决定。

### 举例：典型的 .rela.plt 条目结构

假设 `.rela.plt` 节包含重定位条目，条目可能如下所示：

```text
Offset      Type      Symbol    Addend
0x401       R_X86_64_PC32  foo     0x0
0x101       R_X86_64_64    bar     0x0
0x408       R_X86_64_PC32  baz     0x0
```

而在 Android 的源码动态的重定向的定义下

![](https://attach.52pojie.cn/forum/202412/30/134418dc4cci329cx92k2a.png)

![](https://attach.52pojie.cn/forum/202412/30/134421t7pqz760i70rnjrp.png)

其实是有出现类似的字眼出现的，但是能够直接确定是重定向中的哪个节，还是有点难道的。

“这个函数中的 `switch` 是用来处理重定位的，因为重定位主要有基址重定位和符号重定位，这两个的值分别是 `0x403` 和 `0x402`，所以 `extra_part2` 和 `extra_part3` 分别对应着 `.rela.plt`（`0x402` 重定位）和 `.rela.dyn`（`0x403` 重定位）。”

同时再来看看 `sub_5E6C()`，也就是我们最初找到的条目大小和 `num` 的位置，这里也可以重新附上之前创建的结构体了。

![](https://attach.52pojie.cn/forum/202412/30/134423wqe4o2k0sxzk9gmx.png)

这里确实就可以知道 `a1` 也就是 `extra_part1` 代表的就是程序头 `program header table` 的位置了。

- 数据块1表示 `program header table`
- 数据块2表示 `.rela.plt`
- 数据块3表示 `.rela.dyn`
- 数据块4表示 `.dynamic`

### 分割四个数据块

```python
import copy
import zlib

def RC4(data, key):
    S = list(range(256))
    j = 0
    out = []

    # KSA Phase
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]

    # PRGA Phase
    i = j = 0
    for ch in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(ch ^ S[(S[i] + S[j]) % 256])
    return out

def RC4decrypt(ciphertext, key):
    return RC4(ciphertext, key)

wrap_elf_start = 0x1e270
wrap_elf_size = 0xb8010
key = b"vUV4#\x91#SVt"

with open('com.oacia.apk_protect/assets/libjiagu_a64.so', 'rb') as f:
    wrap_elf = f.read()

# 对密文进行解密
dec_compress_elf = RC4decrypt(wrap_elf[wrap_elf_start:wrap_elf_start + wrap_elf_size], key)
dec_elf = zlib.decompress(bytes(dec_compress_elf[4::]))

with open('wrap_elf', 'wb') as f:
    f.write(dec_elf)

# 以上解密
# 定义结构体
class part:
    def __init__(self):
        self.name = ""
        self.value = b''
        self.offset = 0
        self.size = 0

index = 1
extra_part = [part() for _ in range(7)]
seg = ["phdr", ".rela.plt", ".rela.dyn", ".dynamic"]
v_xor = dec_elf[0]  # 异或的是第一个的0xD3

for i in range(4):
    size = int.from_bytes(dec_elf[index:index + 4], 'little')  # 获取相应位置的四字节的size
    index += 4
    extra_part[i + 1].name = seg[i]  # 赋值属性
    extra_part[i + 1].value = bytes(map(lambda x: x ^ v_xor, dec_elf[index:index + size]))  # 定义匿名函数，只是去异或
    extra_part[i + 1].size = size
    index += size

for p in extra_part:
    if p.value != b'':
        filename = f"libjiagu.so_{hex(p.size)}_{p.name}"
        print(f"[{p.name}] get {filename}, size: {hex(p.size)}")
        with open(filename, 'wb') as f:
            f.write(p.value)
```

通过解密每个数据块，我们可以去实现恢复最初的 `ELF` 文件格式，在这里之后我们就得到了四个解密之后的数据块的数据，同时四个分别代表了的是 `program header table`、`.rela.plt`、`.rela.dyn`、`.dynamic`，到最后我们就要去实现的是位置的替换填充。

#### program header table：选中之后就直接把我们解密的替换上去

![](https://attach.52pojie.cn/forum/202412/30/141303h99m9vygmq2av330.png)
