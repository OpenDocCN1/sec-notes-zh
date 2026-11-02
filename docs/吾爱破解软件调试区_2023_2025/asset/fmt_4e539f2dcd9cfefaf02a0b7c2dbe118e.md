
```c
#include "init.h"

#include "thread.h"

#include "interrupt.h"

#include "console.h"

#include "process.h"

#include "syscall-init.h"

#include "syscall.h"

void k_thread_a(void*);     //自定义线程函数

void k_thread_b(void*);

void u_prog_a(void);

void u_prog_b(void);

int prog_a_pid = 0, prog_b_pid = 0;

int main(void){

  put_str("I am Kernel\n");

  init_all();

  process_execute(u_prog_a, "user_prog_a");

  process_execute(u_prog_b, "user_prog_b");

  intr_enable();

  console_put_str(" main_pid:0x");

  console_put_int(sys_getpid());

  console_put_char('\n');

  thread_start("k_thread_a", 31, k_thread_a, " A_");

  thread_start("k_thread_b", 31, k_thread_b, " B_");

  while(1);//{

    //console_put_str("Main ");

  //};

  return 0;

}

/* 在线程中运行的函数 */

void k_thread_a(void* arg){

  /* 这里传递通用参数，这里被调用函数自己需要什么类型的就自己转换 */

  char* para = arg;

  console_put_str(" thread_a_pid:0x");

  console_put_int(sys_getpid());

  console_put_char('\n');

  console_put_str(" prog_a_pid:0x");

  console_put_int(prog_a_pid);

  console_put_char('\n');

  while(1);

}

void k_thread_b(void* arg){

  /* 这里传递通用参数，这里被调用函数自己需要什么类型的就自己转换 */

  char* para = arg;

  console_put_str(" thread_b_pid:0x");

  console_put_int(sys_getpid());

  console_put_char('\n');

  console_put_str(" prog_b_pid:0x");

  console_put_int(prog_b_pid);

  console_put_char('\n');

  while(1);

}

void u_prog_a(void){

  prog_a_pid = getpid();

  while(1);

}

void u_prog_b(void){

  prog_b_pid = getpid();

  while(1);

}
```

这里十分逻辑十分简单，那就是内核线程照例使用`sys_getpid()`函数来获取pid打印，而用户进程则使用`getpid()`来进行系统调用，然后最后会执行`sys_getpid()`，但是目前他还不会打印，所以让另外两个线程帮忙打印，下面就是咱们的实现结果：

