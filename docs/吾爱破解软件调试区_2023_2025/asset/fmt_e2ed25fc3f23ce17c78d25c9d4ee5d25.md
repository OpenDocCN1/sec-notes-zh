
# 将用户程序写入硬盘

我们要将自己实现的用户程序写入硬盘，先完成三项工作：

- 编写一个真正的用户程序
- 将用户程序写入文件系统
- 在 `shell` 中执行用户程序，也就是外部命令

## 编写用户程序

首先我们来编写用户程序，新建函数`command/prog_no_arg.c`，我们先实现一个比较简单无参数的：

```c
#include "stdio.h"
int main(void){
  printf("prog_no_arg from disk\n");
  while(1);
  return 0;
}
```

还是十分简单的，这里加上 `while(1)` 循环来防止程序乱跑。接下来我们的编译也要重新设置，编写函数`command/compile.sh`：

```bash
######## 应在command目录下执行 #########

if [[ ! -d "../lib" || ! -d "../build" ]]; then
  echo "dependent dir do not exist!"
  cwd=$(pwd)
  cwd=${cwd##*/}
  cwd=${cwd%/}
  if [[ $cwd != "command" ]]; then
    echo -e "you would better in command dir\n"
  fi
  exit
fi

BIN="prog_no_arg"
CFLAGS="-m32 -fno-stack-protector -Wall -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes -Wsystem-headers"
LIB="../lib/"
OBJS="../build/string.o ../build/syscall.o ../build/stdio.o ../build/assert.o"
DD_IN=$BIN
DD_OUT="/home/dawn/repos/OS_learning/bochs/hd60M.img"

gcc $CFLAGS -I $LIB -o $BIN".o" $BIN".c"
ld -e main $BIN".o" $OBJS -o $BIN
SEC_CNT=$(ls -l $BIN|awk '{printf("%d", ($5+511)/512)}')

if [[ -f $BIN ]]; then
  dd if=./$DD_IN of=./$DD_OUT bs=512 count=$SEC_CNT seek=300 conv=notrunc
fi
```

这里的 `shell` 脚本首先将我们的程序编译打入`hd60M.img`，然后再通过主线程将其写入`hd80M.img`，这样就成功加载了用户程序。

然后我们再修改`kernel/main.c`来写入`hd80M.img`：

```c
int main(void){
  put_str("I am Kernel\n");
  init_all();
  intr_enable();
  /************** 写入应用程序 ************/
  uint32_t file_size = 10236;
  uint32_t sec_cnt = DIV_ROUND_UP(file_size, 512);
  struct disk* sda = &channels[0].devices[0];
  void* prog_buf = sys_malloc(file_size);
  ide_read(sda, 300, prog_buf, sec_cnt);
  int32_t fd = sys_open("/prog_no_arg", O_CREAT|O_RDWR);
  if(fd != -1){
    if(sys_write(fd, prog_buf, file_size) == -1){
      printk("file write error!\n");
      while(1);
    }
  }
  /****×***** 写入应用程序结束  *********/
  cls_screen();
  console_put_str("[peiwithhao@localhost /]$ ");
  while(1);
  return 0;
}
```

这里我们会碰到一个错误，就是我之前写的`switch.c`中有个错误，把`ebx`写成了`edx`，导致我调试了一下午，这时间真白费了。然后上面`main()`函数中的 10236 是咱们用户程序的大小，我们使用`ls`命令就可以获得：

```text
dawn@dawn-virtual-machine:~/repos/OS_learning$ ls -l command/prog_no_arg

-rwxrwxr-x 1 dawn dawn 10236 Feb 18 23:06 command/prog_no_arg
```

然后我们到虚拟机中测试看是否真的可以运行这个程序了，结果如下：

