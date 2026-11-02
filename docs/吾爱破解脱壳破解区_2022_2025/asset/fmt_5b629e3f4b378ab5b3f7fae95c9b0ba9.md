
# `_IO_FILE` 利用分析

## `_IO_OVERFLOW` 宏与 `vtable` 校验

```
#define _IO_OVERFLOW(FP, CH) JUMP1 (__overflow, FP, CH)
#define JUMP1(FUNC, THIS, X1) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1)
# define _IO_JUMPS_FUNC(THIS) \
  (IO_validate_vtable                                                   \
   (*(struct _IO_jump_t **) ((void *) &_IO_JUMPS_FILE_plus (THIS)   \
                 + (THIS)->_vtable_offset)))

IO_validate_vtable (const struct _IO_jump_t *vtable)
{
    // 计算 _IO_vtable 长度
    uintptr_t section_length = __stop___libc_IO_vtables - __start___libc_IO_vtables;
    const char *ptr = (const char *) vtable;
    // 计算 ptr 与 __start___libc_IO_vtables 距离
    uintptr_t offset = ptr - __start___libc_IO_vtables;
    // 如果即不在 _IO_vtable_段内, 则调用 _IO_vtable_check ()
    if (__glibc_unlikely (offset >= section_length))
        _IO_vtable_check ();
    return vtable;
}

void attribute_hidden
_IO_vtable_check (void)
{
#ifdef SHARED
  /* Honor the compatibility flag.  */
  void (*flag) (void) = atomic_load_relaxed (&IO_accept_foreign_vtables);
#ifdef PTR_DEMANGLE
  PTR_DEMANGLE (flag);
#endif
  if (flag == &_IO_vtable_check)
    return;

  /* In case this libc copy is in a non-default namespace, we always
     need to accept foreign vtables because there is always a
     possibility that FILE * objects are passed across the linking
     boundary.  */
  {
    Dl_info di;
    struct link_map *l;
    if (_dl_open_hook != NULL
        || (_dl_addr (_IO_vtable_check, &di, &l, NULL) != 0
            && l->l_ns != LM_ID_BASE))
      return;
  }

#else /* !SHARED */
  /* We cannot perform vtable validation in the static dlopen case
     because FILE * handles might be passed back and forth across the
     boundary.  Therefore, we disable checking in this case.  */
  if (__dlopen != NULL)
    return;
#endif

  __libc_fatal ("Fatal error: glibc detected an invalid stdio handle\n");
}
```

- 判断 `vtable` 的地址是否处于 glibc 中的 `vtable` 数组段，是的话，通过检查。
- 否则判断是否为外部的合法 `vtable`（重构或是动态链接库中的 `vtable`），是的话，通过检查。
- 否则报错，输出 `Fatal error: glibc detected an invalid stdio handle`，程序退出。

## 4. `_fileno` 相关利用

`_IO_FILE` 在使用标准 IO 库时会进行创建并负责维护一些相关信息，其中有一些域是表示调用 `fwrite`、`fread` 等函数时写入地址或读取地址的，如果可以控制这些数据就可以实现任意地址写或任意地址读。进程中包含了系统默认的三个文件流 `stdin`、`stdout`、`stderr`，因此这种方式可以不需要进程中存在文件操作，通过 `scanf`、`printf` 一样可以进行利用。

- `fp->_fileno` 的值就是文件描述符，`stderr` 值为 2，`stdout` 值为 1，`stdin` 值为 0。
- `fp->_IO_buf_base` 表示操作的起始地址。
- `fp->_IO_buf_end` 表示结束地址。

### `stdin` 任意写

大致了解 `fread` 的执行流程后，还需要绕过以下检查：

```
_IO_size_t

_IO_file_xsgetn (_IO_FILE *fp, void *data, _IO_size_t n)

{

    ...

    /* fp->_IO_buf_base == NULL 会调用 _IO_doallocbuf (fp) 初始化缓冲区 */

      if (fp->_IO_buf_base == NULL)

    {

        /* Maybe we already have a push back pointer.  */

        if (fp->_IO_save_base != NULL)

        {

            free (fp->_IO_save_base);

               fp->_flags &= ~_IO_IN_BACKUP;

        }

        _IO_doallocbuf (fp);

    }

    /* 如果 fp->_IO_read_end > fp->_IO_read_ptr 则会将缓冲区内容复制到目标地址 */

    ...

    have = fp->_IO_read_end - fp->_IO_read_ptr;

    ...

    if (have > 0)

    {

        s = __mempcpy (s, fp->_IO_read_ptr, have);

        want -= have;

        fp->_IO_read_ptr += have;

     }

     ...

     /* 如果输入长度大于缓冲区大小则会直接读入 */

     if (fp->_IO_buf_base && want _IO_buf_end - fp->_IO_buf_base))

     {

         if (__underflow (fp) == EOF)

            break;

        continue;

     }

    ...

}
```

