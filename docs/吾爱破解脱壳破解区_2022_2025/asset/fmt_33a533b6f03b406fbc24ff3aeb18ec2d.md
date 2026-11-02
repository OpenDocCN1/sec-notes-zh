

# [CTF] [PWN] Linux IO_FILE Exploit

- 原帖：https://www.52pojie.cn/forum.php?mod=viewthread&tid=1711489
- 抓取时间：2026-11-01T23:07:06

## IO_FILE Exploit

### Structure

[源码链接](https://code.woboq.org/userspace/glibc/libio/bits/types/struct_FILE.h.html#_IO_FILE)

```c
struct _IO_FILE
{
  int _flags;                /* High-order word is _IO_MAGIC; rest is flags. */

  /* The following pointers correspond to the C++ streambuf protocol. */
  char *_IO_read_ptr;        /* Current read pointer */
  char *_IO_read_end;        /* End of get area. */
  char *_IO_read_base;       /* Start of putback+get area. */
  char *_IO_write_base;      /* Start of put area. */
  char *_IO_write_ptr;       /* Current put pointer */
  char *_IO_write_end;       /* End of put area. */
  char *_IO_buf_base;        /* Start of reserve area. */
  char *_IO_buf_end;         /* End of reserve area. */

  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base;       /* Pointer to start of non-current get area. */
  char *_IO_backup_base;     /* Pointer to first valid character of backup area */
  char *_IO_save_end;        /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;
  int _flags2;
  __off_t _old_offset;       /* This used to be _offset but it's too small.  */

  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```

[源码链接](https://code.woboq.org/userspace/glibc/libio/libioP.h.html#_IO_FILE_plus)

```c
struct _IO_FILE_plus
{
  FILE file;
  const struct _IO_jump_t *vtable;
};
```

[源码链接](https://code.woboq.org/userspace/glibc/libio/libioP.h.html#_IO_jump_t)

```c
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
};
```

下面简单说说一些 C 函数对`_IO_jump_t`虚表里面函数的调用情况：

- `printf`/`puts`最终会调用`_IO_file_xsputn`
- `fclose`最终会调用`_IO_file_finish`
- `fwrite`最终会调用`_IO_file_xsputn`
- `fread`最终会调用`_IO_fiel_xsgetn`
- `scanf`/`gets`最终会调用`_IO_file_xsgetn`

[源码链接](https://code.woboq.org/userspace/glibc/libio/strfile.h.html#_IO_strfile_)

```c
struct _IO_str_fields
{
  /* These members are preserved for ABI compatibility.  The glibc
     implementation always calls malloc/free for user buffers if
     _IO_USER_BUF or _IO_FLAGS2_USER_WBUF are not set.  */

  _IO_alloc_type _allocate_buffer_unused;
  _IO_free_type _free_buffer_unused;
};

/* This is needed for the Irix6 N32 ABI, which has a 64 bit off_t type,
   but a 32 bit pointer type.  In this case, we get 4 bytes of padding
   after the vtable pointer.  Putting them in a structure together solves
   this problem.  */

struct _IO_streambuf
{
  FILE _f;
  const struct _IO_jump_t *vtable;
};

typedef struct _IO_strfile_
{
  struct _IO_streambuf _sbf;
  struct _IO_str_fields _s;
} _IO_strfile;
```

```c
#define _IO_MAGIC 0xFBAD0000 /* Magic number */
#define _OLD_STDIO_MAGIC 0xFABC0000 /* Emulate old stdio. */
#define _IO_MAGIC_MASK 0xFFFF0000
#define _IO_USER_BUF 1 /* User owns buffer; don't delete it on close. */
#define _IO_UNBUFFERED 2
#define _IO_NO_READS 4 /* Reading not allowed */
#define _IO_NO_WRITES 8 /* Writing not allowd */
#define _IO_EOF_SEEN 0x10
#define _IO_ERR_SEEN 0x20
#define _IO_DELETE_DONT_CLOSE 0x40 /* Don't call close(_fileno) on cleanup. */
#define _IO_LINKED 0x80 /* Set if linked (using _chain) to streambuf::_list_all.*/
#define _IO_IN_BACKUP 0x100
#define _IO_LINE_BUF 0x200
#define _IO_TIED_PUT_GET 0x400 /* Set if put and get pointer logicly tied. */
#define _IO_CURRENTLY_PUTTING 0x800
#define _IO_IS_APPENDING 0x1000
#define _IO_IS_FILEBUF 0x2000
#define _IO_BAD_SEEN 0x4000
#define _IO_USER_LOCK 0x8000
```

### FSOP

FSOP 的核心思想就是劫持`_IO_list_all`的值来伪造链表和其中的`_IO_FILE`项，但是单纯的伪造只是构造了数据，还需要某种方法进行触发。

FSOP 选择的触发方法是调用[`_IO_flush_all_lockp`](https://code.woboq.org/userspace/glibc/libio/genops.c.html#_IO_flush_all_lockp)，这个函数会刷新`_IO_list_all`链表中所有项的文件流，相当于对每个`FILE`调用`fflush`，也对应着会调用`_IO_FILE_plus.vtable`中的`_IO_overflow`。

```c
_IO_flush_all_lockp (int do_lock)
{
  int result = 0;
  FILE *fp;

  // ...

  for (fp = (FILE *) _IO_list_all; fp != NULL; fp = fp->_chain)
    {
      run_fp = fp;

      if (do_lock)
        _IO_flockfile (fp);

      if (((fp->_mode _IO_write_ptr > fp->_IO_write_base)
           || (_IO_vtable_offset (fp) == 0
               && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
                                    > fp->_wide_data->_IO_write_base))
           )
          && _IO_OVERFLOW (fp, EOF) == EOF)
        result = EOF;

      //...
    }

  // ...

  return result;
}
```

[`_IO_flush_all_lockp`](https://code.woboq.org/userspace/glibc/libio/genops.c.html#_IO_flush_all_lockp)的触发方式：

- 当执行`exit`函数时
- 当执行流从`main`函数返回时
- 当 libc 执行`abort`流程时 [glibc trace：

