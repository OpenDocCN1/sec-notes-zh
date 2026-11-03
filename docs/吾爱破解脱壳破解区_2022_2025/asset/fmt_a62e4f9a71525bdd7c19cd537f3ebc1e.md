
# 3> forest

## Python 解密脚本

```python
flag = [0, 0x9D, 0x6B, 0xA1, 0x02, 0xD7, 0xED, 0x40, 0xF6, 0x0E, 0xAE, 0x84, 0x19]
for i in range(12, 1, -1):
    flag[i] ^= flag[i-1]

flag[12] ^= 7
flag[12] = (flag[12] > 1)
flag[12] &= 0xFF

flag[11] ^= flag[12]
flag[11] = (flag[11] > 4)
flag[11] &= 0xFF

flag[10] ^= flag[11]
flag[10] = (flag[10] > 4)
flag[10] &= 0xFF

flag[9] ^= flag[10]
flag[9] = (flag[9] > 6)
flag[9] &= 0xFF

flag[8] ^= flag[9]
flag[8] = (flag[8] > 1)
flag[8] &= 0xFF

flag[7] ^= flag[8]
flag[7] ^= 7
flag[7] = (flag[7] > 1)
flag[7] &= 0xFF

flag[6] ^= flag[7]
flag[6] = (flag[6] > 4)
flag[6] &= 0xFF

flag[5] ^= flag[6]
flag[5] ^= 4
flag[5] = (flag[5] > 4)
flag[5] &= 0xFF

flag[4] ^= flag[5]
flag[4] ^= 7
flag[4] = (flag[4] > 1)
flag[4] &= 0xFF

flag[3] ^= flag[4]
flag[3] = (flag[3] > 2)
flag[3] &= 0xFF

flag[2] ^= flag[3]
flag[2] = (flag[2] > 3)
flag[2] &= 0xFF

flag[1] ^= flag[2]
flag[1] ^= 3
flag[1] = (flag[1] > 5)
flag[1] &= 0xFF

# flag[0] ^= flag[1]
print(bytes(flag))
```

解密拿到flagd3ctf{m3owJumpVmvM}

## `main` 与异常处理

`main`里面检查`flag`格式，把输入的17个字符转换称01字符串，末尾加0

重点在`sub_CE1F50`里面，把一段内存设置为可执行，然后触发`int 3`异常，使用`sub_CE1A00`函数（`execption_handle`）处理异常

