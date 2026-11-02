
# glibc `_IO_FILE` `vtable` 劫持与利用

```
[#0] _IO_flush_all_lockp(do_lock=0x0)
[#1] _IO_cleanup()
[#2] __run_exit_handlers(...)
[#3] __GI_exit(status=)
[#4] __libc_start_main(...)
[#5] _start()
```

## 伪造 `vtable`

glibc 由于位于 libc 数据段的 `vtable` 是不可以进行写入的，所以我们只能伪造 `vtable`。

伪造 `vtable` 劫持程序流程的中心思想就是针对 `_IO_FILE_plus` 的 `vtable` 动手脚，通过把 `vtable` 指向我们控制的内存，并在其中布置函数指针来实现。

### 2.24：`_IO_OVERFLOW` 中的 `vtable` 检测

`_IO_OVERFLOW` 中加入 [`IO_validate_vtable`](https://code.woboq.org/userspace/glibc/libio/libioP.h.html#IO_validate_vtable) 对 `vtable` 做了检测，`vtable` 必须要满足在 `__stop___IO_vtables` 和 `__start___libc_IO_vtables` 之间，而我们伪造的 `vtable` 通常不满足这个条件。

但是我们找到一条出路，那就是 [`_IO_str_jumps`](https://code.woboq.org/userspace/glibc/libio/strops.c.html#_IO_str_jumps) 与 [`_IO_wstr_jumps`](https://code.woboq.org/userspace/glibc/libio/wstrops.c.html#_IO_wstr_jumps) 就位于二者之间，

所以我们是可以利用他们来通过 `IO_validate_vtable` 的检测的，只需要将 `vtable` 填成 `_IO_str_jumps` 或 `_IO_wstr_jumps` 就行。

下面列出几种可行方法，但不限于这几种：

- 利用 `_IO_str_jumps` 中的 [`_IO_str_finish`](https://elixir.bootlin.com/glibc/glibc-2.27/source/libio/strops.c#L346) 函数
  - 为绕过 `_IO_flush_all_lockp` 中的检查进入到 `_IO_OVERFLOW`，需构造：
    - `_mode`
    - `_IO_write_ptr > _IO_write_base`
  - 构造 `vtable = _IO_str_jumps - 0x8`，使 `_IO_str_finish` 函数替代 `_IO_OVERFLOW` 函数（因为 `_IO_str_finish` 在 `_IO_str_jumps` 中的偏移为 `0x10`，而 `_IO_OVERFLOW` 在原 `vtable` 中的偏移为 `0x18`）
  - 构造 `fp -> _IO_buf_base` 作为参数
  - 构造 `fp->flags & _IO_USER_BUF == 0`
  - 构造 `fp->_s._free_buffer = &system`（`_free_buffer = fp->vtable + 0x10`）
  - 调用 `_IO_flush_all_lockp` 函数，触发 `_IO_str_finish`，从而达到调用 `(((_IO_strfile *) fp)->_s._free_buffer) (fp->_IO_buf_base)`

- 利用 `_IO_str_jumps` 中的 [`_IO_str_overflow`](https://elixir.bootlin.com/glibc/glibc-2.27/source/libio/strops.c#L81) 函数
  - 为绕过 `_IO_flush_all_lockp` 中的检查进入到 `_IO_OVERFLOW`，需构造：
    - `_mode`
    - `_IO_write_ptr > _IO_write_base`
  - 构造 `vtable = _IO_str_jumps`，使 `_IO_str_overflow` 函数替代 `_IO_OVERFLOW` 函数（因为 `_IO_str_finish` 在 `_IO_str_jumps` 中的偏移为 `0x18`，而 `_IO_OVERFLOW` 在原 `vtable` 中的偏移为 `0x18`）
  - 构造 `fp->flags & _IO_USER_BUF == 0 && fp->flags & _IO_NO_WRITES == 0`
  - 构造 `new_size = (fp->_IO_buf_end - fp->_IO_buf_base) * 2 + 100 = binsh_addr`
  - 构造 `fp->_s._allocate_buffer = &system`（`fp->_s._allocate_buffer = fp->vtable + 0x8`）
  - 调用 `_IO_flush_all_lockp` 函数，触发 `_IO_str_finish`，从而达到调用 `(*((_IO_strfile *) fp)->_s._allocate_buffer) (new_size)`

### 2.28：指针函数被替换为库函数

- `_IO_str_finish` 中 `(((_IO_strfile *) fp)->_s._free_buffer) (fp->_IO_buf_base)` 被替换为 `free (fp->_IO_buf_base)`
- `_IO_str_overflow` 中 `(*((_IO_strfile *) fp)->_s._allocate_buffer) (new_size)` 被替换为 `malloc (new_size)`

因此，上述两种方法不能再可行，但可以利用 `_IO_str_overflow` 中的 `malloc` 任意大小和 `free` 任意地址的 chunk。

## Problem

- [https://www.root-me.org/en/Challenges/App-System/ELF-x64-FILE-structure-hijacking](https://www.root-me.org/en/Challenges/App-System/ELF-x64-FILE-structure-hijacking)

## Ref

- [https://www.anquanke.com/post/id/216290](https://www.anquanke.com/post/id/216290)

## 利用 `_IO_write_base` 实现 leak

修改 `libc.sym["_IO_2_1_stdout_"] - 0x43` 处的 chunk 为

```python
payload = b''.join([

    b'\0' * 0x33,
    p64(0xfbad1887),                # _flags
    p64(0),                         # _IO_read_ptr
    p64(0),                         # _IO_read_end
    p64(0),                         # _IO_read_base
    p64(libc.sym['__curbrk']),      # _IO_write_base
    p64(libc.sym['__curbrk'] + 8)   # _IO_write_ptr
])
```

从而利用 [`_IO_new_file_xsputn`](https://code.woboq.org/userspace/glibc/libio/fileops.c.html#_IO_new_file_xsputn) 中的

```c
/* Next flush the (full) buffer. */
if (_IO_OVERFLOW (f, EOF) == EOF)
// [#0] write
// [#1] _IO_new_file_write
// [#2] new_do_write
// [#3] _IO_new_do_write
// [#4] _IO_new_file_overflow
// [#5] _IO_new_file_xsputn
// [#6] _IO_puts
```

进行泄漏，泄漏后

```c
count = new_do_write (f, s, do_write);

// [#0] write
// [#1] _IO_new_file_write
// [#2] new_do_write
// [#3] _IO_new_do_write
// [#4] _IO_new_file_xsputn
// [#5] _IO_puts
```

打印后续数据。

## Refs

- [https://ray-cp.github.io/archivers/IO_FILE_arbitrary_read_write](https://ray-cp.github.io/archivers/IO_FILE_arbitrary_read_write)
- [https://jcxp.github.io/2019/09/19/利用-IO-2-1-stdout-泄露信息/](https://jcxp.github.io/2019/09/19/利用-IO-2-1-stdout-泄露信息/)
- [https://xz.aliyun.com/t/2411](https://xz.aliyun.com/t/2411)
