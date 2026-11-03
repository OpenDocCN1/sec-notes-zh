
# [原创] Linux下修改Wine运行的PE的内存

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=1878892
- 抓取时间: 2026-11-01T22:39:28

众~~~所周知，`Wine`是个好东西，Linux系统可以用它跑Windows程序。它通过实现一系列的Windows API，将Windows PE (可执行和可载入文件格式)转换为Linux操作系统可以理解的格式。让Windows应用在Linux上找到家的感觉。

再众~~~所周知，Linux下有个有趣的特性——“一切皆文件”。这意味着能在`/proc/{pid}/maps`里一窥进程的内存分布，还能在`/proc/{pid}/mem`里随心所欲的“画饼”。理论成立，开始实践。

实践只是为了验证前面的猜想，当然得挑个软柿子捏，于是就找到了一个快20年前的老游戏：“XXX大作战”。

![](https://attach.52pojie.cn/forum/202401/10/141617ya04mwmwmqunuz8u.png)
Screenshot_20240110_141415.png (383.45 KB, 下载次数: 1)
[href="forum.php?mod=attachment&aid=MjY2ODk5NnxmN2JmYmY0M3wxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2ODk5NnxmN2JmYmY0M3wxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes)
2024-1-10 14:16 上传

操作第一步，探个究竟，`Wine`是否真把PE内存镜像本本原原地摆放好了。

通过命令`ps -e | grep -i MONE`，锁定目标进程`PID`为`17551`。

![](https://attach.52pojie.cn/forum/202401/10/141637uaa5own1hjwavsuh.png)
2.png (15.93 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY2ODk5N3wyNDc3ZTJlZnwxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2ODk5N3wyNDc3ZTJlZnwxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes)
2024-1-10 14:16 上传

接着，`cat /proc/17551/maps | grep exe`揭秘了内存布局，果然跟Windows如出一辙，PE文件映射从`0x00400000`起点开始。

![](https://attach.52pojie.cn/forum/202401/10/141707vbgbeqq3et1sk3ee.png)
3.png (116.87 KB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY2ODk5OXxmYTQxMDBiZHwxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2ODk5OXxmYTQxMDBiZHwxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes)
2024-1-10 14:17 上传

这时就能来点`write`和`read`操作，悄悄摸进`/proc/{pid}/mem`修改内存，居然比Windows还要简单。

首先是捕捉游戏进程的`PID`，一段轮子代码搞定。

```c++
int findPid(string processName) {
    FILE *fp;
    char path[1035];
    fp = popen("ps -ef", "r");
    if (fp == nullptr) {
        printf("Failed to run command\n");
        exit(1);
    }
    while (fgets(path, sizeof(path) - 1, fp) != nullptr) {
        if (::strstr(path, processName.c_str()) != nullptr) {
            ::strtok(path, " ");
            auto token = ::strtok(nullptr, " ");
            return stoi(token);
        }
    }
    pclose(fp);
    return -1;
}
```

接下来，打开对应的`mem`文件，妙哉，这文件描述符就相当于是Windows的句柄。

```c++
int openMem(int pid) {
    char mem_filename[1024];
    sprintf(mem_filename, "/proc/%d/mem", pid);
    return open(mem_filename, O_RDWR);
}
```

后续两个模板函数分别操控读写内存。

```c++
template
T readMem(long address, int mem_file) {
    T a;
    if (lseek(mem_file, address, SEEK_SET) == -1) {
        ::printf("lseek error %s \n", strerror(errno));
    }
    read(mem_file, &a, sizeof(T));
    return a;
}
```

```c++
template
void writeMem(long address, T value, int mem_file) {
    if (lseek(mem_file, address, SEEK_SET) == -1) {
        ::printf("lseek error %s \n", strerror(errno));
    }
    write(mem_file, &value, sizeof(T));
}
```

该有的都有了，最后把它们组合起来，验证一下功能。

```c++
#include 
#include 
#include 
#include 
const unsigned int num_current_offset = 0x268c; // 当前收集的数量
const unsigned int kind_current_offset = 0x2690; // 当前收集的种类
const unsigned int isShot_current_offset = 0x2680; // 是否可以释放
using namespace std;
int findPid(string processName) {
    FILE *fp;
    char path[1035];
    fp = popen("ps -ef", "r");
    if (fp == nullptr) {
        printf("Failed to run command\n");
        exit(1);
    }
    while (fgets(path, sizeof(path) - 1, fp) != nullptr) {
        if (::strstr(path, processName.c_str()) != nullptr) {
            ::strtok(path, " ");
            auto token = ::strtok(nullptr, " ");
            return stoi(token);
        }
    }
    pclose(fp);
    return -1;
}
int openMem(int pid) {
    char mem_filename[1024];
    sprintf(mem_filename, "/proc/%d/mem", pid);
    return open(mem_filename, O_RDWR);
}
template
T readMem(long address, int mem_file) {
    T a;
    if (lseek(mem_file, address, SEEK_SET) == -1) {
        ::printf("lseek error %s \n", strerror(errno));
    }
    read(mem_file, &a, sizeof(T));
    return a;
}
template
void writeMem(long address, T value, int mem_file) {
    if (lseek(mem_file, address, SEEK_SET) == -1) {
        ::printf("lseek error %s \n", strerror(errno));
    }
    write(mem_file, &value, sizeof(T));
}
int main() {
    int pid = findPid("MONE~OUQ.EXE");
    if (pid == -1)
        return -1;
    int mem_file = openMem(pid);
    long address = 0x484a3c;
    auto class_address = readMem(address, mem_file);
    while (true) {
        writeMem(class_address + num_current_offset, 2, mem_file);
        writeMem(class_address + kind_current_offset, 21, mem_file);
        writeMem(class_address + isShot_current_offset, 1, mem_file);
        sleep(1);
    }
    return 0;
}
```

最后就是效果图啦

![](https://attach.52pojie.cn/forum/202401/10/141314ei9z3d4p17d3mylc.gif)
2024-01-10 13-16-40 (6).gif (2.82 MB, 下载次数: 2)
[href="forum.php?mod=attachment&aid=MjY2ODk5NXwxNzNkNjBmOHwxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY2ODk5NXwxNzNkNjBmOHwxNzkzNTcyNzY1fDE2ODA4NnwxODc4ODky&nothumb=yes)
2024-1-10 14:13 上传

ps：因为不是用windows api `OpenProcess`、`ReadProcessMemory`、`WriteProcessMemory`，不知道windows下传统防内存修改方法能不能防得住。