![](http://imgsrc.baidu.com/forum/pic/item/50da81cb39dbb6fd791bbbe04c24ab18962b37b7.jpg)

这里看出我们确实打印出来了各自的pid，十分顺利。（这里pid的值从1开始的原因在以后我们讲解pid的时候再次进行解释）

## 0x02 实现进程打印的好帮手 - printf

接下来大伙可能会碰到很多老朋友了，比如说`printf`，`write`，`stdio`等，大家以前应该就了解过，所以接下来的工作不要太简单。

首先我们知道`printf`跟咱们内核实现的`put_str`肯定是有所不同的，其中还包括了格式化的功能，也就是`%d`等，然后解析相应参数。而最终真正实现打印的是咱们的`write`系统调用，这里由于目前我们并没有实现文件系统，所以先编写一个简单的`write`系统调用，这里我们跟上面添加`getpid`系统调用即可。

首先是修改一下`syscall.h`，在里面的枚举类型中添加一下`SYS_WRITE`，这里比较简单就不贴代码然后我们到`syscall.c`中添加相应函数

```c
/* 系统调用write */
uint32_t write(char* str){
  return _syscall1(SYS_WRITE, str);
}
```

之后我们再修改`syscall-init.c`即可

```c
/* 打印字符串（未实现文件系统版本） */
uint32_t sys_write(char* str){
  console_put_str(str);
  return strlen(str);
}
```

这里还是十分简单的，接下来正式开始咱们的格式化工作。

### 1. vsprintf

调用`printf`后真正实现格式化的工作的是`vsprintf`，这里我们使用`man vsprintf`来查看他的功能。

![](http://imgsrc.baidu.com/forum/pic/item/9213b07eca806538b888fed5d2dda144ac348292.jpg)

该函数的功能就是把`ap`指向的可变参数，以字符串格式`format`中的符号`%`为替换标记，不修改原格式字符串`format`，将`format`中除了`%`类型字符以外的内容复制到`str`，解释完之后我们立刻开始实现，我们创建文件`lib/stdio.c`

```c
#include "stdio.h"
#include "stdint.h"
#define va_start(ap, v) ap = (va_list)&v    //把ap指向第一个固定参数v
#define va_arg(ap, t) *((t*)(ap += 4))      //ap指向下一个参数并返回其值，由于是32位程序，所以栈上一个参数占4字节
#define va_end(ap) ap = NULL                //清楚ap

/* 将整型转换成字符 */
static void itoa(uint32_t value, char** buf_ptr_addr, uint8_t base){
  uint32_t m = value % base;        //取余
  uint32_t i = value / base;        //取整
  if(i){                            //如果倍数部位0,则递归调用
    itoa(i, buf_ptr_addr, base);
  }
  if(m 
```

这里的函数个人认为还是仔细点看比较好，其中`va_list`我是在`stdio.h`中定义为`char *`变量，这里我们的工作就是将原本的格式化字符串一个字符一个字符的写入我们提前预留的字符缓冲区当中，然后进行`write`系统调用，接下来我们就来`main`函数中测试一下：

```c
void k_thread_b(void* arg){
  /* 这里传递通用参数，这里被调用函数自己需要什么类型的就自己转换 */
  char* para = arg;
  console_put_str(" thread_b_pid:0x");
  console_put_int(sys_getpid());
  console_put_char('\n');
  while(1);
}

void u_prog_a(void){
  printf(" prog_a_pid:0x%x\n",getpid());
  while(1);
}

void u_prog_b(void){
  printf(" prog_b_pid:0x%x\n",getpid());
  while(1);
}
```

这里可以看到咱们进程不再依赖于线程打印了，而是使用`printf`进行输出，我们来看看效果

![](http://imgsrc.baidu.com/forum/pic/item/8ad4b31c8701a18bd8365789db2f07082938fe13.jpg)

这里可以看到咱们进程自己实现的pid成功打印！！

### 2. 完善 printf

上面我们在`vsprintf`当中仅仅处理了`%x`，但其他的类型却没有实现，这里我们将他们补充完整，如下：

```c
/* 将参数ap按照格式format输出到字符串str，并返回替换后的str长度 */
uint32_t vsprintf(char* str, const char* format, va_list ap){
  char* buf_ptr = str;
  const char* index_ptr = format;
  char index_char = *index_ptr;
  int32_t arg_int;
  char* arg_str;
  while(index_char){
    if(index_char != '%'){
      *(buf_ptr++) = index_char;
      index_char = *(++index_ptr);
      continue;
    }
    index_char = *(++index_ptr);    //获取%后面的字符
    switch(index_char){
      case 'x':             //十六进制打印
        arg_int = va_arg(ap, int);  //最开始ap是指向format，然后每次取下一个参数
        itoa(arg_int, &buf_ptr, 16);
        index_char = *(++index_ptr); //跳过格式字符并且更新index_char
        break;
      case 's':             //字符串打印
        arg_str = va_arg(ap, char*);
        strcpy(buf_ptr, arg_str);
        buf_ptr += strlen(arg_str);
        index_char = *(++index_ptr);
        break;
      case 'c':
        *(buf_ptr++) = va_arg(ap, char);
        index_char = *(++index_ptr);
        break;
      case 'd':
        arg_int = va_arg(ap, int);
        /* 如果是负数，将其转换为正数后，在正数前面输出一个负号‘-’ */
        if(arg_int 
```

我们使用`printf`测试一下：

```c
  printf(" prog_a_pid:0x%x,and integer is %d, string is %s ,char is %c\n",getpid(),22,"hello world", 'S');
```

结果仍然十分成功

![](http://imgsrc.baidu.com/forum/pic/item/f3d3572c11dfa9ecf93112d127d0f703908fc1dd.jpg)

## 0x03 堆内存管理

首先我们需要了解一点`arena`的知识，因为咱们目前分配的粒度是`4KB`，但是真正用户进程开始工作之后一般就是多少字节的分配，所以咱们必须实现更小的粒度分配，因此咱们有必要对于这种情况进行管理，大伙可能都对`main_arena`比较熟悉，这里我们仅仅实现简单的版本，每一个`arena`都提供同种大小的内存块供用户分配。所以我们只需要知道他是一个内存供应商就行了.

### 1. 底层初始化

首先我们先到`memory.h`中添加相应的数据结构：

```c
/* 内存块 */
struct mem_block{
  struct list_elem free_elem;
};

/* 内存块描述符，一个描述符描述对应的arena */
struct mem_block_desc{
  uint32_t block_size;  //内存块大小
  uint32_t blocks_per_arena;    //本arena中可容纳此mem_block的数量
  struct list free_list;        //目前可用的mem_block链表
};

#define DESC_CNT 7              //内存块描述符个数，这里我们实现了16,32,64,128,256,512,1024字节这几种规格
```

这里`free_list`是存放对应`arena`中空闲的块链表。然后就是添加的`memory.c`函数

```c
/* 内存仓库 */
struct arena{
  struct mem_block_desc* desc;  //此arena关联的mem_block_desc
  /* large为true时，cnt表示的是页框数。
   * 否则cnt表示空闲的mem_block数量 */
  uint32_t cnt;
  bool large;
};

struct mem_block_desc k_block_descs[DESC_CNT];  //内核内存块描述符数组

/* 为malloc做准备 */
void block_desc_init(struct mem_block_desc* desc_array){
  uint16_t desc_idx,block_size = 16;
  /* 初始化每个mem_block_desc描述符 */
  for(desc_idx = 0; desc_idx 
```

这里给出一个图来解释`mem_block_desc`和`arena`和`mem_block`的关系

![](http://imgsrc.baidu.com/forum/pic/item/0df3d7ca7bcb0a467954cdff2e63f6246a60af86.jpg)

也就是说`arena`是存在于已分配的页面中的，所以这里我们内存块的数量需要减去他结构体的大小再进行计算,其余就是一些普通的初始化过程。

### 2. 实现 sys_malloc

大伙一看前面的`sys_`应该就反应到需要改写`syscall`了，`sys_malloc`的功能是分配并维护内存块资源，动态创建`arena`以满足内存块的分配，为了完成`sys_malloc`，我们这里还需要做一些细小的改动,首先就是让咱们的线程以及用户进程都支持内存管理，所以先添加下面`task_struct`中的变量，然后在`process.c`中添加初始化资源即可（内核管理模块我们在`memory.c`已经写了）

```c
struct mem_block_desc u_block_desc[DESC_CNT];     //用户进程的内存块描述符
```

然后就是最繁琐的`memory.c`的改进:

```c
/* 返回arena中第idx个内存块的地址 */
static struct mem_block* arena2block(struct arena* a, uint32_t idx){
  return (struct mem_block*) ((uint32_t)a + sizeof(struct arena) + idx * a->desc->block_size);
}

/* 返回内存块b所在的arena地址 */
static struct arena* block2arena(struct mem_block* b){
  return (struct arena*)((uint32_t)b & 0xfffff000);
}

/* 在堆中申请size字节内存 */
void* sys_malloc(uint32_t size){
  enum pool_flags PF;
  struct pool* mem_pool;
  uint32_t pool_size;
  struct mem_block_desc* descs;
  struct task_struct* cur_thread = running_thread();
  /* 判断使用哪个内存池 */
  if(cur_thread->pgdir == NULL){    //若为内核线程
    PF = PF_KERNEL;
    pool_size = kernel_pool.pool_size;
    mem_pool = &kernel_pool;
    descs = k_block_descs;
  }else{
    PF = PF_USER;
    pool_size = user_pool.pool_size;
    mem_pool = &user_pool;
    descs = cur_thread->u_block_desc;
  }
  /* 若申请的内存不再内存池容量范围内，则直接返回NULL */
  if(!(size > 0 && size lock);
  /* 超过最大内存块，就分配页框 */
  if(size > 1024){
    uint32_t page_cnt = DIV_ROUND_UP(size + sizeof(struct arena), PG_SIZE);     //向上取整需要的页框数
    a = malloc_page(PF, page_cnt);
    if(a != NULL){
      memset(a, 0, page_cnt * PG_SIZE);     //将分配的内存清0
      /* 对于分配的大块页框，将desc置为NULL,
      * cnt置为页框数，large置为true */
```