![](https://attach.52pojie.cn/forum/202404/28/223737rj9j3jo9sjnzn9ts.png)

2024-04-27-23-56-17-image.png (52.6 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MjcwOXxkYjA5ZDI2Y3wxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MjcwOXxkYjA5ZDI2Y3wxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes)

2024-4-28 22:37 上传

```c
unsigned int __thiscall execption_handle(unsigned int **this)

{

  unsigned int result; // eax

  unsigned int *v3; // ecx

  unsigned int *v4; // eax

  _DWORD *v5; // eax

  int v6; // ecx

  unsigned int v7; // edx

  void **v8; // ecx

  int v9; // esi

  int i; // ecx

  int v11; // edx

  int v12; // eax

  int v13; // esi

  int v14; // edx

  void **v15; // eax

  int v16; // [esp+10h] [ebp-2Ch]

  int v17; // [esp+14h] [ebp-28h]

  _DWORD *v18; // [esp+18h] [ebp-24h]

  GetCurrentThreadId();

  result = **this;

  if ( result  v7 )

        {

          v8 = &flag;

          if ( HIDWORD(len) >= 0x10 )

            v8 = (void **)flag;

          v16 = R2;

          v9 = 0;

          *(_DWORD *)R2 = *((_BYTE *)v8 + v7) != 0x30;

          kk = v7 + 1;

          for ( i = 0; i = (unsigned int)len )

  {

    return(1);

    goto LABEL_39;

  }

  v15 = &flag;

  if ( HIDWORD(len) >= 0x10 )

    v15 = (void **)flag;

  *(_DWORD *)R2 = *((_BYTE *)v15 + kk) != 0x30;

  kk = v13 + 1;

  this[1][46] = (unsigned int)&code + 0x40 * v14;

  v4 = this[1];

LABEL_33:

  v4[48] |= 0x100u;

  return -1;

}
```

在`execption_handle`中根据异常码进行处理

```c
// ksarm.h
#define STATUS_ACCESS_VIOLATION 0xc0000005
#define STAUSBREAKPOINT 0x80000003
#define STATUS_SINGLE_STEP 0x80000004
#define STATUS_PRIVILEGED_INSTRUCTION 0xc0000096
```

### 断点异常处理

第一次`int 3`触发断点异常0x80000003，对应下面的异常处理代码

```c
if (byte_CE6028)

{

    byte_CE6028 = 0;

    R2 = (int)malloc(4u);

    R1 = (int)malloc(4u);

    v5 = malloc(4u);

    v6 = R1;

    v7 = kk;

    v18 = v5;

    R0 = (int)v5;

    *(_DWORD *)R1 = 0;

    *v5 = 0;

    v17 = v6;

    if ((unsigned int)len > v7)

    {

        v8 = &flag;

        if (HIDWORD(len) >= 0x10)

            v8 = (void **)flag;

        v16 = R2;

        v9 = 0;

        *(_DWORD *)R2 = *((_BYTE *)v8 + v7) != 0x30;

        kk = v7 + 1;

        for (i = 0; i 
```

这段代码只能执行一次，后续再触发0x80000003异常程序会结束运行，代码里面`malloc`了3个变量，并使用这3个变量替换`code`(前面修改了可执行权限的内存)中的0xFFFFFFFF

`R2`的初值由`*(_DWORD *)R2 = *((_BYTE *)v8 + v7) != 0x30;`决定，这里是`flag`的第一个字符

将`code`初始化之后，`this[1][46] = (unsigned int)&code;`修改线程结构体的`EIP`，使异常处理结束后程序进入到`code`中执行

分析一下`code`部分代码，`code`代码中包含很多的代码块，每一块的长度为0x40，根据前面修改的规律可以知道在`code`块中`R0`、`R1`、`R2`的位置固定

![](https://attach.52pojie.cn/forum/202404/28/223739rx6g7zmljljpr36f.png)

2024-04-28-10-42-40-image.png (45.18 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MjcxMHxlZjk2ZDE2YXwxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MjcxMHxlZjk2ZDE2YXwxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes)

2024-4-28 22:37 上传

### 特权指令异常处理

代码块中还有`cli`指令，由于`cli`指令是特权指令，ring3无法执行，因此运行到`cli`指令会触发0xc0000096异常，分析对应的处理代码

```c
v13 = kk;
v14 = *(_DWORD *)R1 + 17 * *(_DWORD *)R0;
if (kk >= (unsigned int)len)
{
    return (1);
    goto LABEL_39;
}
v15 = &flag;
if (HIDWORD(len) >= 0x10)
    v15 = (void **)flag;
*(_DWORD *)R2 = *((_BYTE *)v15 + kk) != 0x30;
kk = v13 + 1;
this[1][46] = (unsigned int)&code + 0x40 * v14;
v4 = this[1];
v4[48] |= 0x100u;
```

`cli`指令触发异常后，根据`R1+17*R0`的值确定下一个执行的代码块，修改`EIP`进入新的代码块，并且根据`flag`的01字符设置`R2`

结合`code`中的内容，在`code`代码中会设置`R0`、`R1`的值，接着使用`cli`触发异常，根据`R0`、`R1`的值选择下一个代码块，`R2`中储存了`flag`的一个bit

### 单步异常处理

调试发现在`code`中还会触发0x80000004异常

![](https://attach.52pojie.cn/forum/202404/28/223742qae87azr74x7e7la.png)

2024-04-28-10-59-38-image.png (62.27 KB, 下载次数: 2)

[href="forum.php?mod=attachment&aid=MjY5MjcxMXwxZmVjYzVhMHwxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MjcxMXwxZmVjYzVhMHwxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes)

2024-4-28 22:37 上传

![](https://attach.52pojie.cn/forum/202404/28/223744t2ct7htoca9z0y8c.png)

2024-04-28-11-07-03-image.png (26.23 KB, 下载次数: 1)

[href="forum.php?mod=attachment&aid=MjY5MjcxMnwxZjRiZjZlMXwxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MjcxMnwxZjRiZjZlMXwxNzkzNTcyMjY5fDE2ODA4NnwxOTE4Nzg4&nothumb=yes)

2024-4-28 22:37 上传

跟踪一下发现这个异常是`TF`标志位造成的，`TF`标志位为1时，执行一条指令后就会触发0x80000004异常，在`execption_handle`中，返回时会设置`v4[48] |= 0x100u;`，这里设置的就是`TF`标志位

`TF`标志位触发异常后，会确定异常发生的位置，当异常发生位置为7，即`mov eax, [eax]`执行后，根据`eax`寄存器的确定是否进行跳转

```c
if (result != 0x80000004)
    return result;
v3 = this[1];
if (((v3[46] - (_DWORD)&code) & 0x3F) == 7 && v3[44] == 1)
    v3[46] += 0x17;
v4 = this[1];
v4[48] |= 0x100u;
```

结合跳转的位置和`code`中的代码，这里实际是根据`R2`进行调整，如果`R2`，即`flag`为1，跳转到下一个`cli`块中设置`R1`、`R0`，综合分析知道这里是根据`flag`的01值决定下一个代码块，在每一个代码块中有两个可跳转的位置

## `IDA Python` 解析代码块

使用`IDA Python`解析代码块，获取每个代码块中的两个跳转位置

```python
import idc
import idaapi

image_base = idaapi.get_imagebase()
code_off = 0x6030
code_size = 0x4840

start = image_base + code_off
end = image_base + code_off + code_size

op_ls = [(-1, -1) for _ in range(0x121)]
for idx in range(0x121):
    ea = start + idx * 0x40
    print(f"[{hex(ea)}]: ")
    ins = idc.generate_disasm_line(ea, 0)
    if (ins.startswith("mov     eax, 0FFFFFFFFh")):
        op1 = idc.print_operand(ea+0x0C, 1)
        if (op1[-1] == 'h'):
            op1 = int(op1[:-1], 16)
        else:
            op1 = int(op1)
        op2 = idc.print_operand(ea+0x17, 1)
        if (op2[-1] == 'h'):
            op2 = int(op2[:-1], 16)
        else:
            op2 = int(op2)
        op3 = idc.print_operand(ea+0x23, 1)
        if (op3[-1] == 'h'):
            op3 = int(op3[:-1], 16)
        else:
            op3 = int(op3)
        op4 = idc.print_operand(ea+0x2E, 1)
        if (op4[-1] == 'h'):
            op4 = int(op4[:-1], 16)
        else:
            op4 = int(op4)
        ins = (op1+17*op2, op3+17*op4)
    elif (ins.startswith("int     3")):
        ins = (-1, -1)
    print(ins)
    op_ls[idx] = ins

print(op_ls)
```

在解析时发现除了上面正常的代码块，还有下面几种代码块

```asm
"""
int     3
add     [ecx-51h], ebx
mov     eax, 1; mov     dword ptr [eax], 39393939h
add     [ebp+28D5560Ah], ecx
"""
```

其中`int 3`触发0x80000003异常，由于该异常只能处理一次，所有遇到`int 3`的代码块会直接die掉

剩余3条指令会触发0xC0000005异常，该异常会进入`right`，输出正确信息

## 迷宫分析

分析后可以知道本题其实是一道迷宫题，根据输入的`flag`决定走的路径，走到出口胜利

先找一下实际可到达的出口