即：

- 设置 `_IO_read_end` 等于 `_IO_read_ptr`。
- 设置 `_flag &~ _IO_NO_READS`，即 `_flag &~ 0x4`。
- 设置 `_fileno` 为 0，表示读入数据的来源是 `stdin`。
- 设置 `_IO_buf_base` 为 `write_start`，`_IO_buf_end` 为 `write_end`；
- 使得 `_IO_buf_end - _IO_buf_base` 大于 `fread` 要读的数据。

#### 示例

```
#include
#include
#define _GNU_SOURCE

typedef unsigned long long i64;
char buf[100];

int main() {
    char stack_buf[100];
    i64 libc_base = (i64) &puts - 0x84420;
    // 0x1ec980 为 _IO_2_1_stdin_ 偏移
    FILE *fp = libc_base + 0x1ec980;
    fp->_IO_read_end = fp->_IO_read_ptr = 0x0;
    fp->_flags &= ~0x4;
    fp->_fileno = 0x0;
    fp->_IO_buf_base = (char *) buf;
    fp->_IO_buf_end = (char *) &buf[99];
    fread(stack_buf, 1, 3, fp);
    printf("buf: %s", buf);
    printf("stack_buf: %s\n", stack_buf);
    return 0;
}
```

使用 `libc6_2.31-0ubuntu9.9_amd64` 版本编译运行结果如下：

![image-20231020153758598.png](https://attach.52pojie.cn/forum/202310/30/170235chfbb2bhbzgbhhe9.png)

`image-20231020153758598.png` (9.61 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjU4M3xiYmY5OTRhY3wxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

30

2023-10-30 17:02 上传

### `stdout` 任意写

有如下源码：

```
_IO_new_file_xsputn (_IO_FILE *f, const void *data, _IO_size_t n)
{
    ...
    else if (f->_IO_write_end > f->_IO_write_ptr)
        count = f->_IO_write_end - f->_IO_write_ptr;

    if (count > 0)
    {
        if (count > to_do)
            count = to_do;

        f->_IO_write_ptr = __mempcpy (f->_IO_write_ptr, s, count);

        s += count;

        to_do -= count;
    }

    ...
}
```

将 `_IO_write_ptr` 指向 `write_start`，`_IO_write_end` 指向 `write_end` 即可实现在目标地址写入数据。

#### 示例

```
#include
#include
#define _GNU_SOURCE

typedef unsigned long long i64;
char buf[5];

int main() {
    char *stack_buf = "abcdefg";
    i64 libc_base = (i64) &puts - 0x84420;
    FILE *fp = (FILE *) (libc_base + 0x1ed6a0);
    fp->_IO_write_ptr = (char *) &buf[0];
    fp->_IO_write_end = (char *) &buf[4];
    fwrite(stack_buf, 1, 8, fp);
    printf("\nbuf: %s\n", buf);
    return 0;
}
```

使用 `libc6_2.31-0ubuntu9.9_amd64` 版本编译运行结果如下：

![image-20231020182423234.png](https://attach.52pojie.cn/forum/202310/30/170255mu1vho4xk6g16doo.png)

`image-20231020182423234.png` (7.47 KB, 下载次数: 2)

[附件下载](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY1MjU4NHw0NTE2Y2E3M3wxNzkzNTczMDQwfDE2ODA4NnwxODUwNDcw&nothumb=yes)

31

2023-10-30 17:02 上传

### `stdout` 任意读

程序正确执行到 `_IO_overflow` 时，有如下源码：

```
_IO_size_t

_IO_new_file_xsputn (_IO_FILE *f, const void *data, _IO_size_t n)

{

    ...

    /*

    *  f->_IO_write_end > f->_IO_write_ptr 就会将待输出的数据写入缓冲区，_IO_overflow 只有在输出缓冲区写满的时候才将其       *  输出。因此为了不造成不必要的麻烦，直接令 f->_IO_write_end = f->_IO_write_ptr 。

    */

    else if (f->_IO_write_end > f->_IO_write_ptr)

        count = f->_IO_write_end - f->_IO_write_ptr; /* Space available. */

      /* Then fill the buffer. */

      if (count > 0)

    {

          if (count > to_do)

            count = to_do;

          f->_IO_write_ptr = __mempcpy (f->_IO_write_ptr, s, count);

          s += count;

          to_do -= count;

    }

      ...

}
```