![Screenshot 2023-02-19 173604.png](https://attach.52pojie.cn/forum/202302/20/160359cdzmaguvjs0cs8mz.png)

Screenshot 2023-02-19 173604.png（25.18 KB，下载次数：0）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5MTc5MHxjZTA3ZTA3OXwxNzkzNTcwMjQ5fDE2ODA4NnwxNzQ4NDEz&nothumb=yes>

2023-2-20 16:03 上传

## 4. 支持参数传递

我们在平时的程序中都是依靠栈来传递参数，但是对于我们现在的用户程序来说创建之初还没有栈怎么办呢？实际上我们知道用户程序都有一个`main()`函数，但是这个`main()`函数并不是第一个运行的函数，因此他的参数就是由其他人来传递。而这里那些计算机的大佬就使用库函数对其进行协助，这个库函数实际上也就是一些通用的函数，写到一起方便调用。而后来为了规定一个标准，于是就有了 CRT，也就是 C 运行库，而他的主要功能就是初始化运行环境，也就是进入`main()`函数之前为他准备环境，传递参数等，然后我们再调用`main()`函数才能正确跑起来。

而目前咱们的用户程序大家都可以看到最后是使用`while(1)`来防止程序不会到处乱跑，所以我们还要实现`exit`系统调用来使得`main()`函数结束后陷入内核来将处理器的控制权返回内核。

下面给出 CRT 与用户程序的关系，如下：

![Screenshot 2023-02-19 180446.png](https://attach.52pojie.cn/forum/202302/20/160401e4sv553r9sfv4w3d.png)

Screenshot 2023-02-19 180446.png（96.46 KB，下载次数：0）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5MTc5MXwwMGZkMmE2MXwxNzkzNTcwMjQ5fDE2ODA4NnwxNzQ4NDEz&nothumb=yes>

2023-2-20 16:04 上传

接下来我们首先实现一个 CRT 的简易版本：

```nasm
[bits 32]
extern main
section .text
global _start
_start:
  ;下面这两个要和execv中load之后指定的寄存器一致，也就是ebx和ecx啦
  push ebx  ;压入argv
  push ecx  ;压入argc
  call main
```

这里使用`call`而不是`jmp`就是让`main()`有去有回，不会到处乱跑。我们刚写的`exec.c`大伙应该还记得，当时我们将新进程的参数压入内核中相应的寄存器，因此在这里我们将寄存器中的值压入栈中，这样下去我们的`main()`函数就会有参数可以调用了。

至此我们来重新编写一个有参数的用户程序：

```c
#include "stdio.h"
#include "syscall.h"
#include "string.h"

int main(int argc, char** argv){
  int arg_idx = 0;
  while(arg_idx 
```

该程序的作用就是父进程来延迟显示操作系统的进程信息，而子进程来运行`argv[1]`指向的用户进程。

```bash
######## 应在command目录下执行 #########
if [[ ! -d "../lib" || ! -d "../build" ]]; then
  echo "dependent dir do not exist!"
  cwd=$(pwd)
  cwd=${cwd##*/}
  cwd=${cwd%/}
  if [[ $cwd != "command" ]]; then
    echo -e "you would better in command dir\n"
  fi
  exit
fi

BIN="prog_arg"
CFLAGS="-m32 -fno-stack-protector -Wall -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes -Wsystem-headers"
LIBS="-I ../lib/ -I ../lib/user -I ../fs"
OBJS="../build/string.o ../build/syscall.o ../build/stdio.o ../build/assert.o start.o"
DD_IN=$BIN
DD_OUT="/home/dawn/repos/OS_learning/bochs/hd60M.img"

nasm -f elf ./start.S -o ./start.o
ar rcs simple_crt.a $OBJS start.o
gcc $CFLAGS $LIBS -o $BIN".o" $BIN".c"
ld -m elf_i386 $BIN".o" simple_crt.a -o $BIN
SEC_CNT=$(ls -l $BIN|awk '{printf("%d", ($5+511)/512)}')

if [[ -f $BIN ]]; then
  dd if=./$DD_IN of=$DD_OUT bs=512 count=$SEC_CNT seek=300 conv=notrunc
fi
```

这里我们重新编写一下`compile.sh`来编译`prog_arg.c`。我们首先将`start.S`编译为`start.o`，然后使用`ar`命令将`string.o`、`syscall.o`、`stdio.o`、`assert.o`和`start.o`打包成为一个静态库文件`simple_crt.a`，这就是我们刚刚讲到的简单的 CRT 了。然后我们再修改`main()`函数：

```c
int main(void){
  put_str("I am Kernel\n");
  init_all();
  /************** 写入应用程序 **********/
  uint32_t file_size = 10384;
  uint32_t sec_cnt = DIV_ROUND_UP(file_size, 512);
  struct disk* sda = &channels[0].devices[0];
  void* prog_buf = sys_malloc(file_size);
  ide_read(sda, 300, prog_buf, sec_cnt);
  int32_t fd = sys_open("/prog_arg", O_CREAT|O_RDWR);
  if(fd != -1){
    if(sys_write(fd, prog_buf, file_size) == -1){
      printk("file write error!\n");
      while(1);
    }
  }
  sys_close(fd);
  /******* 写入应用程序结束  *********/
  intr_enable(); 
  cls_screen();
  console_put_str("[peiwithhao@localhost /]$ ");
  while(1);
  return 0;
}

/* init进程 */
void init(void){
  uint32_t ret_pid = fork();
  if(ret_pid){  //父进程
    while(1);
  }else{    //子进程
    my_shell();
  }
  panic("init: should not be here");
}
```

这里我们父进程来打印`ps`信息，子进程使用`execv`来执行`prog_no_arg`，还是十分简单的。然后执行结果如下，我们确实正确地执行：

![Screenshot 2023-02-19 190304.png](https://attach.52pojie.cn/forum/202302/20/160404vy5gyaoja8ycz3jg.png)

Screenshot 2023-02-19 190304.png（26.95 KB，下载次数：0）

<https://www.52pojie.cn/forum.php?mod=attachment&aid=MjU5MTc5Mnw3NDE2ZDQwNHwxNzkzNTcwMjQ5fDE2ODA4NnwxNzQ4NDEz&nothumb=yes>

2023-2-20 16:04 上传

## 0x02 实现系统调用 `exit` 和 `wait`

### 1. `wait` 和 `exit` 的作用

`exit`的作用十分直白，那就是退出进程。在 C 运行库当中会调用`main()`函数执行，`main()`函数结束后程序流程会回到 C 运行库，C 运行库结束之后就会调用`exit`系统调用。

而`wait`的作用是阻塞父进程自己，直到任意一个子进程运行结束。也就是说，某个进程调用`wait`的时候，内核就会去寻找它的子进程。如果没有子进程，这里`wait`会返回`-1`；如果有子进程，那么内核就要去遍历其所有的子进程并查找哪个子进程已经退出了，并且将子进程退出时的返回值传递给父进程，然后将父进程唤醒。这里主要的功能就是阻塞父进程。像我们上一节那样，为了避免父进程和子进程的输出混在一起，我们让父进程循环了 900000，这样不仅不够优雅，且十分浪费 cpu 资源，因此咱们迫切使用`wait`来阻塞自身。

### 2. 孤儿进程和僵尸进程

首先来说说孤儿进程，简单来说也就是子进程没父进程了。也就是说，在子进程还都没有完全执行完毕的时候，此时父进程却因某种事情终止了，此时这些子进程便被称为孤儿进程。

这里僵尸进程就是子进程调用`exit`系统调用进行退出，但是父进程并没有调用`wait`系统调用（用来接收子进程的返回值以及收回子进程所占用的资源），这样就没人帮子进程收尸，那么此时它就变成了僵尸进程。

这俩解释都挺直观的，我们接下来总结两句：

- `exit`由子进程调用，表面上功能是使自己结束并且传递返回值给内核，本质上是内核在幕后将进程除`pcb`外的一切资源回收
- `wait`由父进程调用，表面上是父进程阻塞自己，直到子进程唤醒父进程，然后获得子进程的返回值，本质上是内核在幕后将子进程的返回值传递给父进程并唤醒他，然后回收`PCB`

### 3. 基础代码们

首先我们来写一点基础代码来方便这两个系统调用。

我们刚刚知道子进程的返回值是存放在`pcb`当中的，所以这里我们在`pcb`当中增加一个成员来记录返回值：

```c
int8_T exit_status
```

这里我们再到`kernel/memory.c`当中添加一个内存释放的函数：

```c
/* 根据物理页框地址pg_phy_addr在相应的内存池位图清0,不改动页表 */
void free_a_phy_page(uint32_t pg_phy_addr){
  struct pool* mem_pool;
  uint32_t bit_idx = 0;

  if(pg_phy_addr >= user_pool.phy_addr_start){  //如果是用户内存池
    mem_pool = &user_pool;
    bit_idx = (pg_phy_addr - user_pool.phy_addr_start) / PG_SIZE;
  }else{
    mem_pool = &kernel_pool;
    bit_idx = (pg_phy_addr - kernel_pool.phy_addr_start) / PG_SIZE;
  }
  bitmap_set(&mem_pool->pool_bitmap, bit_idx, 0);
}
```

我们使用它来回收某个指定的物理页框，所以这里我们介绍一下重点函数，我们添加在`thread/thread.c`。
