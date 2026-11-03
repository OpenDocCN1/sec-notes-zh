
```c
//go:build ignore

#include "vmlinux.h"

char __license[] SEC("license") = "GPL";

struct sys_enter_args {
   unsigned short common_type;
   unsigned char common_flags;
   unsigned char common_preempt_count;
   int common_pid;

   long id;
   unsigned long args[6];
};

SEC("tracepoint/raw_syscalls/sys_enter")
int trace_sys_enter(struct sys_enter_args *args)
{
    u32 syscall_nr;

    syscall_nr = args->id;

    bpf_printk("syscall_nr: %d",syscall_nr);

    return 0;
}
```

`bpf_printk`函数打印的结果在`/sys/kernel/tracing/trace_pipe`文件中(PS:有些机型在`/sys/kernel/debug/tracing/trace_pipe`文件中，下同，下面的不在重复解释)，观看`bpf_printk`函数结果需要先将`/sys/kernel/tracing/tracing_on`文件中的值置为1

### 用户层

```go
package main

import (

        "log"

        "time"

        "github.com/cilium/ebpf/link"

        "github.com/cilium/ebpf/rlimit"

)

//go:generate go run github.com/cilium/ebpf/cmd/bpf2go --target=amd64 bpf blog.c -- -I./headers

func main() {

        if err := rlimit.RemoveMemlock(); err != nil {

                log.Fatal(err)

        }

        objs := bpfObjects{}

        if err := loadBpfObjects(&objs, nil); err != nil {

                log.Fatalf("loading objects: %v", err)

        }

        defer objs.Close()

        kp, err := link.Tracepoint("raw_syscalls","sys_enter",objs.TraceSysEnter,nil)

        if err != nil {

            log.Fatal(err)

        }

        defer kp.Close()

        ticker := time.NewTicker(1 * time.Minute)

        defer ticker.Stop()

        log.Println("Waiting for events..")

        for range ticker.C {

            log.Printf("get rule\n")

        }

}
```

效果图如下：

![](https://attach.52pojie.cn/forum/202404/24/184101vpp1frpk8zi1pakc.png)

6.png (160.87 KB, 下载次数: 0)

[href="forum.php?mod=attachment&aid=MjY5MTc0MXw4NjYzOTFiY3wxNzkzNTgyMTA3fDE2ODA4NnwxOTE3MzM5&nothumb=yes" target="_blank"](https://www.52pojie.cn/forum.php?mod=attachment&aid=MjY5MTc0MXw4NjYzOTFiY3wxNzkzNTgyMTA3fDE2ODA4NnwxOTE3MzM5&nothumb=yes)

2024-4-24 18:41 上传

## 7.3 其他事件类型

`ebpf`还有其他事件类型，例如`socket`、`sockops`、`tc`、`xdp`等等，但这些更多与流量控制息息相关，跟我们在移动安全上的关联性不是很大，这里不在举例说明，当然还有`uprobe`事件类型，这个是用户层插桩的，但用户层插桩更推荐`frida`这些框架，而且`uprobe`在`linux`使用体验感还好，在`Android`端使用去插桩`APP`过于麻烦了。

## 八、一些使用技巧

### 8.1 将数据从用户空间传输到内核空间

在`cilium`中，`ringbuffer`并不支持将数据从用户空间传递到内核空间，只支持将数据从内核空间发送到用户空间，在新的数据传输框架`BPF_MAP_TYPE_USER_RINGBUF`支持将数据从用户空间传输到内核空间，但是遗憾的是，`cilium`暂不支持该框架

在我们需要传输一些过滤条件或者动态的全局配置到内核层去过滤的时候需要怎么做喃？可以考虑监控特定的文件名、特定的命令等来获取数据，当然这种方式仅时候传递数据量不大的情况

### 8.2 获取UID

`UID`是啥，`UID`是`android`中`uid`用于标识一个应用程序，`uid`在应用安装时被分配，并且在应用存在于手机上期间，都不会改变，可以理解为`app`的唯一身份标识，在`ebpf`中，可以用来过滤指定`app`的数据

`ebpf`可以使用`bpf_get_current_uid_gid`函数来获取`UID`，该函数返回值为`u32`类型
